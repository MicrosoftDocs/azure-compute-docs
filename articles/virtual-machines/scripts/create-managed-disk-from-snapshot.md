---
title: Create managed disk from snapshot (Linux) - CLI sample
description: Use an Azure CLI script to create a managed disk from a snapshot and monitor the background copy process.
author: roygara
ms.service: azure-disk-storage
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.date: 09/02/2026
ms.author: rogarana
ms.custom: mvc, devx-track-azurecli, linux-related-content
# Customer intent: As a cloud engineer, I want to create managed disks from snapshots by using Azure CLI, so that I can use the disks to restore data or create a virtual machine.
---

# Create a managed disk from a snapshot with CLI (Linux)

This article provides two Azure CLI scripts for creating a managed disk from a snapshot. The first script creates a managed disk with platform-managed keys, and the second script creates a managed disk with customer-managed keys. After you create OS and data disks from their respective snapshots, you can use the disks to create a VM or restore data on an existing VM.

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

## Create managed disks from snapshots

[!INCLUDE [cli-launch-cloud-shell-sign-in.md](~/reusable-content/ce-skilling/azure/includes/cli-launch-cloud-shell-sign-in.md)]

### Disks with platform-managed keys

Use this script to create a managed disk with platform-managed keys. Provide the subscription, resource group, source snapshot, new disk name, disk size, and storage type. For Premium SSD v2 and Ultra Disk, also provide an availability zone. The script sets the subscription, gets the snapshot resource ID, and creates the managed disk from the snapshot in the same location.

:::code language="azurecli" source="~/azure_cli_scripts/virtual-machine/create-managed-disks-from-snapshot/create-managed-disks-from-snapshot.sh" id="FullScript":::

### Disks with customer-managed keys

Use this script to create a managed disk with customer-managed keys. In addition to the subscription, resource group, snapshot, and new disk values, provide the target disk encryption set and its resource group. The script gets the snapshot and disk encryption set information, and then creates the managed disk from the snapshot with the specified disk encryption set.

```azurecli
#Provide the subscription Id of the subscription where you want to create managed disks
subscriptionId="<subscriptionId>"

#Provide the name of your resource group
resourceGroupName=myResourceGroupName

#Provide the name of the snapshot that will be used to create managed disks
snapshotName=mySnapshotName

#Provide the name of the new managed disks that will be create
diskName=myDiskName

#Provide the name of the target disk encryption set
diskEncryptionSetName=myName

#Provide the target disk encryption set resource group
diskEncryptionResourceGroup=myGroup

#Required for Premium SSD v2 and Ultra Disks
#Provide the Availability Zone you'd like the disk to be created in, default is 1
zone=1

#Set the context to the subscription Id where managed disk will be created
az account set --subscription $subscriptionId

#Get the snapshot ID
snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)

#Get the disk encryption set ID
diskEncryptionSetId=$(az disk-encryption-set show --name $diskEncryptionSetName --resource-group $diskEncryptionResourceGroup --query [id] -o tsv)

#Create a new managed disk using the snapshot ID
#The managed disk is created in the same location as the snapshot
#If you're creating a Premium SSD v2 or an Ultra Disk, add "--zone $zone" to the end of the command
az disk create -g $resourceGroupName -n $diskName --source $snapshotId --disk-encryption-set $diskEncryptionSetId
```

## Performance impact - background copy process

When you create a managed disk from a snapshot, it starts a background copy process. You can attach a disk to a VM while this process is running but you'll experience performance impact (4k disks experience read impact, 512e experience both read and write impact) with higher latency, lower IOPS and throughput until background copy completes. For Ultra Disks and Premium SSD v2, use the following command to check the `completionPercent` property. The background copy is complete when the value reaches `100`.

> [!IMPORTANT]
> You can't use the following sections to get the status of the background copy process for disk types other than Ultra Disk or Premium SSD v2. Other disk types will always report 100%.

```azurecli
subscriptionId=yourSubscriptionID
resourceGroupName=yourResourceGroupName
diskName=yourDiskName
az account set --subscription $subscriptionId
az disk show -n $diskName -g $resourceGroupName --query [completionPercent] -o tsv
```

## Clean up resources

Run the following command to delete the resource group and all resources it contains.

```azurecli-interactive
az group delete --name myResourceGroupName
```

## Azure CLI command reference

The scripts use the following Azure CLI commands to create managed disks, monitor the background copy process, and clean up resources.

| Command | Notes |
| --- | --- |
| [az account set](/cli/azure/account#az-account-set) | Sets the subscription where the managed disk is created. |
| [az snapshot show](/cli/azure/snapshot#az-snapshot-show) | Gets the source snapshot resource ID. |
| [az disk-encryption-set show](/cli/azure/disk-encryption-set#az-disk-encryption-set-show) | Gets the target disk encryption set information for a disk with customer-managed keys. |
| [az disk create](/cli/azure/disk#az-disk-create) | Creates a managed disk from the source snapshot. |
| [az disk show](/cli/azure/disk#az-disk-show) | Gets the background copy completion percentage for Premium SSD v2 and Ultra Disk. |
| [az group delete](/cli/azure/group#az-group-delete) | Deletes the resource group and its resources. |

## Next steps

[Create a virtual machine by attaching a managed disk as OS disk](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

For more information on the Azure CLI, see [Azure CLI documentation](/cli/azure).

More virtual machine and managed disks CLI script samples can be found in the [Azure Linux VM documentation](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
