---
title: Create a snapshot of an Azure managed disk
description: Learn how to create a point-in-time copy of an Azure managed disk for backup or troubleshooting by using the Azure portal, Azure PowerShell, or Azure CLI.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/17/2026
# Customer intent: "As an IT administrator, I want to create an Azure managed disk snapshot as a point-in-time copy, so that I can use it for backup or to troubleshoot virtual machine issues efficiently."
---

# Create a snapshot of an Azure managed disk

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

An Azure managed disk snapshot is a full, read-only copy of a managed disk. Use a snapshot as a point-in-time backup or to help troubleshoot virtual machine (VM) issues. You can take a snapshot of either an operating system (OS) disk or a data disk. These snapshots exist independently of the source disk, and you can use them to create new managed disks.

## Azure managed disk snapshot billing

Snapshots are billed based on the used size. For example, if you create a snapshot of a managed disk with provisioned capacity of 64 GiB and an actual used data size of 10 GiB, that snapshot is billed only for the used data size of 10 GiB. You can see the used size of your snapshots by checking the [Azure usage report](/azure/cost-management-billing/understand/review-individual-bill). For example, if the used data size of a snapshot is 10 GiB, the *daily* usage report shows 10 GiB/(31 days) = 0.3226 as the consumed quantity. Snapshots are billed separately from their original disk. For details, see the [pricing page](https://azure.microsoft.com/pricing/details/managed-disks).

## Create an Azure managed disk snapshot

If you want to use a snapshot to create a new VM, ensure that you first cleanly shut down the VM. This action clears any processes that are in progress.

# [Azure portal](#tab/portal)

To create a snapshot using the Azure portal, complete these steps.

1. In the [Azure portal](https://portal.azure.com), select **Create a resource**.
1. Search for and select **Snapshot**.
1. In the **Snapshot** window, select **Create**. The **Create snapshot** window appears.
1. For **Resource group**, select an existing [resource group](/azure/azure-resource-manager/management/overview#resource-groups) or enter the name of a new one.
1. Enter a **Name**, and then select a **Region** and **Snapshot type** for the new snapshot. To store your snapshot in zone-redundant storage, select a region that supports [availability zones](/azure/reliability/availability-zones-overview). For a list of supporting regions, see [Azure regions with availability zones](/azure/reliability/availability-zones-region-support).
1. For **Source subscription**, select the subscription that contains the managed disk to be backed up.
1. For **Source disk**, select the managed disk to snapshot.
1. For **Storage type**, select **Standard HDD**, unless you require zone-redundant storage or high-performance storage for your snapshot.
1. If needed, configure settings on the **Encryption**, **Networking**, and **Tags** tabs. Otherwise, default settings are used for your snapshot.
1. Select **Review + create**.

# [Azure PowerShell](#tab/powershell)

This example requires that you use [Cloud Shell](https://shell.azure.com/bash) or install the [Azure PowerShell module](/powershell/azure/install-azure-powershell).

Follow these steps to take a snapshot with the `New-AzSnapshotConfig` and `New-AzSnapshot` cmdlets. This example assumes that you have a VM called *myVM* in the *myResourceGroup* resource group. The code sample provided creates a snapshot in the same resource group and within the same region as your source VM.

First, you'll use the [New-AzSnapshotConfig](/powershell/module/az.compute/new-azsnapshotconfig) cmdlet to create a configurable snapshot object. You can then use the [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) cmdlet to take a snapshot of the disk.

1. Set the required parameters. Update the values to reflect your environment.

   ```azurepowershell-interactive
   $resourceGroupName = 'myResourceGroup' 
   $location = 'eastus' 
   $vmName = 'myVM'
   $snapshotName = 'mySnapshot'  
   ```

1. Use the [Get-AzVM](/powershell/module/az.compute/get-azvm) cmdlet to get the VM with the managed disk you want to snapshot.

   ```azurepowershell-interactive
   $vm = Get-AzVM `
       -ResourceGroupName $resourceGroupName `
       -Name $vmName
   ```

1. Create the snapshot configuration. In the example, the snapshot is of the OS disk. By default, the snapshot uses locally redundant standard storage. We recommend that you store your snapshots in standard storage instead of premium storage whatever the storage type of the parent disk or target disk. Premium snapshots incur additional cost.

   ```azurepowershell-interactive
   $snapshot =  New-AzSnapshotConfig `
       -SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
       -Location $location `
       -CreateOption copy
   ```

    To store your snapshot in zone-redundant storage, create the snapshot in a region that supports [availability zones](/azure/reliability/availability-zones-overview) and include the `-SkuName Standard_ZRS` parameter. For a list of regions that support availability zones, see [Azure regions with availability zones](/azure/reliability/availability-zones-region-support).

1. Take the snapshot.

   ```azurepowershell-interactive
   New-AzSnapshot `
       -Snapshot $snapshot `
       -SnapshotName $snapshotName `
       -ResourceGroupName $resourceGroupName 
   ```

1. Use the [Get-AzSnapshot](/powershell/module/az.compute/get-azsnapshot) cmdlet to verify that your snapshot exists.

    ```azurepowershell-interactive
    Get-AzSnapshot `
        -ResourceGroupName $resourceGroupName
    ```

# [Azure CLI](#tab/cli)

This example requires that you use [Cloud Shell](https://shell.azure.com/bash) or have the [Azure CLI](/cli/azure/) installed.

Follow these steps to take a snapshot with the `az snapshot create` command and the `--source-disk` parameter. This example assumes that you have a VM called *myVM* in the *myResourceGroup* resource group. The code sample provided creates a snapshot in the same resource group and within the same region as your source VM.

1. Get the disk ID with [az vm show](/cli/azure/vm#az-vm-show).

    ```azurecli-interactive
    osDiskId=$(az vm show \
       -g myResourceGroup \
       -n myVM \
       --query "storageProfile.osDisk.managedDisk.id" \
       -o tsv)
    ```

1. Take a snapshot named *osDisk-backup* using [az snapshot create](/cli/azure/snapshot#az-snapshot-create). In the example, the snapshot is of the OS disk. By default, the snapshot uses locally redundant standard storage. We recommend that you store your snapshots in standard storage instead of premium storage whatever the storage type of the parent disk or target disk. Premium snapshots incur additional cost.

    ```azurecli-interactive
    az snapshot create \
        -g myResourceGroup \
    	--source "$osDiskId" \
    	--name osDisk-backup
    ```

    To store your snapshot in zone-redundant storage, create it in a region that supports [availability zones](/azure/reliability/availability-zones-overview) and include the optional `--sku Standard_ZRS` parameter. See the list of [availability zone-enabled regions](/azure/reliability/availability-zones-region-support).
    
1. Use [az snapshot list](/cli/azure/snapshot#az-snapshot-list) to verify that your snapshot exists.
    
    ```azurecli-interactive
    az snapshot list \
       -g myResourceGroup \
       -o table
    ```

---

## Next steps

To recover by using a snapshot, create a managed disk from the snapshot. Then use the managed disk as the OS disk for a new VM or attach it as a data disk to an existing VM.

- [Create a Windows VM from a specialized disk by using the Azure portal or Azure PowerShell](attach-os-disk.md)
- [Create a VM from a snapshot by using Azure CLI](scripts/create-vm-from-snapshot.md)
