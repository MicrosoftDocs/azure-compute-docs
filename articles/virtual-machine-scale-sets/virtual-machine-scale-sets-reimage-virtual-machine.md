---
title: Reimage a virtual machine in a scale set
description: Learn how to reimage a virtual machine in a scale set.
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 05/19/2026
ms.reviewer: wwilliams
ms.custom: upgradepolicy, ignite-2024 
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator managing Virtual Machine Scale Sets, I want to reimage individual instances, so that I can apply necessary OS and configuration updates to ensure optimal performance and security."
---

# Reimage a virtual machine

When updating an instance in a Virtual Machine Scale Set, there are some changes that can't be applied to existing instances without performing a reimage. Reimaging a virtual machine in a Virtual Machine Scale Set replaces the old OS disk with a new OS disk. This allows changes to the OS, data disk profile (such as admin username and password), and [custom data](../virtual-machines/custom-data.md) to be applied. To reimage a set of existing instances in a scale set, you must individually reimage each instance. 

If reimaging a virtual machine using an ephemeral OS disk, the instance is restored to its initial state and any local data is lost. For instances using nonephemeral OS disks, retention of the old OS disk depends on the OS disk's delete option. For more information, see [set delete options when creating a virtual machine](../virtual-machines/delete.md)

Reimaging a virtual machine that was created outside of the scale set and later attached can only be reimaged if the virtual machine OS profile matches the OS profile of the scale set. 

## [Portal](#tab/portal)

In the menu under **Settings**, navigate to **Instances** and select the instances you want to reimage. Once selected, click the **Reimage** option.


:::image type="content" source="../virtual-machine-scale-sets/media/maxsurge/reimage-upgrade-1.png" alt-text="Screenshot showing reimaging scale set instances using the Azure portal.":::


## [CLI](#tab/cli)
To reimage one or more instances using Azure CLI, use the [az vmss reimage](/cli/azure/vmss#az-vmss-reimage) command. The `--instance-ids` parameter accepts one or more space-separated instance identifiers: the instance ID if using Uniform Orchestration, or the instance name if using Flexible Orchestration. When `--instance-ids` is omitted, all virtual machines in the scale set are reimaged.

To reimage specific instances, provide one or more instance IDs:

```azurecli-interactive
az vmss reimage \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --instance-ids instanceId1 instanceId2
```

To reimage all instances in the scale set, omit `--instance-ids`:

```azurecli-interactive
az vmss reimage \
    --resource-group myResourceGroup \
    --name myScaleSet
```

## [PowerShell](#tab/powershell)
To reimage a specific instance using Azure PowerShell, use the [Set-AzVmssVM](/powershell/module/az.compute/set-azvmssvm) command.  The `instanceid` parameter refers to the ID of the instance if using Uniform Orchestration and the Instance name if using Flexible Orchestration. 

```azurepowershell-interactive
Set-AzVmssVM `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -InstanceId instanceId -Reimage
```

## [REST API](#tab/rest)
To reimage scale set instances using REST, use the [reimage](/rest/api/compute/virtualmachinescalesets/reimage) command. You can specify multiple instances to be reimaged in the request body. 

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/reimage?api-version={apiVersion}
```

**Request Body**
```HTTP
{
  "instanceIds": [
    "myScaleSet1",
    "myScaleSet2"
  ]
}
```
---

## Next steps
Learn how to [set the Upgrade Policy](virtual-machine-scale-sets-set-upgrade-policy.md) of your Virtual Machine Scale Set.
