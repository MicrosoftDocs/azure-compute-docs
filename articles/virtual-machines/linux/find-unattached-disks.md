---
title: Azure CLI - Find and delete unattached managed and unmanaged disks
description: How to find and delete unattached Azure managed and unmanaged (VHDs/page blobs) disks by using Azure CLI.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 02/18/2026
ms.author: rogarana
ms.custom:
  - devx-track-azurecli
  - linux-related-content
  - sfi-ropc-nochange
# Customer intent: "As a cloud administrator, I want to find and delete unattached managed and unmanaged disks using scripts, so that I can reduce unnecessary storage costs and manage resources efficiently."
---

# Find and delete unattached Azure managed and unmanaged disks by using the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

[!INCLUDE [disks-unmanaged-disks-retirement](../includes/disks-unmanaged-disks-retirement.md)]

When you delete a virtual machine (VM) in Azure, by default, any disks that are attached to the VM aren't deleted. This feature helps to prevent data loss due to the unintentional deletion of VMs. After a VM is deleted, you will continue to pay for unattached disks. This article shows you how to find and delete any unattached disks and reduce unnecessary costs. 

> [!NOTE]
> Use the [az disk show](/cli/azure/disk) command to get the `LastOwnershipUpdateTime` for any disk. This property shows when the disk's state was last updated. For an unattached disk, this property shows the time when the disk was unattached. This property is blank for newly created disks, until their state changes.


## Managed disks: Find and delete unattached disks 

The following script looks for unattached [managed disks](../managed-disks-overview.md) by examining the value of the **ManagedBy** property. When you attach a managed disk to a VM, the **ManagedBy** property contains the resource ID of the VM. When a managed disk is unattached, the **ManagedBy** property is null. The script examines all the managed disks in an Azure subscription. When the script locates a managed disk with the **ManagedBy** property set to null, the script determines that the disk is unattached.

>[!IMPORTANT]
>First, run the script by setting the **deleteUnattachedDisks** variable to 0. This action lets you find and view all the unattached managed disks.
>
>After you review all the unattached disks, run the script again and set the **deleteUnattachedDisks** variable to 1. This action lets you delete all the unattached managed disks.
>

```azurecli
# Set deleteUnattachedDisks=1 if you want to delete unattached Managed Disks
# Set deleteUnattachedDisks=0 if you want to see the Id of the unattached Managed Disks
deleteUnattachedDisks=0
unattachedDiskIds=$(az disk list --query '[?managedBy==`null`].[id]' -o tsv)
for id in ${unattachedDiskIds[@]}
do
    if (( $deleteUnattachedDisks == 1 ))
    then

        echo "Deleting unattached Managed Disk with Id: "$id
        az disk delete --ids $id --yes
        echo "Deleted unattached Managed Disk with Id: "$id

    else
        echo $id
    fi
done
```

## Unmanaged disks: Find and delete unattached disks 

Unmanaged disks are VHD files that are stored as [page blobs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) in [Azure storage accounts](/azure/storage/common/storage-account-overview). The following script looks for unattached unmanaged disks (page blobs) by checking the value of the **LeaseStatus** property. When you attach an unmanaged disk to a VM, the **LeaseStatus** property is **Locked**. When an unmanaged disk is unattached, the **LeaseStatus** property is **Unlocked**. The script checks all the unmanaged disks in all the Azure storage accounts in an Azure subscription. When the script finds an unmanaged disk with a **LeaseStatus** property set to **Unlocked**, the script determines that the disk is unattached.

>[!IMPORTANT]
>First, run the script by setting the **deleteUnattachedVHDs** variable to 0. This lets you find and view all the unattached unmanaged VHDs.
>
>After you review all the unattached disks, run the script again and set the **deleteUnattachedVHDs** variable to 1. This deletes all your unattached unmanaged VHDs.
>

```azurecli
# Set deleteUnattachedVHDs=1 if you want to delete unattached VHDs
# Set deleteUnattachedVHDs=0 if you want to see the details of the unattached VHDs
deleteUnattachedVHDs=0
storageAccountIds=$(az storage account list --query [].[id] -o tsv)
for id in ${storageAccountIds[@]}
do
    connectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
    containers=$(az storage container list --connection-string $connectionString --query [].[name] -o tsv)

    for container in ${containers[@]}
    do 
        
        blobs=$(az storage blob list --show-next-marker -c $container --connection-string $connectionString --query "[?properties.blobType=='PageBlob' && ends_with(name,'.vhd')].[name]" -o tsv)
        
        for blob in ${blobs[@]}
        do
            leaseStatus=$(az storage blob show -n $blob -c $container --connection-string $connectionString --query "properties.lease.status" -o tsv)
            
            if [ "$leaseStatus" == "unlocked" ]
            then 

                if (( $deleteUnattachedVHDs == 1 ))
                then 

                    echo "Deleting VHD: "$blob" in container: "$container" in storage account: "$id

                    az storage blob delete --delete-snapshots include  -n $blob -c $container --connection-string $connectionString

                    echo "Deleted VHD: "$blob" in container: "$container" in storage account: "$id
                else
                    echo "StorageAccountId: "$id" container: "$container" VHD: "$blob
                fi

            fi
        done
    done
done 
```

## Next steps

For more information, see [Delete a storage account](/azure/storage/common/storage-account-create#delete-a-storage-account).
