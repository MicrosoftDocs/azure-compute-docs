---
title: Instantly access managed disk snapshots
description: Learn how instant access works for managed disk snapshots of varying disk types.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 01/12/2026
ms.author: rogarana
ms.custom: references_regions, portal
# Customer intent: As a cloud administrator, I want to create incremental snapshots for managed disks, so that I can efficiently back up and restore disk data while minimizing storage costs and improving performance.
---

# Instant access snapshots for Azure managed disks 

Azure Managed Disk snapshots provide point-in-time backups of disks that can be used as backup during software upgrades, disaster recovery, or to create new environments. When creating snapshots from Azure Managed Disks, Azure automatically copies the data from the disk to the snapshot in the background.

Snapshots of Premium SSD, Standard SSD, and Standard HDD disks are by default instant access. Immediately upon creation, these snapshots can be used to restore new disks, download underlying data, and copy to other Azure regions. 

Snapshots of Ultra Disks and Premium SSD v2 aren't instant access by default and require the background data copy to complete before they can be used. To create instant access snapshots, you can opt in to enable instant access using the existing snapshot API at the time of snapshot creation.


## Snapshots of Premium SSD, Standard SSD, and Standard HDD disks

Snapshots of Premium SSD, Standard SSD, and Standard HDD disks are instant access by default. As soon as snapshots are created, you can immediately use these snapshots to create new disks of any supported disk types, generate SAS URIs to download data, or copy snapshots to other Azure regions for regional disaster recovery. After snapshot creation, Azure automatically initiates a background data copy from the source disk to snapshot.

Disks created from snapshots of Premium SSD, Standard SSD, and Standard HDD can be immediately attached to running virtual machines. During disk creation, Azure automatically initiates a background data copy to hydrate the disk from snapshot data. During this process, the disk may experience temporary performance degradation until the background copy completes. To reduce the performance impact, you can create a full snapshot on Premium Storage and restore the disk from that snapshot.

## Snapshots of Ultra Disk and Premium SSD v2

Snapshots of Ultra Disk and Premium SSD v2 aren't instant access by default and can only be used after snapshot’s background data copy completes. To bypass this, you can choose to configure instant access by specifying a value in the `InstantAccessDurationMins` parameter at the time of snapshot creation. This will allow you to create an instant access snapshot and can be immediately used to create new disks. Disks created from instant access snapshots are rapidly hydrated with minimal performance impact and can be immediately attached to a running VM.

After the time specified in `InstantAccessDurationMins` elapses, the snapshot automatically transitions to a Standard Storage snapshot and can be used once the background data copy has completed. You can monitor the snapshot's state via the `SnapshotAccessState` property. 

Until the data is fully copied to Standard Storage, snapshots in the InstantAccess state depend on the availability of the source disk and do not provide protection against disk or zonal failures. To ensure durability and protection, the snapshot must complete its background data copy.

Until the data is fully copied to Standard Storage, snapshots in the **InstantAccess** state depend on the availability of the source disk and  don't provide protection against disk or zonal failures. To ensure zonal redundancy, the snapshot must complete its background data copy to Standard ZRS. You can monitor a snapshot's background data copy progress by checking [`SnapshotAccessState` property](disks-incremental-snapshots.md#check-snapshot-status).

### Limitations

- Only Ultra Disks and Premium SSD v2 can be created from instant access snapshots of Ultra Disks and Premium SSD v2 disks
- `InstantAccessDurationMins` must be between 60 and 300 minutes
- Instant access snapshots count towards the Ultra Disk and Premium SSD v2 limit of three in-progress snapshots per disk
- You can create up to 15 disks concurrently from all instant access snapshots of an individual disk
- You can't use an instant access snapshot to create an Ultra Disk or a Premium SSD v2 larger than the snapshot's size
- If a Ultra Disk or Premium SSD v2 is being hydrated from a snapshot, you can't create an instant access snapshot of that disk
    - Check the `CompletionPercent` property of the disk, if it's below 100 then it's currently being hydrated
- Instant access snapshots of Ultra Disk or Premium SSD v2 can't be copied across regions and the underlying data can't be downloaded until the background data copy completes
    - Check the `CompletionPercent` property on the snapshot, when it reaches 100 then it can be copied across regions and the underlying data can be downloaded
- The encryption property of a disk created from an instant access snapshot can't be updated during disk hydration and you can't update the encryption settings for Ultra Disks and Premium SSD v2 that currently have active instant access snapshots
- Attaching Ultra Disks and Premium SSD v2 across fault domains (using either a VM in an availability set or a Virtual Machine Scale Set) triggers the background data copy and prevents you from creating an instant access snapshot during the background data copy. 
- Ultra Disk and Premium SSD v2 that have active instant access snapshots can't be attached across fault domains
- To create instant access snapshots from Ultra disks, the snapshot must be created from a newly provisioned Ultra Disks.
- The greatest read latency improvements for disks created from instant access snapshots are currently available in Germany West Central, East Asia, Southeast Asia, Central India, and Sweden Central.

### Regional availability

Instant access snapshots are currently supported in all pulic regions.


### Billing of instant access snapshots of Ultra Disks and Premium SSD v2 disks

Instant access snapshots use a usage-based billing model with two components: a storage charge and a one-time restore fee.
- Storage charge: You are billed only for the additional storage consumed by an instant access snapshot during its active lifetime. When a snapshot is first created, it starts at zero additional cost, as it references the source disk as its base. As data on the source disk changes or is deleted over time, the snapshot preserves the original point-in-time state, and its used size grows accordingly. This means you pay only for incremental changes of instant access snapshot, not for a full copy of the disk.
- Restore charge: Each time you restore a disk from an instant access snapshot, a one-time restore operation fee is applied. This fee is calculated based on the provisioned size of the disk at the time of restore, providing predictable billing for restore operations.

Learn more about Instant Access Snapshot billing in [here](https://azure.microsoft.com/pricing/details/managed-disks/).

### Create an instant access snapshot

Instant access snapshots of Ultra DIsk and Premium SSD v2 are not a separate snapshot resource class to manage. They are incremental snapshots that temporarily enter Instant Access state for the specified duration. When this duration expires, snapshots automatically transition out of the Instant Access state and continue as Standard ZRS snapshots for better reliability and long-term retention.

To create instant access snapshots of Ultra Disk and Premium SSD v2, you use the same Snapshot API and commands, but add an additional parameter that specifies the how long the snapshot remains in the instant access state (minimum 60 minutes, maximum 300 minutes). After the specified duration expires, the snapshot automatically transitions to a Standard Storage snapshot. You can monitor your snapshot's state by [checking the snapshot's access state](#check-snapshot-access-state).

In the Azure portal, you can't specify a duration, so snapshots created through the portal remain instant access snapshots for 300 minutes (5 hours).

#### [Azure CLI](#tab/azure-cli)

```azurecli
# Declare variables

subscriptionId="yourSubscriptionId"
diskName="yourDiskName"
resourceGroupName="yourResourceGroupName"
snapshotName="desiredInstantAaccessSnapshotName"

# Set Subscription Id

az account set --subscription $subscriptionId

# Get the disk you need to create an instant access snapshot 

yourDiskID=$(az disk show -n $diskName -g $resourceGroupName --query "id" --output tsv)

# Create an instant access snapshot

az snapshot create -g $resourceGroupName -n $snapshotName --source $yourDiskID --incremental true --location eastus2euap  --sku Standard_ZRS --ia-duration 300 
```

#### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Declare variables
$subscriptionId="yourSubscriptionId"
$diskName = "yourDiskName"
$resourceGroupName = "yourResourceGroupName"
$snapshotName = " desiredInstantAaccessSnapshotName"
$location = "yourLocation"

# Set Subscription Id

Set-AzContext -SubscriptionId $subscriptionId

# Get the disk you need to create an instant access snapshot

$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create instant access snapshot
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental -InstantAccessDurationMinutes 300

New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig
```

#### [Portal](#tab/azure-portal)


1. Sign in to the [Azure portal](https://portal.azure.com) and navigate to the disk you'd like to snapshot.
1. On your disk, select **Create a Snapshot**
1. Select the resource group you'd like to use and enter a name for your snapshot.
1. Select **Enable instant access** and select **Review + Create**

:::image type="content" source="media/disks-instant-access-snapshots/disks-enable-instant-access.png" alt-text="Screenshot displaying enable instant access checked in the snapshot creation process.":::

#### [Resource Manager Template](#tab/azure-resource-manager)

```json
{    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "snapshotName": {
            "type": "String"        
                        },
        "location": {
            "defaultValue": "eastus2euap",
            "type": "String",
            "metadata": {
                "description": "Location for all resources."
                        }
                    },
        "sourceUri": {
            "defaultValue": " <your_managed_disk_resource_ID>",
            "type": "String"        
                     }
                     },
        "resources": [{
            "type": "Microsoft.Compute/snapshots",
            "apiVersion": "2025-01-02",
            "name": "[parameters('snapshotName')]",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "creationData": {
                    "createOption": "Copy",
                    "sourceResourceId": "[parameters('sourceUri')]",
                    "instantAccessDurationMinutes": 300
                                 },
                "incremental": "true"
                          }
                    }]
}
```

---

## Check snapshot access state

You can monitor Managed Disk Snapshot across different states using the `SnapshotAccessState` property of the snapshot resource. This property indicates the current access state of a snapshot, helping you understand its readiness for operations. Here are the states and what they imply:

| **State**                  | **Description**                                                                                                                                                                                                                     | **Applies To**                                                                                                                                                                                                 |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Pending**                | This snapshot can't be used to create a new disk, download data, or copy to another region.                                                                                                 | Incremental snapshots from Ultra Disks and Premium SSD v2 disks during background data copy; snapshots being copied across regions.                                                                            |
| **Available**              | This snapshot can be used to create a new disk (with a performance impact), download data, or copy to another region.                                                                       | Snapshots of Premium SSD, Standard SSD, and Standard HDD disks; incremental snapshots for Ultra Disks and Premium SSD v2 disks after background copy completes; snapshots copied within the same region using shallow copy. |
| **InstantAccess**          | This snapshot can be used to create rapidly hydrated disk with minimal performance impact, but the underlying data can't be downloaded and it can't be copied to another region.                        | Instant access snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy is ongoing.                                                     |
| **AvailableWithInstantAccess** | This snapshot can be used to create rapidly hydrated disk with minimal performance impact, the underlying data can be downloaded, and it can be copied to another region.                              | Instant access snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy completes.                                                      |

### [CLI](#tab/azure-cli-snapshot-state)

```azurecli
snapshotName="DesiredInstantAccessSnapshotTestName"
resourceGroupName="yourResourceGroupName"

snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)

az resource show --ids $snapshotId --query "properties.snapshotAccessState" --output tsv

az resource show --ids $snapshotId --query "properties.creationData.instantAccessDurationMinutes" --output tsv
```

### [Portal](#tab/azure-portal-snapshot-state)

Access your snapshot in the [Azure portal](https://portal.azure.com), the access state is displayed under **Essentials** on **Overview**.

:::image type="content" source="media/disks-instant-access-snapshots/disks-snapshot-instant-access-state.png" alt-text="Screenshot displaying the acesss state of a snapshot in the Azure portal." lightbox="media/disks-instant-access-snapshots/disks-snapshot-instant-access-state.png":::
