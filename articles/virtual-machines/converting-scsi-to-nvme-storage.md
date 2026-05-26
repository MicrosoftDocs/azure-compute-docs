---
title: Automate SCSI-to-NVMe conversion for Linux and Windows VMs
description: Use a single PowerShell script to automate the SCSI-to-NVMe disk controller conversion for Azure virtual machines running Linux or Windows, including OS preparation.
author: padmalathas
ms.author: padmalathas
ms.reviewer: cynthn, glimoli 
ms.date: 05/26/2026
ms.service: azure-virtual-machines
ms.subservice: disks
ms.topic: how-to
ms.custom: linux-related-content
---

# Automate SCSI-to-NVMe conversion for Linux and Windows VMs

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs 

Azure virtual machines (VMs) support two storage interface types: SCSI (Small Computer System Interface) and NVMe (Non-Volatile Memory Express). Newer Azure VM series - including v5, v6, Ebsv5, and Lsv3 - use NVMe, which delivers lower latency and significantly higher throughput than SCSI. This performance boost is beneficial for I/O-intensive workloads such as SAP, SQL Server, and high-transaction databases.

If you deployed your existing VM with a SCSI controller and you want to resize it to a newer NVMe-capable SKU, you must explicitly convert the disk controller type. This article shows you how to do that conversion for both Linux and Windows VMs by using a single PowerShell script.

> [!IMPORTANT]
> Converting from a VM with a temp disk (for example, `Standard_D4ds_v5`) to an Intel or AMD v6 SKU (for example, `Standard_D4ds_v6`) isn't supported through direct controller conversion. In that case, use disk snapshots to migrate your data. Converting VMs *without* a temp disk (for example, `Standard_D4s_v5`) to v6 SKUs is supported.

## How the conversion works

Changing the host interface from SCSI to NVMe **doesn't** change your remote storage - your OS disk and data disks stay the same. What changes is how the operating system presents those disks.

| Disk type      | SCSI-enabled VM | NVMe-enabled VM   |
|----------------|-----------------|-------------------|
| OS disk        | `/dev/sda`      | `/dev/nvme0n1`    |
| Temp disk      | `/dev/sdb`      | `/dev/sda`        |
| First data disk| `/dev/sdc`      | `/dev/nvme0n2`    |

> [!NOTE]
> When running a v6 VM, you can have up to four temp disks. All of them are RAW. Use cloud-init or a similar mechanism to initialize those disks each time the OS starts.

## Prerequisites

Before you begin, make sure you have the following prerequisites:

- An active Azure subscription and a VM that uses the **SCSI** disk controller configuration.
- PowerShell with the following Az module versions installed:
  - `Az.Compute` version 9.0 or later
  - `Az.Accounts` version 4.0 or later
  - `Az.Resources` version 7.0 or later
- If you're running the script locally, sign in to Azure.
  ```powershell
  Connect-AzAccount
  ```
- The target VM SKU must support NVMe. Check the SKU's capabilities in the Azure portal or use the following command:
  ```powershell
  (Get-AzComputeResourceSku -Location "<region>" | Where-Object {$_.Name -eq "<sku>"}).Capabilities | Where-Object { $_.Name -eq "DiskControllerTypes" }
  ```
  The `Value` field should show `SCSI, NVMe` to confirm NVMe support.

> [!NOTE]
> You can't convert VMs configured with **Trusted Launch** from SCSI to NVMe.

> [!NOTE]
> If you use udev rules to identify data disks by LUN ID with `/dev/disk/azure/scsi1/lunX`, those rules aren't valid after migration to NVMe. To re-establish disk identification, install the `azure-vm-utils` package or manually deploy a udev rule available on GitHub.

## Download the conversion script

Download `Azure-NVMe-Conversion.ps1` directly from GitHub:

```powershell
Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" `
  -OutFile "Azure-NVMe-Conversion.ps1"
```

The script is part of the [SAP-on-Azure-Scripts-and-Utilities](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/main/Azure-NVMe-Utils) open-source repository and is licensed under the MIT license.

## Convert a VM to NVMe

The script handles all the conversion steps automatically:

1. **Pre-flight checks** – validates module versions, Azure context, VM existence, ADE for Linux, VM power state, OS type, Windows version, current controller type, Gen1/Gen2, and SKU availability and NVMe capability.
1. **OS preparation** (optional) – checks and, if you specify `-FixOperatingSystemSettings`, corrects OS-level settings required for NVMe to function after conversion.
1. **VM deallocation** – the VM is stopped and deallocated (required before changing the controller type).
1. **Disk controller update** – a REST PATCH sets `supportedCapabilities.diskControllerTypes` to `SCSI, NVMe` on the OS disk.
1. **SKU resize** (if specified) – the VM is resized to the target NVMe-capable SKU.
1. **VM start** (if `-StartVM` is specified) – the VM is restarted after conversion.

### Basic conversion

```powershell
.\Azure-NVMe-Conversion.ps1 `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<vm-name>" `
  -NewControllerType NVMe `
  -VMSize "<target-sku>" `
  -StartVM `
  -WriteLogfile
```

### Conversion with OS preparation

Use `-FixOperatingSystemSettings` to have the script automatically configure the OS for NVMe. This option is recommended for most scenarios.

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

### Example

```powershell
.\Azure-NVMe-Conversion.ps1 `
  -ResourceGroupName "myResourceGroup" `
  -VMName "myVM" `
  -NewControllerType NVMe `
  -VMSize "Standard_E4bds_v5" `
  -FixOperatingSystemSettings `
  -StartVM `
  -WriteLogfile
```

## Script parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `-ResourceGroupName` | String | Yes | Name of the resource group containing the VM. |
| `-VMName` | String | Yes | Name of the VM to convert. |
| `-NewControllerType` | String | Yes | Target controller type. Valid values: `NVMe`, `SCSI`. |
| `-VMSize` | String | Yes | Target VM SKU (must support NVMe). |
| `-StartVM` | Switch | No | Starts the VM after the conversion completes. |
| `-WriteLogfile` | Switch | No | Writes a log file to the current directory (`Azure-NVMe-Conversion-<VMName>-<timestamp>.log`). |
| `-FixOperatingSystemSettings` | Switch | No | Automatically prepares the OS for NVMe. Recommended for most scenarios. |
| `-IgnoreSKUCheck` | Switch | No | Skips the SKU availability check for the target region/zone. |
| `-IgnoreWindowsVersionCheck` | Switch | No | Skips the Windows OS version compatibility check. |
| `-IgnoreAzureModuleCheck` | Switch | No | Skips the Az module version check. |
| `-IgnoreOSCheck` | Switch | No | Skips the OS support check. |

## Troubleshoot

### HTTP 409 PropertyChangeNotAllowed error

This error can occur when the VM has Ultra Disk or Premium SSD v2 data disks attached. The PATCH payload includes `DiskIOPSReadWrite` and `DiskMBpsReadWrite` properties that conflict with the controller change. Detach those disks before running the conversion, then reattach them afterward.

### Windows VM doesn't boot after conversion

The `stornvme` driver service might not be set to start at boot. Run the following command inside the VM before conversion, or rerun the script with `-FixOperatingSystemSettings`:

```cmd
sc.exe config stornvme start=boot
```

### Linux VM boots into emergency shell

This problem almost always happens because of an `fstab` issue after device name changes. For more information, see the [How the conversion works](#how-the-conversion-works) table. To fix the problem, boot from a repair disk, mount the OS disk, and update `/etc/fstab` to use persistent identifiers (UUID or by-id paths) instead of device names like `/dev/sda`.

## Related content

- [NVMe performance on Azure VMs](/azure/virtual-machines/nvme-overview)
- [Convert Linux VMs from SCSI to NVMe](/azure/virtual-machines/nvme-linux)
- [Enable NVMe on Azure VMs - FAQ](/azure/virtual-machines/enable-nvme-faqs)
- [Azure Boost overview](/azure/azure-boost/overview.md)
- [SAP-on-Azure-Scripts-and-Utilities (GitHub)](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/main/Azure-NVMe-Utils)
