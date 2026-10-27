---
title: Change the performance of Azure managed disks
description: Learn how to change performance tiers for existing managed disks using the Azure PowerShell module, the Azure CLI, or the Azure portal.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/10/2026
ms.author: rogarana
ms.custom: devx-track-azurecli, devx-track-azurepowershell, portal
# Customer intent: "As a cloud administrator, I want to change the performance tier of managed disks without downtime, so that I can optimize resources for my virtual machines efficiently without interruption to their operations."
---

# Change your performance tier without downtime

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

> [!NOTE]
> This article focuses on how to change performance tiers. To learn how to change the performance of disks that don't use performance tiers, like Ultra Disks or Premium SSD v2, see either [Adjust the performance of an Ultra Disk](/azure/virtual-machines/disks-enable-ultra-ssd?tabs=azure-portal#adjust-the-performance-of-an-ultra-disk) or [Adjust disk performance of a Premium SSD v2](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli#adjust-disk-performance).

The performance of your Azure managed disk is set when you create your disk, in the form of its performance tier. The performance tier determines the IOPS and throughput your managed disk has. When you set the provisioned size of your disk, a performance tier is automatically selected. You can change the performance tier at deployment or afterwards, without changing the size of the disk and without downtime. To learn more about performance tiers, see [Performance tiers for managed disks](/azure/virtual-machines/disks-change-performance).

Changing your performance tier has billing implications. See [Billing impact](/azure/virtual-machines/disks-change-performance#billing-impact) for details.

## Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](./includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## Prerequisites
# [Azure CLI](#tab/azure-cli)
Install the latest [Azure CLI](/cli/azure/install-az-cli2) and sign in to an Azure account with [az login](/cli/azure/reference-index).

# [PowerShell](#tab/azure-powershell)
Install the latest [Azure PowerShell version](/powershell/azure/install-azure-powershell), and sign in to Azure with [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

# [Azure portal](#tab/portal)

Not applicable.

---

## Create an empty data disk with a tier higher than the baseline tier

Performance tiers apply only to Premium SSD managed disks. Select a tier at or above the baseline tier for the disk size. The P60, P70, and P80 tiers require a disk larger than 4,096 GiB.

# [Azure CLI](#tab/azure-cli)

Use the [az disk create](/cli/azure/disk#az-disk-create) command to create an empty Premium SSD data disk with the specified performance tier.

```azurecli
subscriptionId=<yourSubscriptionIDHere>
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
diskSize=<yourDiskSizeHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus

az account set --subscription $subscriptionId

az disk create -n $diskName -g $resourceGroupName -l $region --sku Premium_LRS --size-gb $diskSize --tier $performanceTier
```

Use [az disk show](/cli/azure/disk#az-disk-show) to confirm that the `tier` value matches `$performanceTier`.

```azurecli
az disk show -n $diskName -g $resourceGroupName --query tier -o tsv
```

### Create an OS disk with a tier higher than the baseline tier from an Azure Marketplace image

Select a tier at or above the baseline tier for the OS disk size. The following example creates an OS disk from an Azure Marketplace image by using [az disk create](/cli/azure/disk#az-disk-create).

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus
image=Canonical:UbuntuServer:18.04-LTS:18.04.202002180

az disk create -n $diskName -g $resourceGroupName -l $region --image-reference $image --sku Premium_LRS --tier $performanceTier
```

Use [az disk show](/cli/azure/disk#az-disk-show) to confirm that the `tier` value matches `$performanceTier`.

```azurecli
az disk show -n $diskName -g $resourceGroupName --query tier -o tsv
```

# [PowerShell](#tab/azure-powershell)

Use the [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) and [New-AzDisk](/powershell/module/az.compute/new-azdisk) cmdlets to create an empty Premium SSD data disk with the specified performance tier.

```azurepowershell
$subscriptionId='yourSubscriptionID'
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'
$diskSizeInGiB=4
$performanceTier='P50'
$sku='Premium_LRS'
$region='westcentralus'

Connect-AzAccount

Set-AzContext -Subscription $subscriptionId

$diskConfig = New-AzDiskConfig -SkuName $sku -Location $region -CreateOption Empty -DiskSizeGB $diskSizeInGiB -Tier $performanceTier
New-AzDisk -DiskName $diskName -Disk $diskConfig -ResourceGroupName $resourceGroupName
```

Use [Get-AzDisk](/powershell/module/az.compute/get-azdisk) to confirm that the `Tier` value matches `$performanceTier`.

```azurepowershell
(Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName).Tier
```

# [Azure portal](#tab/portal)

The following steps show how to change the performance tier of your disk when you first create the disk:

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Go to the VM that uses the new disk.
1. Create the disk and select the disk size you need.
1. Select a performance tier that's higher than the baseline tier for that disk size.
1. Select **OK** to create the disk.
1. Open the new disk's **Size + performance** page and confirm that **Performance tier** shows the tier you selected.

:::image type="content" source="media/disks-performance-tiers-portal/new-disk-change-performance-tier.png" alt-text="Screenshot of the Azure portal disk size page with the 1,024-GiB P30 disk selected and the performance tier list offering P30, P40, and P50." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::


---

## Update the tier of a disk without downtime

You can change the performance tier of a Premium SSD managed disk without deallocating the VM or detaching the disk. For a shared disk, stop all attached VMs before changing the tier. You can't select a tier that's lower than the disk's baseline tier. You can downgrade the tier only once every 12 hours.

# [Azure CLI](#tab/azure-cli)

1. Use the [az disk update](/cli/azure/disk#az-disk-update) command to update the tier of a disk, even when the disk is attached to a running VM.

    ```azurecli
    resourceGroupName=<yourResourceGroupNameHere>
    diskName=<yourDiskNameHere>
    performanceTier=<yourDesiredPerformanceTier>

    az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
    ```

1. Use [az disk show](/cli/azure/disk#az-disk-show) to confirm that the `tier` value matches `$performanceTier`.

    ```azurecli
    az disk show -n $diskName -g $resourceGroupName --query tier -o tsv
    ```

# [PowerShell](#tab/azure-powershell)

1. Use the [New-AzDiskUpdateConfig](/powershell/module/az.compute/new-azdiskupdateconfig) and [Update-AzDisk](/powershell/module/az.compute/update-azdisk) cmdlets to update the tier of a disk, even when the disk is attached to a running VM.

    ```azurepowershell
    $resourceGroupName='yourResourceGroupName'
    $diskName='yourDiskName'
    $performanceTier='P1'

    $diskUpdateConfig = New-AzDiskUpdateConfig -Tier $performanceTier

    Update-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -DiskUpdate $diskUpdateConfig
    ```

1. Use [Get-AzDisk](/powershell/module/az.compute/get-azdisk) to confirm that the `Tier` value matches `$performanceTier`.

    ```azurepowershell
    (Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName).Tier
    ```

# [Azure portal](#tab/portal)

A disk's performance tier can be changed without downtime, so you don't have to deallocate your VM or detach your disk to change the tier.

1. Navigate to the VM containing the disk you'd like to change.
1. Select your disk
1. Select **Size + Performance**.
1. In the **Performance tier** dropdown, select a tier other than the disk's current performance tier.
1. Select **Resize**.
1. Confirm that **Performance tier** shows the tier you selected.

:::image type="content" source="media/disks-performance-tiers-portal/change-tier-existing-disk.png" alt-text="Screenshot of the Azure portal Size + performance page with the Performance tier set to P10 at 500 IOPS and 100 MBps and the Resize button highlighted." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::

---

## Show the tier of a disk

# [Azure CLI](#tab/azure-cli)

Use the [az disk show](/cli/azure/disk#az-disk-show) command to return the current performance tier.

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>

az disk show -n $diskName -g $resourceGroupName --query tier -o tsv
```

# [PowerShell](#tab/azure-powershell)

Use the [Get-AzDisk](/powershell/module/az.compute/get-azdisk) cmdlet to retrieve the disk and return its current performance tier.

```azurepowershell
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'

$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName

$disk.Tier
```

# [Azure portal](#tab/portal)

To find a disk's current performance tier in the Azure portal, navigate to that individual disk's **Size + Performance** page and examine the **Performance tier** dropdown's default selection.

---

## Next steps

If you need to resize a disk to take advantage of the higher performance tiers, see these articles:

- [Expand virtual hard disks on a Linux VM with the Azure CLI](linux/expand-disks.md)
- [Expand a managed disk attached to a Windows virtual machine](windows/expand-disks.md)
