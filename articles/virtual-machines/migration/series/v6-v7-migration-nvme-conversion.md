---
title: Convert a VM from SCSI to NVMe in place
description: Use the Azure NVMe conversion script to switch a VM's disk controller from SCSI to NVMe in place, including prerequisites, parameters, verification, and revert.
author: rod-reis
ms.author: rod-reis
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 07/03/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Convert a VM from SCSI to NVMe in place

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

The [wave-based runbook](v6-migration-migrate.md) covers the full migration sequence. This page focuses on the tool that does the in-place work: a script that switches a VM's disk controller between SCSI and NVMe without a rebuild, so the existing OS disk is kept.

**Use this script only in non-production environments and only when there is a justified business or technical requirement.**

For the platform reference behind the conversion, how the disk controller type changes and how to verify the result, see [Convert SCSI to NVMe for Linux and Windows VMs](/azure/virtual-machines/nvme-linux).

> [!NOTE]
> The conversion script is a community utility published by the SAP on Azure team in the [`Azure/SAP-on-Azure-Scripts-and-Utilities`](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities) repository. It isn't an official Microsoft product and is provided as-is. Review the script and run it in a non-production environment. Pair it with the [runbook sequence](v6-migration-migrate.md) and the [readiness assessment](v6-migration-assess.md) so you run it with a backup and a rollback already in hand.

> [!WARNING]
> This script is for **no-temp-disk source VMs only**, sizes without a `d` before the version suffix (for example, `Standard_D4s_v5` or `Standard_D8s_v5`). VMs that already have a local temporary disk (for example, `Standard_D4ds_v5`) can't convert in place to a v6 size; use the [redeploy-from-image path](v6-migration-migrate.md) instead. See the [temporary-disk dependency row](v6-migration-assess.md#readiness-signals-to-check) in the readiness assessment.

## Before you run the conversion

Confirm the following on the source VM. Each one produces an unsupported result if you skip it:

- **Generation 1 VM.** Generation 1 VMs can't use NVMe. Redeploy from a Generation 2 image instead of converting in place.
- **Azure Disk Encryption (ADE) for Linux.** ADE for Linux isn't supported with NVMe, so converting an ADE-encrypted Linux VM produces an unsupported configuration. **Check for ADE yourself and decrypt the VM first (or use the redeploy-from-image path).** The current script doesn't detect ADE and can complete without warning, leaving the VM in an unsupported state.

## Get the script

Pull the current version straight from the source repository:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure/SAP-on-Azure-Scripts-and-Utilities/refs/heads/main/Azure-NVMe-Utils/Azure-NVMe-Conversion.ps1" -OutFile "Azure-NVMe-Conversion.ps1"
```

> [!CAUTION]
> Older versions of the script can hit an HTTP 409 (`PropertyChangeNotAllowed`) error when Ultra Disk or Premium SSD v2 data disks are attached, because of IOPS/throughput properties in the VM PATCH payload. Always pull the current version as shown (the fix is upstream). This is most relevant for SAP and database workloads, which commonly attach these disk types.

## Set up your session

```powershell
Set-ExecutionPolicy Unrestricted -Scope Process
Connect-AzAccount
Select-AzSubscription -SubscriptionId "<your-subscription-id>"
```

Use a process-scoped execution policy so you don't loosen the policy for the whole machine.

## Run the conversion

A typical NVMe conversion, starting the VM afterward and writing a log:

```powershell
.\Azure-NVMe-Conversion.ps1 `
  -ResourceGroupName "rg-prod-app" `
  -VMName "vm-app-01" `
  -NewControllerType "NVMe" `
  -VMSize "Standard_D8s_v6" `
  -StartVM `
  -FixOperatingSystemSettings `
  -WriteLogfile
```

### Parameters that matter

| Parameter | What it does |
| --- | --- |
| `-NewControllerType` | Set to `NVMe` for the move to v6. Set to `SCSI` to revert a VM if validation fails. |
| `-VMSize` | The target size (for example, a `_v6` size). The script resizes as part of the conversion. In the example, `Standard_D8s_v6` is a non-`d` (no-temp-disk) size, chosen because the script doesn't support temp-disk source VMs. |
| `-FixOperatingSystemSettings` | Applies the OS-side changes NVMe needs so the VM boots cleanly on the new interface. |
| `-StartVM` | Starts the VM once the conversion completes, so you can move straight to validation. |
| `-IgnoreSKUCheck` | Skips the SKU compatibility guardrail. Use only when you know the target is valid and the check is too strict. |
| `-WriteLogfile` | Writes a log of the run. Keep it, it's your record of what changed and your first stop if something looks off. |

## Verify the conversion

Before you hand the VM back, confirm it came up on NVMe. [Convert SCSI to NVMe for Linux and Windows VMs](/azure/virtual-machines/nvme-linux) walks through the full set of checks. The essentials:

- Confirm the target series presents NVMe disks. The [Azure Boost availability table](/azure/azure-boost/overview#current-availability) lists which sizes do.
- Check that the OS now sees the disks over NVMe. On Linux, list namespaces with `lsblk` or `nvme list` and confirm the OS disk is `/dev/nvme0n1`. On Windows, confirm the disks are healthy in Disk Management under the in-box NVMe controller.
- Make sure mounts use stable identifiers, `/dev/disk/azure/*` symlinks or UUIDs in `/etc/fstab`, because NVMe device numbers aren't stable across reboots.

If a check fails, revert with the script (see [Revert the conversion](#revert-the-conversion)) and retry after fixing the OS image.

## Revert the conversion

If the converted VM doesn't validate, run the script again with `-NewControllerType "SCSI"` and the original size to switch the controller back. Because the disks are untouched, the revert is a controller and size change, not a restore. Even so, keep the backup from the checklist until validation passes.

> [!TIP]
> Treat the conversion as routine, not risky. It makes an in-place change you can reverse. The discipline that makes it safe, backup, pilot, and a validation plan, lives in the [readiness assessment](v6-migration-assess.md), not in the command itself.

## Next steps

- [Validate and optimize](v6-migration-validate.md)
- [Convert SCSI to NVMe for Linux and Windows VMs](/azure/virtual-machines/nvme-linux)
