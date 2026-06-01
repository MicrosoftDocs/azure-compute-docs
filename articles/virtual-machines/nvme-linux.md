---
author: MelissaHollingshed
ms.author: mehollin
ms.date: 05/29/2026
ms.topic: how-to
ms.service: azure-virtual-machines
title: Convert SCSI to NVMe for Linux and Windows VMs
description: Learn how to convert SCSI to NVMe by using Linux.
ms.custom: sfi-image-nochange, linux-related-content, windows-related-content
# Customer intent: As a cloud solutions architect, I want to convert virtual machines running Linux from SCSI to NVMe storage, so that I can enhance their performance and scalability while ensuring compatibility with modern cloud infrastructure.
---

# Convert Linux and Windows VMs from SCSI to NVMe

Azure virtual machines (VMs) support two types of storage interfaces: Small Computer System Interface (SCSI) and NVMe. The SCSI interface is a legacy standard that provides physical connectivity and data transfer between computers and peripheral devices. NVMe is similar to SCSI in that it provides connectivity and data transfer, but NVMe is a faster and more efficient interface for data transfer between servers and storage systems.

In this article, you learn how to convert Azure virtual machines (VMs) running Linux or Windows from a Small Computer System Interface (SCSI) disk controller to NVM Express (NVMe) by using Azure Boost and the Azure NVMe Conversion script.

Azure continues to support the SCSI interface on the versions of VM offerings that provide SCSI storage. However, not all new VM series have SCSI storage as an option going forward.

## What's changing for your VM?

Changing the host interface from SCSI to NVMe doesn't change the remote storage (OS disk or data disks), but it changes the way the operating system sees the disks.

| Disk | SCSI-enabled VM | NVMe VM with SCSI temporary disk (for example, Ebds_v5) | NVMe VM with NVMe temporary disk |
| ---- | --------------- | ------------------------------------------- | ----------------------------- |
| OS disk | /dev/sda | /dev/nvme0n1 | /dev/nvme0n1 |
| Temporary disk | /dev/sdb | /dev/sda | /dev/nvme1n1 |
| First data disk | /dev/sdc | /dev/nvme0n2 | /dev/nvme0n2 |

> [!TIP]
> Some VM types have more than one temporary disk (for example, E64ds_v6)

Converting your Azure VM from SCSI to NVMe by using Azure Boost can help you take full advantage of these performance improvements and maintain a competitive edge in the cloud computing landscape.

## Migrate a Linux VM from SCSI to NVMe

To migrate from SCSI to NVMe, you need to follow these high-level steps:

1. Check if your virtual machine series supports NVMe.
2. Check your operating system for NVMe readiness.
3. Convert your virtual machine to NVMe.
4. Check your operating system.

### 1. Check if your virtual machine series supports NVMe

The supported virtual machines to support NVMe attached disks is described on the [Azure Boost overview site in the availability table](/azure/azure-boost/overview#current-availability).

### 2. Check your operating system for NVMe readiness

The operating system needs to support NVMe devices. For example, you need to prepare device drivers and *initrdm*, the temporary file system used during boot. You also need to validate the mount points of the file systems, because they check if you use the SCSI device name (/dev/sdX).

The migration script can automatically take care of these readiness checks for you when you use `-FixOperatingSystemSettings`.

#### 2.1 Check the controller type of the VM

##### Check the controller type by using PowerShell

```powershell
PS C:\Users\user1> $vm = Get-AzVM -name [your-vm-name]
PS C:\Users\user1> $vm.StorageProfile.DiskControllerType
SCSI
PS C:\Users\user1>
```

##### Check the controller type by using the Azure CLI

```powershell
$ az vm show --name [your-vm-name] --resource-group [your-resource-group-name]
{
"additionalCapabilities": {
...
 "storageProfile": {
 ...
   "diskControllerType": "SCSI",
 ...
 ```

##### Check the controller type by using the Azure portal

:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-2.png" alt-text="Screenshot of Azure portal to check controller.":::

#### 2.2 Prepare for migration

The migration script can automatically take care of the prerequisites when you use the `-FixOperatingSystemSettings` parameter.

If you want to manually take care of the required changes, validate that:

- NVMe modules are installed and part of initrd/initramfs.
- GRUB configuration includes the parameter `nvme_core.io_timeout=240`.
- `/etc/fstab` checks for devices.

Check with your OS vendor to cover all required commands to update initrd/initramfs.

##### 2.2.1 Prepare PowerShell

> [!TIP]
> This step isn't required when you run the script in Azure Cloud Shell.

1. Install PowerShell by using the [PowerShell documentation](https://aka.ms/powershell).

1. Connect to Azure by using `Connect-AzAccount`.

1. Select the correct subscription by using `Select-AzSubscription -Subscription [your-subscription-id]`.

1. Set the execution policy by using `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`.

##### 2.2.2 Download the script

You can download the script by using a PowerShell command:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" -OutFile ".\Azure-NVMe-Conversion.ps1"
```

#### 2.3 Run the migration

The script has multiple parameters available:

| Parameter | Description | Required |
| ------------------------------- | ----------------------------------------------------------------------------- | --------- |
| `-ResourceGroupName` | Resource group name of your VM. | Yes |
| `-VMName` | Name of your VM on Azure. | Yes |
| `-NewControllerType` | Storage controller type that the VM should be converted to (NVMe or SCSI). | Yes |
| `-VMSize` | Azure VM size that you want to convert the VM to. | Yes |
| `-StartVM` | Start the VM after conversion. | No |
| `-IgnoreSKUCheck` | Ignore the check of the VM size. | No |
| `-IgnoreWindowsVersionCheck` | Ignore the Windows version check. | No |
| `-FixOperatingSystemSettings` | Automatically fix the OS settings by using Azure run commands. | No |
| `-WriteLogfile` | Create a log file. | No |
| `-IgnoreAzureModuleCheck` | Don't run the check for installed Azure modules. | No |
| `-IgnoreOSCheck` | Don't check for OS readiness. The expectation is that the OS is ready. | No |
| `-SleepSeconds` | Time for Azure to settle changes before starting the VM. | No |

Here's a sample command:

```powershell
# Example usage
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName <your-RG> -VMName <your-VMname> -NewControllerType <NVMe/SCSI> -VMSize <new-VM-SKU> -StartVM -FixOperatingSystemSettings
```

> [!TIP]
> You can always revert back to SCSI. The script will share a command with you to directly revert to your original configuration.

##### 2.3.1 Sample output

```powershell
PS /home/philipp> ./NVMe-Conversion.ps1 -ResourceGroupName testrg -VMName testvm -NewControllerType NVMe -VMSize Standard_E4bds_v5 -StartVM -FixOperatingSystemSettings                                          
00:00 - INFO      - Starting script Azure-NVMe-Conversion.ps1
00:00 - INFO      - Script started at 06/27/2025 15:41:39
00:00 - INFO      - Script version: 2025062704
00:00 - INFO      - Script parameters:
00:00 - INFO      -   ResourceGroupName -> testrg
00:00 - INFO      -   VMName -> testvm
00:00 - INFO      -   NewControllerType -> NVMe
00:00 - INFO      -   VMSize -> Standard_E4bds_v5
00:00 - INFO      -   StartVM -> True
00:00 - INFO      -   FixOperatingSystemSettings -> True
00:00 - INFO      - Script Version 2025062704                                                                           
00:00 - INFO      - Module Az.Compute is installed and the version is correct.
00:00 - INFO      - Module Az.Accounts is installed and the version is correct.
00:00 - INFO      - Module Az.Resources is installed and the version is correct.
00:00 - INFO      - Connected to Azure subscription name: AG-GE-CE-PHLEITEN
00:00 - INFO      - Connected to Azure subscription ID: aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e
00:00 - INFO      - VM testvm found in Resource Group testrg
00:01 - INFO      - VM testvm is running
00:01 - INFO      - VM testvm is running Linux
00:01 - INFO      - VM testvm is running SCSI
00:02 - INFO      - Running in Azure Cloud Shell
00:02 - INFO      - Authentication token is a SecureString
00:02 - INFO      - Authentication token received
00:02 - INFO      - Getting available SKU resources
00:02 - INFO      - This might take a while ...
00:06 - INFO      - VM SKU Standard_E4bds_v5 is available in zone 1
00:06 - INFO      - Resource disk support matches between original VM size and new VM size.
00:06 - INFO      - Found VM SKU - Checking for Capabilities
00:06 - INFO      - VM SKU has supported capabilities
00:06 - INFO      - VM supports NVMe
00:06 - INFO      - Pre-Checks completed
00:06 - INFO      - Entering Linux OS section
00:37 - INFO      -    Script output: Enable succeeded: 
00:37 - INFO      -    Script output: [stdout]
00:37 - INFO      -    Script output: [INFO] Operating system detected: sles
00:37 - INFO      -    Script output: [INFO] Checking if NVMe driver is included in initrd/initramfs...
00:37 - INFO      -    Script output: [INFO] NVMe driver found in initrd/initramfs.
00:37 - INFO      -    Script output: [INFO] Checking nvme_core.io_timeout parameter...
00:37 - INFO      -    Script output: [INFO] nvme_core.io_timeout is set to 240.
00:37 - INFO      -    Script output: [INFO] Checking /etc/fstab for deprecated device names...
00:37 - INFO      -    Script output: [INFO] /etc/fstab does not contain deprecated device names.
00:37 - INFO      -    Script output: 
00:37 - INFO      -    Script output: [stderr]
00:37 - INFO      -    Script output: 
00:37 - INFO      - Errors: 0 - Warnings: 0 - Info: 7
00:37 - INFO      - Shutting down VM testvm
01:18 - INFO      - VM testvm stopped
01:18 - INFO      - Checking if VM is stopped and deallocated
01:19 - INFO      - Setting OS Disk capabilities for testvm_OsDisk_1_165411276cbe459097929b981eb9b3e2 to new Disk Controller Type to NVMe
01:19 - INFO      - generated URL for OS disk update:
01:19 - INFO      - https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/testrg/providers/Microsoft.Compute/disks/testvm_OsDisk_1_165411276cbe459097929b981eb9b3e2?api-version=2023-04-02
01:19 - INFO      - OS Disk updated
01:19 - INFO      - Setting new VM Size from Standard_E4s_v3 to Standard_E4bds_v5 and Controller to NVMe
01:19 - INFO      - Updating VM testvm
01:54 - INFO      - VM testvm updated
01:54 - INFO      - Start after update enabled for VM testvm
01:54 - INFO      - Waiting for 15 seconds before starting the VM
02:09 - INFO      - Starting VM testvm
03:31 - INFO      - VM testvm started
03:31 - INFO      - As the virtual machine got started using the script you can check the operating system now
03:31 - INFO      - If you have any issues after the conversion you can revert the changes by running the script with the old settings
03:31 - IMPORTANT - Here is the command to revert the changes:
03:31 - INFO      -    .\Azure-NVMe-Conversion.ps1 -ResourceGroupName testrg -VMName testvm -NewControllerType SCSI -VMSize Standard_E4s_v3 -StartVM
03:31 - INFO      - Script ended at 06/27/2025 15:45:11
03:31 - INFO      - Exiting
PS /home/philipp>
```

If you have challenges accessing the operating system afterward, try to check:

- The serial console for Linux operating systems.
- The screenshot from the operating system in the Azure portal.

If necessary, you can always revert to SCSI by using the command shown at the end of the script:

```powershell
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName testrg -VMName testvm -NewControllerType SCSI -VMSize Standard_E4s_v3 -StartVM
```

#### 2.4 Check the result

##### Check the result in the Azure portal

:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-3.png" alt-text="Screenshot of information for a virtual machine in the Azure portal.":::

##### Check the result in PowerShell

```Powershell
PS C:\Users> $vm = Get-AzVM -name [your-vm-name]
PS C:\Users> $vm.StorageProfile.DiskControllerType
NVMe
PS C:\Users>
```

### 3. Check your operating system

#### 3.1 Check devices

You can check the devices by using the `nvme` command. If the `nvme` command is missing, install the `nvme-cli` package:

`nvme list`

The output should show the OS disk and the data disks.
:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-4.png" alt-text="Screenshot of OS disks and data disks.":::

#### 3.2 Get the udev file for NVMe (optional)

On SCSI virtual machines, the udev rules integrated in the `waagent` Azure agent created links in `/dev/disk/azure/scsi1/lunX` to identify the data disks. Because SCSI isn't used anymore, the rules don't apply.

With one of the two available options to deploy NVMe-enabled udev rules, you see new symbolic links in the directory `/dev/disk/azure/data/by-lun`. This directory is the replacement for `/dev/disk/azure/scsi1`.

```bash
nvme-conversion-vm:/usr/lib/udev/rules.d # ls -l /dev/disk/azure/data/by-lun/
total 0
lrwxrwxrwx 1 root root 19 Jun 7 13:52 0 -> ../../../../nvme0n2
lrwxrwxrwx 1 root root 19 Jun 7 13:52 1 -> ../../../../nvme0n3
nvme-conversion-vm:/usr/lib/udev/rules.d #
```

##### Manual download of the udev file

To download the new udev rules file, use this command:
`curl https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/88-azure-nvme-data-disk.rules`. Then, run `udevadm control --reload-rules && udevadm trigger` to reload the udev rules.

##### Ready-to-install packages from GitHub

There are precompiled packages from [the GitHub collection of utilities and udev rules for Azure VMs](https://github.com/azure/azure-vm-utils) available on [Index of /results/cjp256/azure-vm-utils/](https://download.copr.fedorainfracloud.org/results/cjp256/azure-vm-utils/) for multiple distributions.

Multiple distributions already started to integrate the package. You can directly install it from their repositories.

| Distribution | Minimum version               |
| ------------ | ----------------------------- |
| SUSE         | SLES 15 SP5 or newer          |
| Red Hat      | RHEL 9.6 or newer             |
| Ubuntu       | Ubuntu 25.04 or newer         |

## Migrate a Windows VM from SCSI to NVMe

This section describes how to convert a Windows VM from SCSI to NVMe by using the Azure NVMe Conversion script. The script handles OS preparation, VM deallocation, disk controller update, optional resize, and VM restart automatically.

### Prerequisites

Before you begin, ensure the following:

- The VM uses Generation 2 (Gen2). You can't convert Gen1 VMs to NVMe.

- The VM runs Windows Server 2019 or later.

  Windows Server 2016 and earlier aren't supported unless you use `-IgnoreWindowsVersionCheck` and verify driver compatibility manually.

- The target VM size supports NVMe. To confirm, see the [Azure Boost availability table](/azure/azure-boost/overview#current-availability).

- You can't convert VMs configured with Trusted Launch from SCSI to NVMe.

- Conversion from a VM with a temporary disk (for example, `Standard_D4ds_v5`) to a v6 size (for example, `Standard_D4ds_v6`) isn't supported through this script. Use disk snapshots for that migration path.

  Conversion from VMs without a temporary disk (for example, `Standard_D4s_v5`) to v6 sizes is supported.

- Non-Microsoft antivirus or security software can interfere with the driver changes made during conversion. Temporarily disable it before you run the script.

  If the VM shows a blue screen after conversion, revert to SCSI and retry after disabling your security solution.

- PowerShell with the following Az module versions is installed:

  - `Az.Compute` 9.0 or later
  - `Az.Accounts` 4.0 or later
  - `Az.Resources` 7.0 or later

### Download the script

```powershell
Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" `
  -OutFile ".\Azure-NVMe-Conversion.ps1"
```

The script is part of the open-source [SAP-on-Azure-Scripts-and-Utilities](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/main/Azure-NVMe-Utils) repository and is licensed under the MIT license.

### Run the conversion

Use the `-FixOperatingSystemSettings` switch to have the script automatically configure the `stornvme` driver for boot-start. This is required for Windows to recognize the NVMe controller after the VM restarts. Omitting it equires you to set the driver manually before conversion.

```powershell
.\Azure-NVMe-Conversion.ps1 `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<vm-name>" `
  -NewControllerType NVMe `
  -VMSize "<target-sku>" `
  -FixOperatingSystemSettings `
  -StartVM `
  -WriteLogfile
```

The script performs the following steps automatically:

1. Validates module versions, VM existence, OS type, Windows version, Gen2, current controller type, and NVMe capability with the VM size.
2. Optionally fixes the `stornvme` driver service (`sc.exe config stornvme start=boot`) and validates other OS settings for NVMe readiness (with `-FixOperatingSystemSettings`).
3. Stops and deallocates the VM.
4. Updates `supportedCapabilities.diskControllerTypes` to `SCSI, NVMe` on the OS disk via a REST PATCH.
5. Resizes the VM to the target size.
6. Starts the VM (with `-StartVM`).

> [!TIP]
> The script outputs a revert command at the end of a successful run. Save that command before closing the session so you can roll back to SCSI if needed.

### What changes for your Windows VM

Unlike Linux, Windows uses drive letters rather than device paths, so the OS disk remains `C:\` after conversion. However, the underlying disk interface changes, and data disk assignments may shift if you don't use persistent disk identifiers.

| Disk | SCSI-enabled VM | NVMe-enabled VM |
| ---- | --------------- | --------------- |
| OS disk | `C:\` (unchanged) | `C:\` (unchanged) |
| Temp disk | `D:\` (typically) | `D:\` (typically, RAW on v6 VMs—not pre-formatted with NTFS) |
| Data disks | Assigned by LUN order | Assigned by NVMe namespace order |

> [!IMPORTANT]
> On v6 VMs, temp disks are RAW and not pre-formatted with NTFS. Use a startup script or custom script extension to format and mount them at each boot.

### Verify the conversion

After the VM restarts, confirm the disk controller type changed successfully.

**Using PowerShell:**

```powershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -VMName "<vm-name>"
$vm.StorageProfile.DiskControllerType
```

The output should be `NVMe`.

**Using Device Manager inside the VM:**

1. Open **Device Manager**.
2. Expand **Storage controllers**.
3. Confirm **Standard NVM Express Controller** is listed.

### Revert to SCSI

If you need to roll back, rerun the script with `-NewControllerType SCSI` and the original VM size:

```powershell
.\Azure-NVMe-Conversion.ps1 `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<vm-name>" `
  -NewControllerType SCSI `
  -VMSize "<original-size>" `
  -StartVM `
  -WriteLogfile
```
