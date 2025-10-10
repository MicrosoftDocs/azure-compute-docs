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
ms.date: 05/13/2025
ms.author: padmalathas
# Customer intent: As a cloud engineer managing Linux VMs with AMD GPUs, I want to install the necessary AMD GPU drivers and configure my NVv5-V710 instances, so that I can optimize performance for AI and graphics workloads.
---

# Install AMD GPU drivers on NVads V710-series Linux VMs

**Applies to:** :heavy_check_mark: Linux VMs

> [!Note]
> Azure currently provides installation instructions for Ubuntu 22.04 and Ubuntu 24.04. For other Linux distributions and the latest updated guide on setting up ROCm drivers, please see AMDs page - [Quick start installation guide - ROCm installation(Linux)](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.3.3/install/quick-start.html), for all other ROCm versions, see [ROCm release history - ROCm Documentation](https://rocm.docs.amd.com/en/latest/release/versions.html#rocm-release-history)

## NVads V710-series

To utilize the GPU capabilities of the new Azure *NVads V710-series VMs* running Linux, you need to install the AMD GPU drivers. The [AMD GPU Driver Extension](../extensions/hpccompute-amd-gpu-linux.md) simplifies the installation process for AMD GPU drivers on *NVv710-series VMs*. You can manage this extension through the Azure portal, Azure PowerShell, or Azure Resource Manager(ARM) templates. For detailed information on supported operating systems and deployment steps, see [AMD GPU Driver Extension](../extensions/hpccompute-amd-gpu-linux.md) documentation.

The [marketplace image](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/amdinc1746636494855.nvv5_v710_linux_rocm_image?tab=Overview) is preloaded with the AMD GPU driver and can speed up the process when bringing up the VM. 

This article outlines the supported operating systems, drivers, and provides installation and verification steps for **Ubuntu**.

### ROCm

Here are the steps for installing the AMD Linux Driver to harness the capabilities of the AMD Radeon PRO V710 GPU on an NVv5-V710 GPU Linux instance provided by Microsoft Azure. Subsequent sections provide detailed Linux driver installation instructions for users who wish to perform inference using ROCm on the NVv5-V710 GPU Linux instance.

## Step 1: Linux Driver Installation
**Supported Linux Distros**

Confirm the system has a supported Linux version. 

To obtain the Linux distribution information, use the following command: 


```
$ cat /etc/*release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=XX
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu"
PRETTY_NAME="Ubuntu LTS" 
```

Confirm that your Linux distribution matches a [supported distribution](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/reference/system-requirements.html#supported-distributions). 

**Supported Linux Kernel**

To check the kernel version of your Linux system, type the following command: 


```
$ uname -srmv

Linux 5.XX.0-XX-generic #86-Ubuntu SMP Mon Jul 10 16:07:21 UTC 2023 x86_64
```

## Step 2: Pre-configuration
> [!Note]
> The disk size must be greater than 64GB to ensure optimal performance and compatibility.

**Verify the GPU card**

Verify the output of the GPU card, using


```
$ sudo lspci -d 1002:7461` 
c3:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 7461
```
> [!NOTE]
> The Virtual Function Device ID 7461 confirms that the Virtual Machine is configured with the AMD Radeon PRO V710 GPU.

### AMD Driver Installation
#### Installation

The following steps demonstrate the use of the amdgpu-install script for a single-version driver installation. To install the latest ROCm driver, run the following commands on your terminal:
<details>
  <summary><strong>Ubuntu 22.04</strong></summary>

  ```bash
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
wget https://repo.radeon.com/amdgpu-install/6.4.3/ubuntu/jammy/amdgpu-install_6.4.60403-1_all.deb
sudo apt install ./amdgpu-install_6.4.60403-1_all.deb
sudo apt update
sudo apt install amdgpu-dkms rocm
  ```
  </details> 
  <details>
  <summary><strong>Ubuntu 24.04</strong></summary>









```bash
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
wget https://repo.radeon.com/amdgpu-install/7.0.1/ubuntu/noble/amdgpu-install_7.0.1.70001-1_all.deb
sudo apt install ./amdgpu-install_7.0.1.70001-1_all.deb
sudo apt update
sudo apt install amdgpu-dkms rocm
```

  </details> 

>[!NOTE]
> Azure currently supports Ubuntu 22.04 and Ubuntu 24.04, for all other Linux distros refer to [AMD's documentation](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.3.3/install/quick-start.html).

#### Load amdgpu driver

```bash
$ sudo modprobe amdgpu
```

Review the output of **" dmesg | grep amdgpu "** to confirm that the GPU driver is loaded and initialized successfully.

```bash
$ sudo dmesg | grep amdgpu 
[ 66.177373] [drm] amdgpu kernel modesetting enabled. 
[ 66.177379] [drm] amdgpu version: 6.7.0 
[ 66.177623] amdgpu: Virtual CRAT table created for CPU 
[ 66.177653] amdgpu: Topology: Add CPU node 
[ 66.184259] amdgpu 045b:00:00.0: enabling device (0000 -> 0002) 
[ 66.670226] [drm] add ip block number 5 <amdgpu_vkms> 
[ 66.685726] amdgpu 045b:00:00.0: amdgpu: Fetched VBIOS from VRAM BAR 
[ 66.685733] amdgpu: ATOM BIOS: 113-D7190300-104 
[ 66.689542] amdgpu 045b:00:00.0: amdgpu: CP RS64 enable
```

Run AMD-SMI to confirm the driver is loaded successfully using `$ amd-smi monitor` <br>
```bash
GPU  POWER  GPU_TEMP  MEM_TEMP  GFX_UTIL  GFX_CLOCK  MEM_UTIL  MEM_CLOCK  ENC_UTIL  ENC_CLOCK  DEC_UTIL  DEC_CLOCK     THROTTLE  SINGLE_ECC  DOUBLE_ECC  PCIE_REPLAY  VRAM_USED  VRAM_TOTAL   PCIE_BW 

  0   11 W     43 °C     58 °C      84 %   1814 MHz       1 %     96 MHz       N/A    812 MHz       N/A    512 MHz  UNTHROTTLED           0           0            0     227 MB    25476 MB  N/A Mb/s
```

#### Enable the driver

To automatically load the `amdgpu` driver on every reboot of the VM, we need to remove any blacklist entry that is preventing it from loading automatically.

* Search for any file that containing blacklisted amdgpu, using `$ grep amdgpu /etc/modprobe.d/* -rn`. The output must render a string similar to <br>
*/etc/modprobe.d/blacklist.conf:10:blacklist amdgpu*
* Remove the blacklist from the listed file, using `$ sudo nano /etc/modprobe.d/blacklist.conf` and delete the line with *blacklist amdgpu*.
* Update the initramfs to apply changes on the next boot, using `$ sudo update-initramfs -uk all`

* Reboot the system to load the updated configuration using `$ sudo reboot`. After rebooting, ensure that amdgpu driver isn't blacklisted and it's available for use.

### Graphics+ROCM

### 1. Installation Guide
#### 1.1 Introduction

Here are the steps for installing the AMD Linux Driver to use the power of the AMD Radeon™ PRO V710 GPU on an NVv5-V710 GPU Linux instance offered by Microsoft Azure. The Linux Driver installation also includes installing the ROCm™ Libraries, graphic libraries, and Development Tools. Subsequent sections of the document thoroughly discuss the driver installation for the graphics use case.

### 2. Linux Driver Prerequisites
#### 2.1 Supported Linux Distros

The AMD Linux Driver software supports the following Linux distributions:

| Linux Distribution | Kernel Version | Supported |
|--------------------|----------------|-----------|
| Ubuntu® 22.04      | 6.5            | ✅ Yes   |

Confirm the system has a supported Linux version. To obtain the Linux distribution information, use the following command:
```bash
$ uname -a && cat /etc/*release
```
Output is similar to the following example
``` bash
Linux amd-Virtual-Machine 6.5#18~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Feb  7 
11:40:03 UTC 2 x86_64 x86_64 x86_64 GNU/Linux 
DISTRIB_ID=Ubuntu 
DISTRIB_RELEASE=22.04 
DISTRIB_CODENAME=jammy 
DISTRIB_DESCRIPTION="Ubuntu 22.04" 
PRETTY_NAME="Ubuntu 22.04 LTS" 
```
Ensure your Linux distribution and kernel version are listed in the table above.  
>[!Note]
>Refer to the troubleshooting section at the end of the document for instructions on how to set the 
>6.5 kernel as the default (at every boot time) on the NvV5 V710 GPU instance. 

>[!Note]
>If you plan to run the graphics workload, use the Linux distribution with graphics enabled (e.g., Ubuntu-22.04-desktop-amd64.iso).
### 3. Troubleshooting
This section outlines troubleshooting techniques to address issues that may arise during the driver installation process.
If you're using the Kernel 6.8, follow the below steps to downgrade to kernel 6.5.

##### Check Loaded Kernels: 

Run the following command to list the loaded kernels
```bash
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'
```
Review the output to see the currently loaded kernels.

##### Install Kernel 6.5: 

If Kernel 6.5 isn't loaded, install it using
```bash
sudo apt install linux-image-6.5.0-1025-azure
```
##### Purge Kernels Above 6.5: 

Use the following command to purge kernels above version 6.5
```bash
sudo apt purge linux-headers-6.8.0-1025-azure linux-image-6.8.0-1025-azure linux-modules-6.8.0-1025-azure
```
##### Verify Kernel Version: 

Verify that only Kernel 6.5 is present by running
```bash
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'
```
The output should be similar to the following example:
```bash
linux-image-6.5.0-1025-azure
linux-headers-6.5.0-1025-azure
linux-modules-6.5.0-1025-azure
```
##### Loading Kernel 6.5 by default on boot:
When the NVv5-V710 GPU Linux instance is launched, the OS boots to the 6.8.0-1015-azure kernel instead of the 6.5.0-1025-azure kernel. The GRUB settings need to be modified to boot into the 6.5.0-1025-azure kernel. To check the currently installed kernels, use the following command
```bash
$ dpkg --list | egrep -i --color 'linux-image' | awk '{ print $2 }'
```
Output is similar to the following example
```bash
Linux-image-6.5.0-1025-azure 
linux-image-6.8.0-1015-azure 
linux-image-azure
```
Open the GRUB settings and change GRUB_DEFAULT="0" to GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.5.0-1025-azure"
```bash
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.5.0-1025-azure"
```
##### Update GRUB and Reboot:
Update GRUB and reboot the system using
```bash
sudo update-grub sudo reboot
```
##### Validate Kernel Version:
After rebooting, validate the kernel version using
```bash
uname -a
```

### 4. Prerequisites
>[!Note]
> The disk size must be greater than 64GB to ensure optimal performance and compatibility.

#### 4.1 Update the package list
To ensure you have the latest information on the newest versions of packages and their dependencies.
```bash
sudo apt update
```
#### 4.2 Install Python Setuptools and wheel
These packages are essential for building and distributing Python packages.
```bash
$ sudo apt install python3-setuptools python3-wheel
```

#### 4.3 Setting Permissions for groups

Add yourself to the render and video group using the following command:
```bash
$ sudo usermod -a -G render,video $LOGNAME
```

#### 4.4 Kernel headers and development packages

The driver package uses Dynamic Kernel Module Support (DKMS) to build the amdgpu-dkms module for installed kernels. This requires the installation of Linux kernel headers and modules for each kernel. These packages are installed automatically with the kernel. However, if you use multiple kernel versions or download kernel images without the meta-packages, you might need to install them manually.

```bash
$ sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
```

#### 4.5 Verifying GPU Card in Linux&reg;

The output should have the GPU card.

```Bash
$ sudo lspci -d 1002:7461
c3:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 7461
```

>[!NOTE]
> 7461 is the Virtual Function Device ID. This confirmation indicates that the Virtual Machine is configured with the AMD Radeon&trade; PRO V710 GPU.

#### 4.6 Virtual Machine Update

On an NVv5-V710 GPU Linux instance running Ubuntu 22.04 OS, run the update: 
```bash
$ sudo apt update
```
#### 4.7 Blacklist amdgpu Driver

Before installing the latest AMD Linux driver, it's important to **blacklist** the default amdgpu driver. The default driver, present in Linux distributions like Ubuntu or RHEL, isn't certified for use with the **AMD Radeon™ PRO V710 GPU** on an **NVv5-V710 GPU Linux instance**. The driver optimized for Azure NVv5-V710 GPU workloads should be used instead.

##### Check if the Driver is Already Blacklisted

To check if the amdgpu driver is already blacklisted, run the following command:

```bash
grep amdgpu /etc/modprobe.d/* -rn
```

If the driver is blacklisted, you don't need to modify anything else.
Be careful with entries that start with #blacklist amdgpu –  this indication means that the driver isn't blacklisted


##### Disable the amdgpu Driver
 If the `amdgpu` driver is **not already blacklisted** follow the steps to blacklist it.

Open the /etc/modprobe.d/blacklist.conf file to edit:
```bash 
sudo vim /etc/modprobe.d/blacklist.conf
```

Add the following line to blacklist the amdgpu driver:
```bash
blacklist amdgpu
```

After updating the blacklist.conf file, run the following command to apply the changes:
```bash
$ sudo update-initramfs -uk all
```
This command ensures the changes take effect and the driver is properly blacklisted.
#### 4.8 Reboot

After you restart the virtual machine, the default **amdgpu driver** in Ubuntu Linux distributions shouldn't load because it was previously blacklisted. To confirm that the driver isn't loaded, use the following command:

```bash
lsmod | grep amdgpu
```

### 5. AMD Driver Installation
#### 5.1 Installation

The following steps demonstrate the use of the amdgpu-install script for a single-version driver installation. These instruction install ROCm version **6.1.4** on **Ubuntu 22.04 (Jammy)**.
```bash
# Upgrade the system
sudo apt upgrade

# Download amdgpu installer
wget -N -P /tmp/ https://repo.radeon.com/amdgpu-install/6.1.4/ubuntu/jammy/amdgpu-install_6.1.60104-1_all.deb

# If an AMDGPU driver was previously installed, uninstall it
sudo amdgpu-uninstall
sudo apt remove amdgpu-install --purge

# Install the installer package
sudo apt-get install /tmp/amdgpu-install_6.1.60104-1_all.deb

# Install the driver
sudo amdgpu-install --usecase=workstation,rocm,amf --opencl=rocr --vulkan=pro --no-32 --accept-eula
```
#### 5.2 Load amdgpu driver
After installation, load the amdgpu Driver

```bash
$ sudo modprobe amdgpu
```

You can verify the driver is loaded and initialized successfully with
```bash
sudo dmesg | grep amdgpu
```
Example output:
```bash
[ 66.177373] [drm] amdgpu kernel modesetting enabled. 
[ 66.177379] [drm] amdgpu version: 6.7.0 
[ 66.177623] amdgpu: Virtual CRAT table created for CPU 
[ 66.177653] amdgpu: Topology: Add CPU node 
[ 66.184259] amdgpu 045b:00:00.0: enabling device (0000 -> 0002) 
[ 66.670226] [drm] add ip block number 5 <amdgpu_vkms> 
[ 66.685726] amdgpu 045b:00:00.0: amdgpu: Fetched VBIOS from VRAM BAR 
[ 66.685733] amdgpu: ATOM BIOS: 113-D7190300-104 
[ 66.689542] amdgpu 045b:00:00.0: amdgpu: CP RS64 enable
```
#### 5.2.1 Enable the driver

To automatically load the `amdgpu` driver on every reboot of the VM, we need to remove any blacklist entry that is preventing it from loading automatically.

##### Search for the blacklist entry

Run the following command to find any file that contains `blacklist amdgpu`:

```bash
grep amdgpu /etc/modprobe.d/* -rn
```
If the driver is blacklisted, you see output similar to:
```bash
/etc/modprobe.d/blacklist.conf:10:blacklist amdgpu
```
##### Remove the blacklist line
Open the file listed in the output:
```bash
sudo nano /etc/modprobe.d/blacklist.conf
```
Delete the line that says:
```bash
blacklist amdgpu
```
Save and exit the file
##### Update initramfs
Update the initramfs so the changes are applied on the next boot:
```bash
sudo update-initramfs -uk all
```
##### Reboot the system
Reboot the machine to load the updated configuration:
```bash
sudo reboot
```
After rebooting, the `amdgpu` driver should no longer be blacklisted and will be available for use.

Run AMD-SMI to confirm the driver is loaded successfully 

```bash
$ amd-smi monitor
```
```bash
GPU  POWER  GPU_TEMP  MEM_TEMP  GFX_UTIL  GFX_CLOCK  MEM_UTIL  MEM_CLOCK  ENC_UTIL  ENC_CLOCK  DEC_UTIL  DEC_CLOCK     THROTTLE  SINGLE_ECC  DOUBLE_ECC  PCIE_REPLAY  VRAM_USED  VRAM_TOTAL   PCIE_BW 

  0   11 W     43 °C     58 °C      84 %   1814 MHz       1 %     96 MHz       N/A    812 MHz       N/A    512 MHz  UNTHROTTLED           0           0            0     227 MB    25476 MB  N/A Mb/s
```
### 6. x11 Remote Server Configuration
After installing the AMD Graphics Linux drivers with, the default graphical interface (Xserver) doesn't utilize hardware acceleration As a solution, a virtual display should be created with hardware acceleration enabled that can be used for remote access (x11vnc). The following steps walk through the virtual display setup:
#### 6.1 Install Required Packages
Install `x11vnc` and `net-tools`
```bash
$ sudo apt install net-tools 
$ sudo apt install x11vnc 
```
#### 6.2 Update GDM3 Custom Configuration
Edit the GDM3 configuration file to:

   -Disable Wayland (which doesn't support x11vnc)

   -Enable automatic login (so a graphical session is available at boot)

Open the configuration file with:

```bash
$ sudo vim /etc/gdm3/custom.conf
```

After modification the file looks like this
```bash
# GDM configuration storage 
 
[daemon] 
AutomaticLoginEnable=true 
AutomaticLogin=amd 
 
# Uncomment the line below to force the login screen to use Xorg 
WaylandEnable=false 
 
# Enabling automatic login 
 
# Enabling timed login 
#  TimedLoginEnable = true 
#  TimedLogin = user1 
#  TimedLoginDelay = 10 
 
[security] 
 
[xdmcp] 
 
[chooser] 
 
[debug] 
# Uncomment the line below to turn on debugging 
# More verbose logs 
# Additionally lets the X server dump core if it crashes 
#Enable=true
```
#### 6.3 Reboot and Restart gdm3
After reboot, restart the gdm3 by following command
```bash
$ sudo systemctl restart gdm3
```
#### 6.4 Modify X Configuration
##### 6.4.1 Getting Bus ID
The BusID of the AMD Radeon™ PRO V710 GPU must be manually added to the X11 configuration file. 
To get the BusID, follow the steps
```bash
$ lspci -d 1002: | awk '{print $1}' 
3a9e:00:00.0
```
>[!Note]
>Convert BusID of GPU from HEX to Decimal, e.g., "3a9e:00:00.0", convert HEX "3a9e00" into DEC "3841536" 

##### 6.4.2 Updating X Configuration to add Device and Screen
Furthermore, modify the “Screen” section to incorporate this device.

To ensure the driver configuration is correct, modify /usr/share/X11/xorg.conf.d/00-amdgpu.conf to match the content.
>[!Note]
>Make sure to update BusID as per your system configuration (as shown in the previous step)
```bash
Section "OutputClass" 
        Identifier "AMDgpu" 
        MatchDriver "amdgpu" 
        Driver "amdgpu" 
EndSection 
 
Section "Files" 
        ModulePath "/opt/amdgpu-pro/lib/xorg/modules" 
        ModulePath "/opt/amdgpu/lib/xorg/modules" 
        ModulePath "/usr/lib/xorg/modules" 
EndSection 
 
Section "Device" 
    Identifier  "Card0" 
    Driver      "amdgpu" 
    BusID  "PCI:3841536:0:0" 
EndSection 
 
Section "Screen" 
    Identifier "Screen0" 
    Device     "Card0" 
    Monitor    "Monitor0" 
    SubSection "Display" 
        Viewport   0 0 
        Depth     1 
    EndSubSection 
    SubSection "Display" 
        Viewport   0 0 
        Depth     4 
    EndSubSection 
    SubSection "Display" 
        Viewport   0 0 
        Depth     8 
    EndSubSection 
    SubSection "Display" 
        Viewport   0 0 
        Depth     15 
    EndSubSection 
    SubSection "Display" 
        Viewport   0 0 
        Depth     16 
    EndSubSection 
    SubSection "Display" 
        Viewport   0 0 
        Depth     24 
EndSubSection 
EndSection
```
Also modify /usr/share/X11/xorg.conf.d/10-amdgpu.conf to match the following section
```bash
Section "OutputClass" 
Identifier "Card0" 
MatchDriver "amdgpu" 
Driver "amdgpu" 
Option "PrimaryGPU" "yes" 
EndSection 
```
#### 6.5 Reboot

After installation, reboot the virtual machine to apply changes:

```bash
sudo reboot
```
#### 6.6 Load Driver
Once the system is backup, load the amdgpu driver using the following commands:
```bash
$ sudo systemctl stop gdm   
$ sudo modprobe amdgpu  
$ sudo systemctl start gdm 
```
> These commands temporarily stop and restart the GNOME Display Manager(gdm) to load the driver correctly. Make sure you save your work before running them
#### 6.7 Running x11vnc
To start the VNC server and automatically find the correct display and authentication, use the following command:
```bash
x11vnc --forever -find
```
This command searches for the active X display and user credentials (XAUTH) automatically.
>[!Note]
>This setup is only compatible with the supported Ubuntu Desktop image. These instructions do not work for Ubuntu Server images. 

### Uninstallation Steps
If you need to uninstall the existing amdgpu driver, follow these steps:

Check DKMS status:
```bash
dkms status
```
Uninstall the amdgpu driver:
```bash
sudo amdgpu-install --uninstall
sudo amdgpu-uninstall
```
Remove the amdgpu installation package:
```bash
sudo apt autoremove --purge amdgpu-install
```
Reboot the system:
```bash
sudo reboot
```
Check DKMS status again to ensure the driver is uninstalled:
```bash
dkms status
```
This command ensures the old amdgpu driver is fully removed from the system before installing the new driver.
