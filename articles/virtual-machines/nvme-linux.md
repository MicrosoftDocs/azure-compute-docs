---
author: ParthShiras
ms.author: parthshiras
ms.date: 06/19/2026
ms.topic: how-to
ms.service: azure-virtual-machines
title: Convert SCSI to NVMe for Linux and Windows VMs
description: Learn how to convert SCSI to NVMe by using Linux.
ms.custom: sfi-image-nochange, linux-related-content, windows-related-content
# Customer intent: As a cloud solutions architect, I want to convert virtual machines running Linux from SCSI to NVMe storage, so that I can enhance their performance and scalability while ensuring compatibility with modern cloud infrastructure.
---

# Convert Linux and Windows VMs from SCSI to NVMe

Azure virtual machines (VMs) support two storage controller interfaces: Small Computer System Interface (SCSI) and NVM Express (NVMe). SCSI is a legacy standard that provides physical connectivity and data transfer between computers and peripheral devices. NVMe provides similar connectivity, but is a faster and more efficient interface for data transfer between servers and storage systems.

This article shows you how to convert an Azure VM from a SCSI disk controller to NVMe by using Azure Boost and the Azure NVMe Conversion script. The same script and procedure apply to:

- Linux and Windows VMs.
- VMs with or without data disks and a temporary disk.

Azure continues to support the SCSI interface on the VM offerings that provide SCSI storage. However, not all new VM series include SCSI as an option going forward.

## What's changing for your VM?

Changing the host interface from SCSI to NVMe doesn't change the remote storage (OS disk or data disks). It changes the way the operating system uses the disks.

# [Linux](#tab/linux)

| Disk | SCSI VM | NVMe VM (v5 with temp disk) | NVMe VM (v6) |
|---|---|---|---|
| OS disk | /dev/sda | /dev/nvme0n1 | /dev/nvme0n1 |
| Temp disk | /dev/sdb | /dev/sda | /dev/nvme1n1 |
| First data disk | /dev/sdc | /dev/nvme0n2 | /dev/nvme0n2 |

# [Windows](#tab/windows)

Windows uses drive letters rather than device paths, so the OS disk stays `C:\` after conversion. The underlying disk interface changes. Data disk assignments might shift if you don't use persistent disk identifiers.

| Disk | SCSI VM | NVMe VM |
|---|---|---|
| OS disk | C:\ | C:\ |
| Temp disk | D:\ | RAW on v6 SKUs |
| Data disks | Assigned by LUN order | Assigned by NVMe namespace order |

> [!IMPORTANT]
> On v6 SKUs, temp disks are RAW and aren't preformatted with NTFS. Use a startup script or custom script extension to format and mount them at each startup.
> Some VM sizes (for example, Standard_E64ds_v6) have more than one temporary disk.

---
Converting your Azure VM from SCSI to NVMe by using Azure Boost can help you take full advantage of these performance improvements and maintain a competitive edge in the cloud computing landscape.

## Migrate a Linux VM from SCSI to NVMe

To migrate from SCSI to NVMe, follow these high-level steps:

1. Check that your VM series supports NVMe.
2. Check the prerequisites for your operating system.
3. Convert your VM to NVMe.
4. Verify the conversion.

### 1. Check if your virtual machine series supports NVMe

The [Azure Boost availability table](/azure/azure-boost/overview#current-availability) lists the VM sizes that support NVMe attached disks.

### 2. Check the prerequisites

Before you begin, make sure you have the following prerequisites:

- An Azure subscription and a virtual machine that you need to convert from SCSI to NVMe or from NVMe to SCSI
- PowerShell and the Az module installed and configured
- Appropriate permissions to execute scripts on Azure resources

### 3. Usage

To run the script, open PowerShell as an administrator and follow these steps:

1. Install PowerShell or run the script on Azure Cloud Shell.
> [!TIP]
> Install PowerShell by using the [PowerShell documentation](https://aka.ms/powershell).

2. Allow unsigned PowerShell script files.
```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
```
3. Connect to your Azure account by using `Connect-AzAccount`.
```powershell
Connect-AzAccount
```
4. Make sure that you're connected to your subscription. To change the subscription, use `Set-AzContext`.
```powershell
Select-AzSubscription -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
5. Download the script.
```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" -OutFile ".\Azure-NVMe-Conversion.ps1"
```

6. Run the conversion script.
```powershell
.\Azure-NVMe-Conversion.ps1
```
> [!NOTE]
> If you're converting between VM sizes with different resource (temporary) disk support, your Azure subscription must be registered for the `VMTempDiskResizePreview` feature. If the feature isn't registered, the script exits with an error. Register the feature by using the following command, wait 10 minutes for the auto-approval to take effect, and then rerun the script.
>
> ```powershell
> Register-AzProviderFeature -FeatureName VMTempDiskResizePreview -ProviderNamespace Microsoft.Compute

#### Example
```powershell
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName <your-RG> -VMName <your-VMname> -NewControllerType <NVMe/SCSI> -VMSize <new-VM-SKU> -StartVM
```
The script has the following parameters.

| Parameter | Description | Required |
|---|---|---|
| `-ResourceGroupName` | The resource group name for your VM. | Yes |
| `-VMName` | The name of your VM on Azure. | Yes |
| `-NewControllerType` | The storage controller type to convert the VM to (NVMe or SCSI). | Yes |
| `-VMSize` | The Azure VM SKU to convert the VM to. | Yes |
| `-StartVM` | Start the VM after conversion. | No |
| `-FixOperatingSystemSettings` | Automatically fix the OS settings by using Azure Run Commands. | No |
| `-WriteLogfile` | Create a log file. | No |
| `-IgnoreSKUCheck` | Ignore the check of the VM SKU. | No |
| `-IgnoreWindowsVersionCheck` | Ignore the Windows version check. | No |
| `-IgnoreAzureModuleCheck` | Don't run the check for installed Azure modules. | No |
| `-IgnoreOSCheck` | Don't check for OS readiness. The expectation is that the OS is ready. | No |
| `-SleepSeconds` | The time for Azure to settle changes before starting the VM. | No |

The script performs the following steps:

1. Validates Az module versions, VM existence, OS type, generation, current controller type, and NVMe capability of the target VM SKU.
2. With `-FixOperatingSystemSettings`, validates and configures the guest OS for NVMe by using Azure Run Command:
   - On Linux, checks that the NVMe driver is in `initrd`/`initramfs`, sets `nvme_core.io_timeout=240`, and checks `/etc/fstab` for deprecated device names.
   - On Windows, configures the `stornvme` driver to start at boot (`sc.exe config stornvme start=boot`).
3. Stops and deallocates the VM.
4. Updates `supportedCapabilities.diskControllerTypes` to `SCSI, NVMe` on the OS disk.
5. Resizes the VM to the target SKU and sets the disk controller type to NVMe.
6. Starts the VM (with `-StartVM`).

### 4. Verify the conversion

#### Confirm by using PowerShell
After the VM restarts, confirm that the disk controller type changed successfully.
```powershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -VMName "<vm-name>"
$vm.StorageProfile.DiskControllerType
```


#### Check the result inside the guest

# [Linux](#tab/linux)

Check the devices by using the `nvme` command. If the `nvme` command is missing, install the `nvme-cli` package.

```bash
nvme list
```

The output shows the OS disk and the data disks.


# [Windows](#tab/windows)

1. Open Device Manager.
2. Expand **Storage controllers**.
3. Confirm that **Standard NVM Express Controller** is listed.

---

> [!TIP]
> You can always revert to SCSI. The script provides a command at the end of a successful run to revert to your original configuration. Save that command before you close the session.

If you need to roll back, rerun the script with `-NewControllerType SCSI` and the original VM SKU:

```powershell
.\Azure-NVMe-Conversion.ps1 -ResourceGroupName <your-RG> -VMName <your-VMname> -NewControllerType SCSI -VMSize <original-SKU> -StartVM
```

