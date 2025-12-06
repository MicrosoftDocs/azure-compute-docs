---
title: Change the performance of Azure managed disks
description: Learn how to change performance tiers for existing managed disks using the Azure PowerShell module, the Azure CLI, or the Azure portal.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.update-cycle: 90-days
ms.date: 01/15/2025
ms.author: rogarana
ms.custom: devx-track-azurecli, devx-track-azurepowershell, portal
# Customer intent: "As a cloud administrator, I want to change the performance tier of managed disks without downtime, so that I can optimize resources for my virtual machines efficiently without interruption to their operations."
---

# Change your performance tier without downtime

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

[!INCLUDE [virtual-machines-disks-performance-tiers-intro](./includes/virtual-machines-disks-performance-tiers-intro.md)]

## Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](./includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## Prerequisites
# [Azure CLI](#tab/azure-cli)
Install the latest [Azure CLI](/cli/azure/install-az-cli2) and sign in to an Azure account with [az login](/cli/azure/reference-index).

# [PowerShell](#tab/azure-powershell)
Install the latest [Azure PowerShell version](/powershell/azure/install-azure-powershell), and sign in to an Azure account in with `Connect-AzAccount`.

# [Azure portal](#tab/portal)

Not applicable.

---

## Create an empty data disk with a tier higher than the baseline tier
# [Azure CLI](#tab/azure-cli)

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
## Create an OS disk with a tier higher than the baseline tier from an Azure Marketplace image

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus
image=Canonical:UbuntuServer:18.04-LTS:18.04.202002180

az disk create -n $diskName -g $resourceGroupName -l $region --image-reference $image --sku Premium_LRS --tier $performanceTier
```

# [PowerShell](#tab/azure-powershell)

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

# [Azure portal](#tab/portal)

The following steps show how to change the performance tier of your disk when you first create the disk:

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Navigate to the VM you'd like to create a new disk for.
1. When selecting the new disk, first choose the size, of disk you need.
1. Once you've selected a size, then select a different performance tier, to change its performance.
1. Select **OK** to create the disk.

:::image type="content" source="media/disks-performance-tiers-portal/new-disk-change-performance-tier.png" alt-text="Screenshot of the disk creation blade, a disk is highlighted, and the performance tier dropdown is highlighted." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::


---

## Update the tier of a disk without downtime

A disk's performance tier can be changed without downtime, so you don't have to deallocate your VM or detach your disk to change the tier.

# [Azure CLI](#tab/azure-cli)

1. Update the tier of a disk even when it is attached to a running VM

    ```azurecli
    resourceGroupName=<yourResourceGroupNameHere>
    diskName=<yourDiskNameHere>
    performanceTier=<yourDesiredPerformanceTier>

    az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
    ```

# [PowerShell](#tab/azure-powershell)

1. Update the tier of a disk even when it is attached to a running VM

    ```azurepowershell
    $resourceGroupName='yourResourceGroupName'
    $diskName='yourDiskName'
    $performanceTier='P1'

    $diskUpdateConfig = New-AzDiskUpdateConfig -Tier $performanceTier

    Update-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -DiskUpdate $diskUpdateConfig
    ```

# [Azure portal](#tab/portal)

A disk's performance tier can be changed without downtime, so you don't have to deallocate your VM or detach your disk to change the tier.

1. Navigate to the VM containing the disk you'd like to change.
1. Select your disk
1. Select **Size + Performance**.
1. In the **Performance tier** dropdown, select a tier other than the disk's current performance tier.
1. Select **Resize**.

:::image type="content" source="media/disks-performance-tiers-portal/change-tier-existing-disk.png" alt-text="Screenshot of the size + performance blade, performance tier is highlighted." lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::

---

## Show the tier of a disk

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk show -n $diskName -g $resourceGroupName --query [tier] -o tsv
```

# [PowerShell](#tab/azure-powershell)

```azurepowershell
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName

$disk.Tier
```

# [Azure portal](#tab/portal)

To find a disk's current performance tier in the Azure portal, navigate to that individual disk's **Size + Performance** page and examine the **Performance tier** dropdown's default selection.

---

## Next steps

If you need to resize a disk to take advantage of the higher performance tiers, see these articles:

- [Expand virtual hard disks on a Linux VM with the Azure CLI](linux/expand-disks.md)
- [Expand a managed disk attached to a Windows virtual machine](windows/expand-os-disk.md)
