---
author: MelissaHollingshed
ms.author: mehollin
ms.date: 07/31/2024
ms.topic: how-to
ms.service: azure-virtual-machines
title: SCSI to NVMe for Linux VMs
description: How to convert SCSI to NVMe using Linux
# Customer intent: As a cloud solutions architect, I want to convert virtual machines running Linux from SCSI to NVMe storage, so that I can enhance their performance and scalability while ensuring compatibility with modern cloud infrastructure.
---

# Converting Virtual Machines Running Linux from SCSI to NVMe

In this article, we discuss the process of converting virtual machines (VM) running Linux from SCSI to NVMe storage. By migrating to NVMe, you can take advantage of its improved performance and scalability.

## SCSI vs NVMe

Azure VMs support two types of storage interfaces: Small Computer System Interface (SCSI) and NVMe. The SCSI interface is a legacy standard that provides physical connectivity and data transfer between computers and peripheral devices. NVMe is similar to SCSI in that it provides connectivity and data transfer, but NVMe is a faster and more efficient interface for data transfer between servers and storage systems.

### Support for SCSI interface VMs

Azure continues to support the SCSI interface on the versions of VM offerings that provide SCSI storage. However, not all new VM series have SCSI storage as an option going forward.

## What is changing for your VM?
Changing the host interface from SCSI to NVMe doesn't change the remote storage (OS disk or data disks), but change the way the operating systems sees the disks.

| Disk               | SCSI enabled VM   | NVMe VM with SCSI tempdisk (e.g. Ebds_v5) | NVMe VM with NVMe tempdisk  |
|--------------------|-------------------|-------------------------------------------|------------------------------
| **OS disk**        | /dev/sda          | /dev/nvme0n1                              | /dev/nvme0n1                |
| **Temp Disk**      | /dev/sdb          | /dev/sda                                  | /dev/nvme1n1                |
| **First Data Disk**| /dev/sdc          | /dev/nvme0n2                              | /dev/nvme0n2                |

> [!TIP] 
> Some VM types have more than one temporary disk (e.g. E64ds_v6)


In the following sections, we provide a guide to convert your Azure VM from SCSI to NVMe using Azure Boost ensuring you can take full advantage of these performance improvements and maintain a competitive edge in the cloud computing landscape.

## Migrate your virtual machine (VM) from SCSI to NVMe
In order to migrate from SCSI to NVMe, some steps need to be followed:

1. Check if your virtual machine series supports NVMe
2. Check your operating system for NVMe readiness
3. Convert your virtual machine to NVMe
4. Check your operating system 

### 1. Check if your virtual machine series supports NVMe
The supported virtual machines to support NVMe attached disks is described on the [Azure Boost overview site in the availability table](/azure/azure-boost/overview#current-availability).

### 2. Check your operating system for NVMe readiness

The operating system needs to support NVMe devices, includes for example, device drivers and initrdm, the temporary file system used during boot, need to be prepared. In addition to that you need to validate the mount points of the file systems as they check if you use the SCSI device name (/dev/sdX).

The migration script automatically can take care of these readiness checks for you when using the `-FixOperatingSystemSettings`.

#### 2.1 Check Controller Type of VM
 
##### 2.1.1 Check Controller Type using PowerShell

```powershell
PS C:\Users\user1> $vm = Get-AzVM -name [your-vm-name]
PS C:\Users\user1> $vm.StorageProfile.DiskControllerType
SCSI
PS C:\Users\user1>
```

##### 2.1.2 Check Controller Type using Azure CLI

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

##### 2.1.3 Check Controller Type using Azure portal

:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-2.png" alt-text="Screenshot of Azure portal to check controller.":::

#### 2.2 Prepare for migration

The Migration script can automatically take care about the prerequisites when using the `-FixOperatingSystemSettings` parameter.

If you want to manually take care of the required changes validate

- NVMe modules installed and part of initrd/initramfs
- GRUB configuration includes the parameter nvme_core.io_timeout=240
- /etc/fstab checks for devices

Please check back with your OS vendor to cover all required commands to update initrd/initramfs.

##### 2.2.1 Prepare PowerShell

> [!TIP] 
> This step is not required when running the script in Azure CloudShell

1. Install PowerShell using [https://aka.ms/powershell](https://aka.ms/powershell)
1. Connect to Azure using `Connect-AzAccount` and select the correct subscription using `Select-AzSubscription -Subscription [your-subscription-id]`

1. Set the Execution Policy using `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`

##### 2.2.2 Download the script

You can download the script using a PowerShell command

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" -OutFile ".\Azure-NVMe-Conversion.ps1"
```

#### 2.3 Run the migration

The script has multiple parameters available:

| Parameter                      | Description                                                                  | Required |
|--------------------------------|------------------------------------------------------------------------------|----------|
| `-ResourceGroupName`           | The Resource Group Name of your VM                                           | Yes      |
| `-VMName`                      | The name of your Virtual Machine on Azure                                    | Yes      |
| `-NewControllerType`           | The storage controller type the VM should get converted to (NVMe or SCSI)    | Yes      |
| `-VMSize`                      | Azure VM SKU you want to convert the VM to                                   | Yes      |
| `-StartVM`                     | Start the VM after conversion                                                | No       |
| `-IgnoreSKUCheck`              | Ignore the check of the VM SKU                                               | No       |
| `-IgnoreWindowsVersionCheck`   | Ignore the Windows Version check                                             | No       |
| `-FixOperatingSystemSettings`  | Automatically fix the OS settings using Azure RunCommands                    | No       |
| `-WriteLogfile`                | Create a Log File                                                            | No       |
| `-IgnoreAzureModuleCheck`      | Do not run the check for installed Azure modules                             | No       |
| `-IgnoreOSCheck`               | Do not check for OS readiness, expectation is that the OS is ready           | No       |
| `-SleepSeconds`                | Time for Azure to settle changes before starting the VM                      | No       |

Sample Command:

```powershell
# Example usage
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName <your-RG> -VMName <your-VMname> -NewControllerType <NVMe/SCSI> -VMSize <new-VM-SKU> -StartVM -FixOperatingSystemSettings
```

> [!TIP] 
> You can always revert back to SCSI, the script will share a command with you to directly revert to your original configuration.

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
00:00 - INFO      - Connected to Azure subscription ID: 232b6759-xxxx-yyyy-zzzz-757472230e6c
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
01:19 - INFO      - https://management.azure.com/subscriptions/232b6759-xxxx-yyyy-zzzz-757472230e6c/resourceGroups/testrg/providers/Microsoft.Compute/disks/testvm_OsDisk_1_165411276cbe459097929b981eb9b3e2?api-version=2023-04-02
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

If you have challenges accessing the operating system afterwards try to check

- Serial Console for Linux operating systems

- Screenshot from the operating system in Azure portal

When something happens you can always revert back to SCSI using the command shown at the end of the script:


```powershell
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName testrg -VMName testvm -NewControllerType SCSI -VMSize Standard_E4s_v3 -StartVM
```

#### 2.4 Check the result

##### 2.4.1 Check result in Azure portal

:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-3.png" alt-text="Screenshot of Azure portal.":::

##### 2.4.2 Check result in PowerShell

```Powershell
PS C:\Users> $vm = Get-AzVM -name [your-vm-name]
PS C:\Users> $vm.StorageProfile.DiskControllerType
NVMe
PS C:\Users>
```
### 3. Check your operating system
 
#### 3.1 Check devices
You can check the devices using nvme command, if the nvme command is missing, install the "nvme-cli" package.

`nvme list`

The output should show the OS disk and the data disks.
:::image type="content" source="./media/enable-nvme/nvme-vs-scsi-4.png" alt-text="Screenshot of OS disks and data disks.":::


#### 3.2 Get udev file for NVMe (Optional)
On SCSI virtual machines, the udev rules integrated in waagent (Azure agent) created links in `/dev/disk/azure/scsi1/lunX` to identify the data disks. As SCSI isn't used anymore, the rules don't apply.

With one of the two available options to deploy NVMe enabled udev rules you see new symbolic links in the directory `/dev/disk/azure/data/by-lun`. This directory is the replacement for `/dev/disk/azure/scsi1`.

```bash
nvme-conversion-vm:/usr/lib/udev/rules.d # ls -l /dev/disk/azure/data/by-lun/
total 0
lrwxrwxrwx 1 root root 19 Jun 7 13:52 0 -> ../../../../nvme0n2
lrwxrwxrwx 1 root root 19 Jun 7 13:52 1 -> ../../../../nvme0n3
nvme-conversion-vm:/usr/lib/udev/rules.d #
``` 

##### 3.2.1 Manual download of udev file
To download the new udev rules file, use this command:
`curl https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/NVMe-Preflight-Check/88-azure-nvme-data-disk.rules`
and then run `udevadm control --reload-rules && udevadm trigger`
to reload the udev rules.

##### 3.2.2 Ready to install packages using [Azure VM utils](https://github.com/azure/azure-vm-utils )
There are precompiled packages available on [Index of /results/cjp256/azure-vm-utils/](https://download.copr.fedorainfracloud.org/results/cjp256/azure-vm-utils/)for multiple distributions. 

Multiple distributions started already to integrate the package. You can directly install it from their repository.

| Distribution | Minimum version               |
| ------------ | ----------------------------- |
| SUSE         | SLES 15 SP5 or newer          |
| RedHat       | RHEL 9.6 or newer             |
| Ubuntu       | Ubuntu 25.04 or newer         |
