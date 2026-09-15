---
title: Enable on-demand disk bursting
description: Enable on-demand disk bursting on an Azure Premium SSD managed disk.
author: roygara
ms.author: rogarana
ms.date: 09/11/2026
ms.topic: how-to
ms.service: azure-disk-storage
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, portal
ai-usage: ai-assisted
# Customer intent: "As an IT administrator, I want to enable on-demand disk bursting for my managed SSDs, so that I can allow workloads to exceed their provisioned limits as needed for performance scalability."
---

# Enable on-demand disk bursting

Azure Premium SSD managed disks have two available bursting models: credit-based bursting and on-demand bursting. This article explains how to create a Premium SSD managed disk with on-demand bursting or enable on-demand bursting on an existing Premium SSD managed disk. Disks that use the on-demand model can burst beyond their original provisioned targets. On-demand bursting occurs as often as needed by the workload, up to the maximum burst target. On-demand bursting incurs additional charges.

For details on disk bursting, see [managed disk bursting](disk-bursting.md). 

For the max burst targets on each supported disk, see [Scalability and performance targets for VM disks](disks-scalability-targets.md#premium-ssd-managed-disks-per-disk-limits).

> [!IMPORTANT]
> You don't need to follow the steps in this article to use credit-based bursting. By default, credit-based bursting is enabled on all eligible disks.

Before you enable on-demand bursting, understand the following:

[!INCLUDE [managed-disk-bursting-regions-limitations](./includes/managed-disk-bursting-regions-limitations.md)]

## Enable on-demand bursting by deployment method

You can enable on-demand bursting with the Azure portal, Azure PowerShell, Azure CLI, or an Azure Resource Manager (ARM) template. The following examples show how to create a Premium SSD managed disk with on-demand bursting or enable on-demand bursting on an existing Premium SSD managed disk.

# [Portal](#tab/azure-portal)

To enable on-demand bursting, an existing Premium SSD managed disk must be larger than 512 GiB and either detached from its VM or attached to a stopped VM.

To enable on-demand bursting for an existing disk:

1. Sign in to the [Azure portal](https://portal.azure.com/) and navigate to your disk.
1. Select **Configuration** and select **Enable on-demand bursting**.
1. Select **Save**.

# [PowerShell](#tab/azure-powershell)

On-demand bursting cmdlets are available in version 5.5.0 and newer of the Az PowerShell module. Alternatively, you may use the [Azure Cloud Shell](https://shell.azure.com/).

### Create an empty data disk with on-demand bursting

To enable on-demand bursting, a Premium SSD managed disk must be larger than 512 GiB. Replace the `<myResourceGroupDisk>` and `<myDataDisk>` parameters in the following script. The script uses [Set-AzContext](/powershell/module/az.accounts/set-azcontext) to select the subscription, [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) to create a disk configuration, and [New-AzDisk](/powershell/module/az.compute/new-azdisk) to create a Premium SSD managed disk with on-demand bursting:

```azurepowershell
Set-AzContext -SubscriptionName <yourSubscriptionName>

$diskConfig = New-AzDiskConfig -Location 'WestCentralUS' -CreateOption Empty -DiskSizeGB 1024 -SkuName Premium_LRS -BurstingEnabled $true

$dataDisk = New-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk> -Disk $diskConfig
```

### Enable on-demand bursting on an existing disk

To enable on-demand bursting, an existing Premium SSD managed disk must be larger than 512 GiB and either detached from its VM or attached to a stopped VM. Replace the `<myResourceGroupDisk>` and `<myDataDisk>` parameters in the following command. The command uses [New-AzDiskUpdateConfig](/powershell/module/az.compute/new-azdiskupdateconfig) to create an update configuration and passes it to [Update-AzDisk](/powershell/module/az.compute/update-azdisk) to update the disk. To disable on-demand bursting, set `-BurstingEnabled` to `$false`.

```azurepowershell
New-AzDiskUpdateConfig -BurstingEnabled $true | Update-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk>
```

# [Azure CLI](#tab/azure-cli)

On-demand bursting commands are available in version 2.19.0 and newer of the [Azure CLI](/cli/azure/install-azure-cli). Alternatively, you can use [Azure Cloud Shell](https://shell.azure.com/).

### Create and attach an on-demand bursting data disk

To enable on-demand bursting, a Premium SSD managed disk must be larger than 512 GiB. Replace the `<yourDiskName>`, `<yourResourceGroup>`, and `<yourVMName>` parameters in the following commands. Use [az disk create](/cli/azure/disk#az-disk-create) to create a Premium SSD managed disk with on-demand bursting, and then use [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) to attach it to a VM:

```azurecli
az disk create -g <yourResourceGroup> -n <yourDiskName> --size-gb 1024 --sku Premium_LRS -l westcentralus --enable-bursting true

az vm disk attach --vm-name <yourVMName> --name <yourDiskName> --resource-group <yourResourceGroup>
```

### Enable on-demand bursting on an existing disk

To enable on-demand bursting, an existing Premium SSD managed disk must be larger than 512 GiB and either detached from its VM or attached to a stopped VM. Replace the `<yourResourceGroup>` and `<yourDiskName>` parameters in the following [az disk update](/cli/azure/disk#az-disk-update) command. To disable on-demand bursting, set `--enable-bursting` to `false`.

```azurecli
az disk update --name <yourDiskName> --resource-group <yourResourceGroup> --enable-bursting true
```

# [Azure Resource Manager](#tab/azure-resource-manager)

By using the `2020-09-30` disk API, you can enable on-demand bursting on newly created or existing Premium SSD managed disks larger than 512 GiB. The `2020-09-30` API introduced the `burstingEnabled` property, which is set to `false` by default. The following sample template creates a 1-TiB Premium SSD managed disk in West Central US with on-demand bursting enabled:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string"
    },
    "diskSkuName": {
      "type": "string",
      "defaultValue": "Premium_LRS"
    },
    "dataDiskSizeInGb": {
      "type": "int",
      "defaultValue": 1024
    },
    "location": {
      "type": "string",
      "defaultValue": "westcentralus"
    },
    "diskApiVersion": {
      "type": "string",
      "defaultValue": "2020-09-30"
    }
  },
  "resources": [
    {
      "apiVersion": "[parameters('diskApiVersion')]",
      "type": "Microsoft.Compute/disks",
      "name": "[parameters('diskName')]",
      "location": "[parameters('location')]",
      "properties": {
        "creationData": {
          "createOption": "Empty"
        },
        "diskSizeGB": "[parameters('dataDiskSizeInGb')]",
        "burstingEnabled": true
      },
      "sku": {
        "name": "[parameters('diskSkuName')]"
      }
    }
  ]
}
```
---
 
## Next steps

To learn how to gain insight into your bursting resources, see [Disk bursting metrics](disks-metrics.md).

