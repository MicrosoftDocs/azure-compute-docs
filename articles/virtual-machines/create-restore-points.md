---
title: Create Virtual Machine restore points
description: Creating Virtual Machine Restore Points with API
author: iamwilliew
ms.author: wwilliams
ms.service: azure-virtual-machines
ms.subservice: recovery
ms.date: 02/14/2022
ms.topic: quickstart
ms.custom: template-quickstart
# Customer intent: As a cloud administrator, I want to create virtual machine restore points using APIs, so that I can implement effective backup and retention policies while maintaining application and file system consistency for my VMs.
---

# Quickstart: Create VM restore points using APIs

Use the Azure Compute REST APIs to create application-consistent or crash-consistent restore points for a VM, in the same region or cross-region.

API references: [Restore Points](/rest/api/compute/restore-points) | [Restore Point Collections](/rest/api/compute/restore-point-collections) | [PowerShell](/powershell/module/az.compute/new-azrestorepoint)

---

## Prerequisites

- [Learn more](concepts-restore-points.md) about the requirements for a VM restore point.
- Review the [support requirements](/azure/virtual-machines/concepts-restore-points) and [limitations](/azure/virtual-machines/virtual-machines-create-restore-points#limitations) before creating restore points.


## Create VM restore points

The following sections outline the steps you need to take to create VM restore points with the Azure Compute REST APIs.

You can find more information in the [Restore Points](/rest/api/compute/restore-points), [PowerShell](/powershell/module/az.compute/new-azrestorepoint), and [Restore Point Collections](/rest/api/compute/restore-point-collections) API documentation.

### Step 1: Create a VM restore point collection

A restore point collection is the parent resource that holds all restore points for a VM.

Call the [Restore Point Collections — Create or Update](/rest/api/compute/restore-point-collections/create-or-update) API:

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/restorePointCollections/{collectionName}?api-version=2021-03-01
```

**Request body:**

```json
  {
    "location": "<region>",
    "properties": {
      "source": {
        "id": "<VM Arm Id>"
      },
      "instantAccess": true
    }
  }
```

- Set `location` to the VM's region for a local collection, or to the target region for a cross-region collection (and include the source restore point collection's ARM resource ID in `source.id`).
- Optionally to enable **Instant Access (Preview)**, add `"instantAccess": true` to `properties`. This applies to all restore points created in the collection. Requires API version **2025-04-01** or later. This is applicable only for VMs with Premium SSD v2 and/or Ultra disks as **data** disks.

---

### Step 2: Create a VM restore point

Call the [Restore Points — Create](/rest/api/compute/restore-points/create) API within the collection created in Step 1:

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/restorePointCollections/{collectionName}/restorePoints/{restorePointName}?api-version=2021-03-01
```
**Request body:**

```json
  {
    "name": "<restorePointName>",
    "properties": {
     "instantAccessDurationMinutes": 120,
      "provisioningState": "Succeeded",
    }
  }
```
> **Note:** instantAccessDurationMinutes is an optional parameter. Default is 300 (5 hours). Can be set to a lower value, but not higher than 300. This is applicable for future restore points and NOT for existing restore points already created.

**Key request body properties:**

| Property | Description |
|---|---|
| `consistencyMode` | Omit for application-consistent (default). Set to `CrashConsistent` for crash-consistent restore points. |
| `excludeDisks` | Optional. Array of disk identifiers to exclude from the restore point, to reduce storage cost. |
| `instantAccessDurationMinutes` | Optional. *(Instant Access only)* Duration in minutes for Instant Access. Valid range: 60–300. Default: 300 minutes (5 hours). |

---

### Step 3: Track the status of the VM restore point creation

**Local restore points** complete within a few seconds. Check `provisioningState` on the restore point — it transitions from `Creating` to `Succeeded` (or `Failed`).

**Cross-region restore points** are a long-running operation. Poll the [Restore Points — Get](/rest/api/compute/restore-points/get) API with `$expand=instanceView` to check per-disk copy progress (`completionPercent`). The restore point is usable only after all disk restore points have completed replication.

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/restorePointCollections/{collectionName}/restorePoints/{restorePointName}?$expand=instanceView&api-version=2021-03-01
```
**Snapshot Access status:** If the collection has Instant Access enabled, the same GET response with instanceView includes `snapshotAccessState` per disk restore point. A status of `InstantAccess` or `AvailableWithInstantAccess` means the restore point is ready for fast disk restoration.

### Step 4: Disable InstantAccess 
Use the following REST API call to disable IA enabled on the VM.

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/restorePointCollections/{restorePointCollectionName}?api-version=2025-04-01
```
**Request body:**

```json
  {
    "location": "<region>",
    "properties": {
      "source": {
        "id": "<VM Arm Id>"
      },
      "instantAccess": false
    }
  }
```


## Next steps
- [Learn more](manage-restore-points.md) about managing restore points.
- Create restore points using the [Azure portal](virtual-machines-create-restore-points-portal.md), [CLI](virtual-machines-create-restore-points-cli.md), or [PowerShell](virtual-machines-create-restore-points-powershell.md).
- [Learn more](backup-recovery.md) about Backup and restore options for virtual machines in Azure.