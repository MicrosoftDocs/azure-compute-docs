---
title: Azure N-series AMD GPU Driver Setup for Linux
description: How to set up AMD Radeon PRO V710 NVv5 Linux Installation Guide.
services: virtual-machines
author: nmagatala-MSFT
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.collection: linux
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 10/15/2025
ms.author: padmalathas
# Customer intent: As a cloud engineer managing Linux VMs with AMD GPUs, I want to install the necessary AMD GPU drivers and configure my NVv5-V710 instances, so that I can optimize performance for AI and graphics workloads.
---

# Install AMD GPU drivers on N-series VMs running Linux

**Applies to:** :heavy_check_mark: Linux VMs

> [!IMPORTANT]
> To align with inclusive language practices, we've replaced the term "blacklist" with "blocklist" throughout this documentation. This change reflects our commitment to avoiding terminology that might carry unintended negative connotations or perceived racial bias.
> However, in code snippets and technical references where "blacklist" is part of established syntax or tooling (for example, configuration files, command-line parameters), the original term is retained to preserve functional accuracy. This usage is strictly technical and doesn't imply any discriminatory intent.

## NVads V710-series

To leverage the GPU capabilities of Azure’s new NVads V710-series virtual machines running Linux, you’ll need to install AMD GPU drivers. The AMD GPU driver extension streamlines this process by automating driver installation for NVv710-series VMs. You can manage the extension via the Azure portal, Azure PowerShell, or Azure Resource Manager (ARM) templates. For details on supported operating systems and deployment steps, see [AMD GPU Driver Extension](../extensions/hpccompute-amd-gpu-linux.md) documentation.

The [marketplace image](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/amdinc1746636494855.nvv5_v710_linux_rocm_image?tab=Overview) comes preloaded with the AMD GPU driver, helping accelerate VM setup. This guide explains how to install AMD GPU drivers on Azure NVads V710-series Linux virtual machines (VMs). It covers both automated and manual installation methods specifically for **Ubuntu**. 

### ROCm

> [!NOTE]
> Currently, Azure provides installation instructions for:
> - Ubuntu 22.04
> - Ubuntu 24.04
>   
> For other Linux distributions, see:
> - [Quick start installation guide - ROCm installation (Linux)](https://rocm.docs.amd.com/en/latest/install/rocm.html?fam=radeon&w=compute&gpu=amd-radeon-pro-v710&os=ubuntu&ubuntu-ver=24.04.4&i=pkgman&gfx=gfx1101)
> - [ROCm release history - ROCm Documentation](https://rocm.docs.amd.com/en/latest/release/versions.html#rocm-release-history)

Install the AMD Linux Driver to leverage the full capabilities of the AMD Radeon PRO V710 GPU on an NVv5-V710 GPU Linux instance in Microsoft Azure. The sections that follow provide detailed instructions for installing the Linux driver and running inference workloads using ROCm on this instance type.

### Quick start options

#### Option 1: Use the AMD GPU driver extension

The simplest method is using the AMD GPU Driver Extension, which automates driver installation for NVv710-series VMs. You can deploy this extension through:

- Azure portal
- Azure PowerShell
- Azure Resource Manager templates

#### Option 2: Use preconfigured marketplace image

A marketplace image is available with pre-installed AMD GPU drivers, allowing for faster VM deployment.

#### Option 3: Manual installation

Follow these instructions for manual driver installation and configuration.

---

### ROCm driver installation

#### Prerequisites

**System requirements:**

- Disk size must exceed 64GB for optimal performance
- Supported distributions: Ubuntu 22.04 or Ubuntu 24.04
- Virtual Function Device ID: 7461 (AMD Radeon PRO V710 GPU)

#### Step 1: Verify your system

Follow these steps to verify that your GPU card is detected on your system.

1. Check your Linux distribution:

   ```bash
   cat /etc/*release
   ```

1. Check your kernel version:

   ```bash
   uname -srmv
   ```
> - [ROCm Compatibility Matrix - ROCm Documentation](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html?fam=radeon&gpu=amd-radeon-pro-v710&os=ubuntu&gfx=gfx1101)

1. Verify your GPU card is detected:

   ```bash
   sudo lspci -d 1002:7461
   ```

   You should see output similar to:

   ```
   c3:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 7461
   ```

#### Step 2: Install the driver

The driver installation commands are slightly different depending on whether you're running Ubuntu 22.04 or 24.04.

##### For Ubuntu 22.04

```bash
#Driver Installation
sudo apt update
sudo apt install -y "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo usermod -a -G render,video $LOGNAME

##### For Ubuntu 24.04

sudo apt update
sudo apt install -y amdgpu-dkms
sudo reboot

#ROCM Installation
wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
    gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources > /dev/null <<'EOF'
X-Repo-Id: amdrocm-stable
Types: deb
URIs: https://stable.repo.amd.com/rocm/core/packages/ubuntu2404/
Suites: stable
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/amdrocm.gpg
Enabled: yes
EOF

sudo apt update
sudo apt install -y amdrocm10.0-gfx1101
Check — re-login first so the shell picks up the new tools:
rocminfo | grep gfx                # gfx1101
amd-smi version                    # ROCm 10.0.0 | amdgpu 7.1.3.31500000
```

#### Step 3: Load and verify the driver

Follow these steps to load and verify the driver.

1. Load the driver:

   ```bash
   sudo modprobe amdgpu
   ```

1. Check that the driver loaded successfully:

   ```bash
   sudo dmesg | grep amdgpu
   ```

1. Verify driver status with AMD-SMI:

   ```bash
   amd-smi monitor
   ```

#### Step 4: Enable automatic loading on reboot

Follow these steps to enable automatic loading on reboot.

1. Search for blocklist entries:

   ```bash
   grep amdgpu /etc/modprobe.d/* -rn
   ```

1. If the driver is blocklisted, remove the blocklist:

   ```bash
   sudo nano /etc/modprobe.d/blacklist.conf
   ```

1. Delete the line containing `blacklist amdgpu`, then update initramfs:

   ```bash
   sudo update-initramfs -uk all
   ```

1. Reboot to apply changes:

   ```bash
   sudo reboot
   ```

---

### Graphics and ROCm installation

This section covers installing the AMD driver for graphics workloads with ROCm libraries and development tools.

#### Prerequisites

**System requirements:**

- Ubuntu 24.04 with kernel 6.8
- Disk size greater than 64GB
- Desktop environment (for graphics workloads, use Ubuntu Desktop ISO)

#### Pre-installation steps

Complete the following pre-installation steps.

1. Update package list:

   ```bash
   sudo apt update
   ```

1. Install Python packages:

   ```bash
   sudo apt install python3-setuptools python3-wheel
   ```

1. Add user to required groups:

   ```bash
   sudo usermod -a -G render,video $LOGNAME
   ```

1. Install kernel headers:

   ```bash
   sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
   ```

#### Install AMD driver with graphics support

Follow these steps to install the AMD driver with graphics support.

1. Upgrade the system:

   ```bash
   sudo apt upgrade
   ```

1. Download the installer:

   ```bash
   wget https://repo.radeon.com/.hidden/4beb847a345ee8f56d60594fcd2babe1/amdgpu-install/6.4.2.2/ubuntu/noble/amdgpu-install_6.4.2.2.60402-1_all.deb
   ```

1. If a previous driver exists, remove it:

   ```bash
   sudo amdgpu-uninstall
   sudo apt remove amdgpu-install --purge
   ```

1. Install the new driver:

   ```bash
   sudo apt-get install ./amdgpu-install_6.4.2.2.60402-1_all.deb
   sudo amdgpu-setup -b https://repo.radeon.com/.hidden/4beb847a345ee8f56d60594fcd2babe1
   sudo sed -i 's|https://repo\.radeon\.com/amdgpu/6\.4\.2|https://repo.radeon.com/.hidden/4beb847a345ee8f56d60594fcd2babe1/amdgpu/6.4.2.2|' /etc/apt/sources.list.d/amdgpu-proprietary.list
   sudo gpg --keyserver keyserver.ubuntu.com --recv-keys 9386B48A1A693C5C
   sudo gpg --export --armor 9386B48A1A693C5C | sudo tee /etc/apt/trusted.gpg.d/amdgpu.asc
   sudo amdgpu-install --usecase=workstation,rocm,amf --opencl=rocr --vulkan=pro --no-32 --accept-eula
   ```

1. Load the driver:

   ```bash
   sudo modprobe amdgpu
   ```

1. Verify installation:

   ```bash
   sudo dmesg | grep amdgpu
   ```

To remove a blocklist, see [Enable automatic loading on reboot](#step-4-enable-automatic-loading-on-reboot).

---

#### X11 remote server configuration

After installing the graphics driver, follow these steps to configure a virtual display with hardware acceleration for remote access.

##### Step 1: Install required packages

```bash
sudo apt install net-tools
sudo apt install x11vnc
```

##### Step 2: Configure GDM3

Follow these steps to configure GDM3.

1. Edit the GDM3 configuration:

   ```bash
   sudo vim /etc/gdm3/custom.conf
   ```

1. Modify to include:

   ```ini
   [daemon]
   AutomaticLoginEnable=true
   AutomaticLogin=your_username
   WaylandEnable=false
   ```

1. Restart GDM3:

   ```bash
   sudo systemctl restart gdm3
   ```

##### Step 3: Configure X11

Follow these steps to configure X11.

1. Get your GPU's Bus ID:

   ```bash
   lspci -d 1002: | awk '{print $1}'
   ```

1. Convert the hex Bus ID to decimal format. For example, `3a9e:00:00.0` becomes `3841536`.

1. Edit the X configuration file `/usr/share/X11/xorg.conf.d/00-amdgpu.conf`:

   ```
   Section "Device"
       Identifier "Card0"
       Driver "amdgpu"
       BusID "PCI:3841536:0:0"
   EndSection
   
   Section "Screen"
       Identifier "Screen0"
       Device "Card0"
       Monitor "Monitor0"
   EndSection
   ```

1. Edit `/usr/share/X11/xorg.conf.d/10-amdgpu.conf`:

   ```
   Section "OutputClass"
       Identifier "Card0"
       MatchDriver "amdgpu"
       Driver "amdgpu"
       Option "PrimaryGPU" "yes"
   EndSection
   ```

1. Reboot and load the driver:

   ```bash
   sudo reboot
   ```

1. After reboot, run the following commands:

   ```bash
   sudo systemctl stop gdm
   sudo modprobe amdgpu
   sudo systemctl start gdm
   ```

##### Step 4: Start VNC server

To start the server, run the following command.

```bash
x11vnc --forever -find
```

> [!NOTE] 
> X11 configuration only works with Ubuntu Desktop images, not Server images.

---

#### Troubleshooting

##### Downgrade to kernel 6.8

You can downgrade to 6.8 for compatibility by following these steps.

1. Check loaded kernels:

   ```bash
   dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'
   ```

1. Install kernel 6.8:

   ```bash
   sudo apt install linux-image-6.8.0-1025-azure
   ```

1. Edit GRUB:

   ```bash
   sudo vim /etc/default/grub
   ```

1. Set kernel 6.8 as default:

   ```
   GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.8.0-1025-azure"
   ```

1. Update GRUB and reboot:

   ```bash
   sudo update-grub
   sudo reboot
   ```

1. Verify kernel version:

   ```bash
   uname -a
   ```
1. Remove kernel references to 6.17:
   ```bash
   sudo apt purge linux-headers-6.17.0-1011-azure linux-image-6.17.0-1011-azure linux-modules-6.17.0-1011-azure linux-modules-extra-6.17.0-1011-azure
   ```
---

#### Uninstalling the AMD GPU driver

To completely remove the AMD GPU driver, run the following commands:

```bash
dkms status
sudo amdgpu-install --uninstall
sudo amdgpu-uninstall
sudo apt autoremove --purge amdgpu-install
sudo reboot
```

Verify removal:

```bash
dkms status
```

## NGads V620-series ##

To use the GPU on an Azure NGads V620-series Linux VM, manually install the AMD GPU driver by following the steps in this guide. The AMD GPU Driver Extension doesn't currently support NGads V620-series VMs.

Only GPU drivers published by Microsoft are supported on NGads V620 VMs. Don't install GPU drivers from other sources.

> [!NOTE]
> Currently, Azure provides installation instructions for:
> - Ubuntu 24.04

To obtain the Linux distribution and kernel information, use the following command:

```bash
$ $ uname -a && cat /etc/*release
```

Output

```bash
Linux 6.8.0-1025-azure #30-Ubuntu SMP Mon Mar 10 17:36:50 UTC 2025 x86_64 x86_64 
x86_64 GNU/Linux
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.3 LTS"
PRETTY_NAME="Ubuntu 24.04.3 LTS"
```

### Prerequistes
Before you install the AMD GPU driver, verify that the VM meets these requirements:
 
- Run Ubuntu 24.04 LTS with a `6.8.*-azure` kernel. Check the kernel version by running `uname -r`.
- Use an OS disk of at least **100 GB**
- If Secure Boot is enabled for the VM, disable it before installing the driver if the driver module isn’t signed for Secure Boot. In the Azure portal, stop (deallocate) the VM, open **Configuration**, clear **Enable Secure Boot**, save the change, and start the VM.

#### 1.1 Setting permissions for groups

Add yourself to the render and video group using the following command

```bash
$ sudo usermod -a -G render,video $LOGNAME
```

#### 1.2 Kernel headers and development packages

The driver package uses Dynamic Kernel Module Support (DKMS) to build the amdgpu-dkms module for 
installed kernels. This requires the installation of Linux kernel headers and modules for each kernel. 
These are usually installed automatically with the kernel but may need to be installed manually if you 
have multiple kernel versions or have downloaded kernel images without the meta-packages

```bash
$ sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
```

#### 1.3 Verifying GPU card(s) in Linux® 

The output should show the GPU card. 

```bash
$ sudo lspci -d 1002:
```
```bash
0002:00:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Navi 21 [Radeon 
Pro V620 MxGPU]
```

#### 1.4 Virtual Machine Update 
On an NGads-V620 GPU Linux instance running Ubuntu 24.04 OS, run the update:

```bash
$ sudo apt update
```
### AMD driver installation 

#### 2.1 Driver installation for graphics and multimedia workloads (with virtual display support) 

Before you install the graphics driver, run the upgrade command: 

```bash
$ sudo apt upgrade
```
#### 2.1.1 Download amdgpu installer
To download and install the amdgpu-install script, use the following command.
```bash
$ wget https://repo.radeon.com/.hidden/4beb847a345ee8f56d60594fcd2babe1/amdgpu-install/6.4.2.2/ubuntu/noble/amdgpu-install_6.4.2.2.60402-1_all.deb
```
#### 2.1.2 Install the installer package
> [!NOTE]
> If you previously installed the amdgpu driver, run `sudo amdgpu-uninstall` and `sudo apt remove amdgpu-install --purge` before installing the new driver. 

```bash
$ sudo apt-get install ./amdgpu-install_6.4.2.2.60402-1_all.deb
$ sudo amdgpu-setup -b https://repo.radeon.com/.hidden/4beb847a345ee8f56d60594fcd2babe1
$ sudo sed -i 
's|https://repo\.radeon\.com/amdgpu/6\.4\.2|https://repo.radeon.com/.hidden/4beb847a345ee8f56d6
0594fcd2babe1/amdgpu/6.4.2.2|' /etc/apt/sources.list.d/amdgpu-proprietary.list
$ sudo gpg --keyserver keyserver.ubuntu.com --recv-keys 9386B48A1A693C5C
$ sudo gpg --export --armor 9386B48A1A693C5C | sudo tee /etc/apt/trusted.gpg.d/amdgpu.as
```
Install the driver: 
```bash
$ sudo amdgpu-install --usecase=workstation,rocm,amf --opencl=rocr --vulkan=pro --no-32 --accept-eula
```

#### 2.2 Load amdgpu driver 

```bash
$ sudo modprobe amdgpu
```
> [!NOTE]
> You must load the amdgpu driver on every boot.

Review the output of `dmesg | grep amdgpu` to confirm that the GPU driver loads and initializes successfully.

```bash
$ sudo dmesg | grep amdgpu
[ 1414.882361] [drm] amdgpu kernel modesetting enabled.
[ 1414.882366] [drm] amdgpu version: 6.12.12
[ 1414.882491] amdgpu: Virtual CRAT table created for CPU
[ 1414.882508] amdgpu: Topology: Add CPU node
[ 1414.885713] amdgpu 0002:00:00.0: enabling device (0000 -> 0002)
[ 1415.262097] amdgpu 0002:00:00.0: amdgpu: detected ip block number 0 
<soc21_common>
[ 1415.277347] amdgpu 0002:00:00.0: amdgpu: Fetched VBIOS from VRAM BAR
[ 1415.277352] amdgpu: ATOM BIOS: 113-D7190300-104
[ 1415.277566] amdgpu 0002:00:00.0: amdgpu: CP RS64 enable
[ 1415.278988] amdgpu 0002:00:00.0: amdgpu: Trusted Memory Zone (TMZ) 
feature not supported
[ 1415.279039] amdgpu 0002:00:00.0: amdgpu: VRAM: 25712M 0x0000008000000000 
- 0x0000008646FFFFFF (25712M used)
[ 1415.279042] amdgpu 0002:00:00.0: amdgpu: GART: 512M 0x00007FFF00000000 -
0x00007FFF1FFFFFFF
[ 1415.279219] [drm] amdgpu: 25712M of VRAM memory ready
[ 1415.279222] [drm] amdgpu: 64403M of GTT memory ready.
[ 1415.461922] amdgpu 0002:00:00.0: amdgpu: smu driver if version = 
0x0000003d, smu fw if version = 0x00000040, smu fw program = 0, smu fw 
version = 0x00505000 (80.80.0)
[ 1415.461930] amdgpu 0002:00:00.0: amdgpu: SMU driver if version not 
matched
[ 1415.484619] amdgpu 0002:00:00.0: amdgpu: SMU is initialized successfully!
[ 1415.567480] amdgpu: HMM registered 25712MB device memory
[ 1415.568476] kfd kfd: amdgpu: Allocated 3969056 bytes on gart
[ 1415.568495] kfd kfd: amdgpu: Total number of KFD nodes to be created: 1
```

#### 2.2.1 Run AMD-SMI to confirm the driver loads successfully

```bash
$ amd-smi monitor
```

Output
```bash
GPU POWER GPU_T MEM_T GFX_CLK GFX% MEM% ENC% DEC% VRAM_USAGE
 0 11 W 37 °C 54 °C 1250 MHz 83 % 2 % N/A 0 % 0.2/ 25.1 GB
```
