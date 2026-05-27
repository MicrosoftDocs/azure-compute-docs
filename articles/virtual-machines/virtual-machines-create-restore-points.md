---
title: Use virtual machine restore points
description: Learn how to use virtual machine restore points.
author: aarthiv
ms.author: aarthiv
ms.service: azure-virtual-machines
ms.subservice: recovery
ms.topic: concept-article
ms.date: 11/06/2023
ms.custom: conceptual
# Customer intent: As a cloud administrator, I want to implement virtual machine restore points, so that I can ensure data protection and facilitate quick recovery in the event of a failure or data loss.
---

# Overview of Virtual Machine restore points

Virtual Machine restore points capture the configuration and point-in-time disk snapshots of a virtual machine, enabling granular backup and recovery. You can create restore points at regular intervals to reduce data loss exposure and meet recovery time objectives (RTOs).

For a broader comparison of backup options, see [Backup and restore options for Virtual Machines in Azure](/azure/virtual-machines/backup-recovery).

## About Virtual Machine restore points

A Virtual Machine restore point stores:

- The Virtual Machine configuration at the time of capture.
- Point-in-time snapshots of all attached managed disks—one **disk restore point** per disk.

Virtual Machine restore points are grouped into **restore point collections**, an Azure Resource Manager resource scoped to a specific Virtual Machine. For ARM template examples, see the [Virtual-Machine-Restore-Points](https://github.com/Azure/Virtual-Machine-Restore-Points) repository.

Restore points are **incremental**: the first restore point is a full copy; each subsequent one captures only the changes since the previous snapshot. You can exclude individual disks to reduce storage costs.

Each disk of the Virtual Machine has a corresponding disk restore points within the restore points. For example, if a Virtual Machine has three disks (One OS disk and two data disks) and each restore point has three disk restore points in it. 

Once the disk restore points are created, Azure automatically initiates a background data copy from source disk to snapshot.

### Consistency modes

| Mode | How it works | Set via |
|---|---|---|
| **Application-consistent** | Uses Volume Shadow Copy Service (VSS) writers (Windows) or pre/post scripts (Linux) to flush in-flight application data before the snapshot. | Default when `consistencyMode` is omitted. |
| **Crash-consistent** | Captures a write-order-consistent snapshot of all disks, equivalent to Virtual Machine state after a power outage or crash. | Set `consistencyMode` to `crashConsistent` in the creation request. |

> **Note:** Crash consistency cannot be guaranteed for:
> - Disks with read/write host caching enabled (writes during snapshot may not be acknowledged by Azure Storage).
> - Intel V6+ (Dsv6, Edsv6, Esv6 series, etc.) and AMD V7+ (Dasv7, Dadsv7, Easv7, Faldsv7 series, etc.) Virtual Machines with more than one data disk. These SKUs use Azure Boost, which offloads storage to hardware and cannot guarantee simultaneous snapshots across disks.
>
> For these configurations, use application-consistent mode.

### Instant Access (Preview)

Virtual Machine restore points support [**Instant Access**](/azure/virtual-machines/disks-instant-access-snapshots?tabs=azure-cli%2Cazure-cli-snapshot-state) for application-consistent restore points on Virtual Machine that have Premium SSD v2 or Ultra disks as data disks. When Instant Access is enabled on the restore point collection, disk restoration can begin immediately from the snapshot—without waiting for full hydration—significantly reducing RTOs.

### Key concepts in Instant Access (Preview)

| Property               | Description |
|------------------------|-------------|
| Allowlist subscription | Open the Cloud shell (PowerShell) from portal.<br> [https://shell.azure.com/](https://shell.azure.com/) <br> - Ensure your using the subscription, which is used for testing this feature. <br> - Run `Register-AzProviderFeature -FeatureName 'AppConsistentInstantAccessSnapshotForDirectDriveDisks' -ProviderNamespace 'Microsoft.Compute'`
| `instantAccess` | Boolean property set on the restore point **collection**. Set to `true` to enable Instant Access for all restore points in the collection. Default is `false`. |
| `instantAccessDurationMinutes` | Integer property set on each **restore point**. Specifies how long Instant Access remains active, in minutes. Valid range: 60–300 minutes. Default: 300 (5 hours). |
| `snapshotAccessState` | Read-only property on an individual disk restore point. Indicates the Instant Access state of that specific disk. |
| API Version | **2025-04-01 or later** |
| Supported regions | West Central US, West US, North Central US, West US 2, South Central US |
| Pricing | Instant access snapshots are billed using a usage‑based model with two types of charges: <br> 1. Snapshot storage charge <br> 2. One‑time restore operation charge. <br> **Snapshot storage charge**: You are billed only for the additional storage used by an instant access snapshot while it is active. When an instant access snapshot is first created, it does not incur any storage cost. The snapshot initially shares data with the source disk. As data on the source disk is modified or deleted over time, the snapshot preserves the original point‑in‑time data, and its storage usage grows. As a result, you pay only for the changed data, not for a full copy of the disk. If no data is modified on the source disk, the snapshot continues to incur no additional storage charges. <br> **Restore operation charge** Each time you restore a disk from an instant access snapshot, a one‑time restore fee is charged. This fee is calculated based on the provisioned size of the disk at the time of restore, providing predictable and transparent pricing for restore operations. Learn more about Instant Access Snapshot billing in [here](https://azure.microsoft.com/pricing/details/managed-disks/)|

## Restore points for Virtual Machine in scale sets and availability sets

Restore points are created per individual Virtual Machine. To back up all instances in a virtual machine scale set (Flexible orchestration mode) or an availability set, create restore points for each Virtual Machine separately.

> **Note:** Virtual machine scale sets with **Uniform orchestration** are not supported.

## Throttling limits for Restore points

**Scope** | **Operation** | **Limit per hour**
--- | --- | ---
Virtual Machine | RestorePoints.RestorePointOperation.PUT (Create new **Application Consistent**) | 3
Virtual Machine | RestorePoints.RestorePointOperation.PUT (Create new **Crash Consisten**t) | 3
Target restore point collection | RestorePoints.RestorePointOperation.PUT (Copy any Virtual Machine Restore Point) | 3


> [!NOTE]
> Requests that exceed limits return HTTP 429. Retry after the interval specified in the response.

## Limitations

**General:**

- Restore points are supported only for managed disks.
- Ultra Disks, Premium SSD v2 disks, Write-accelerated disks, Ephemeral OS disks, and shared disks aren't supported for crash consistency mode.
- Ephemeral OS disks, and shared disks aren't supported for application consistency mode.
- A maximum of 500 Virtual Machine restore points can be retained at any time for a Virtual Machine, irrespective of the number of restore point collections or consistency type.
- Concurrent creation of restore points for the same Virtual Machine isn't supported.
- Restore points for virtual machine scale sets in Uniform orchestration mode aren't supported.
- Movement of Virtual Machines between resource groups or subscriptions isn't supported when the Virtual Machine has restore points. Moving the Virtual Machine between resource groups or subscriptions doesn't update the source Virtual Machine reference in the restore point and causes a mismatch of Resource Manager IDs between the actual Virtual Machine and the restore points.

**Cross-region copy (Preview):**

- Private links aren't supported when copying restore points across regions.
- CMK-encrypted restore points are copied as PMK-encrypted in the target region.

**Instant Access (Preview):**

- Supported only for application-consistent restore points on Virtual Machines with Premium SSD v2 or Ultra **data** disks.
- Not supported for crash-consistent restore points.
- Maximum 50 concurrent Instant Access restore point creations per subscription per region.
- Currently supported via REST API, Azure SDK, CLI, and ARM templates.

For a complete list of limitations, disk type support, OS support, and API version
requirements, see [Support matrix for VM restore points](/azure/virtual-machines/concepts-restore-points).


## Troubleshoot Virtual Machine restore points

Most common restore point failures are attributed to the communication with the VM agent and extension. To resolve failures, follow the steps in [Troubleshoot restore point failures](restore-point-troubleshooting.md).

## Next steps

- [Create a VM restore point](create-restore-points.md).
- [Learn more](backup-recovery.md) about backup and restore options for Virtual Machines in Azure.
- [Learn more](virtual-machines-restore-points-vm-snapshot-extension.md) about the extensions used with application consistency mode.
- [Learn more](virtual-machines-restore-points-copy.md) about how to copy Virtual Machine restore points across regions.
