---
title: Change the upgrade policy mode on Virtual Machine Scale Sets
description: Learn how to change the upgrade policy mode on Virtual Machine Scale Sets
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 11/7/2024
ms.reviewer: ju-shim
ms.custom: upgradepolicy, ignite-2024
# Customer intent: As a cloud administrator, I want to change the upgrade policy mode on Virtual Machine Scale Sets, so that I can manage software updates effectively during different stages of my deployment process.
---
# Change the upgrade policy mode on Virtual Machine Scale Sets

The upgrade policy mode for a Virtual Machine Scale Set can be changed at any point in time. Depending on your scenario, you may want to use a particular upgrade policy mode when setting up and developing your workload and once you're ready to move to production, change it to another upgrade policy mode. 

### [Portal](#tab/portal)

Select the Virtual Machine Scale Set you want to change the upgrade policy mode for. In the menu under **Settings**, select **Upgrade Policy** and from the drop-down menu, select the upgrade policy mode you want to enable. 

If using a rolling upgrade policy mode, see [configure rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) for more configuration options and suggestions.

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/change-upgrade-policy.png" alt-text="Screenshot showing changing the upgrade policy mode and enabling MaxSurge in the Azure portal.":::

### [CLI](#tab/cli)
Update an existing Virtual Machine Scale Set using [az vmss update](/cli/azure/vmss#az-vmss-update) and the `--upgrade-policy-mode` parameter. 

If using a rolling upgrade policy mode, see [configure rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) for more configuration options and suggestions.

```azurecli-interactive
az vmss update \
    --name myScaleSet \
    --resource-group myResourceGroup \
    --upgrade-policy-mode Automatic
```

### [PowerShell](#tab/powershell)
Update an existing Virtual Machine Scale Set using [Update-AzVmss](/powershell/module/az.compute/update-azvmss) and the `-UpgradePolicyMode` to set the upgrade policy mode. 

If using a rolling upgrade policy mode, see [configure rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) for more configuration options and suggestions.

```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

Update-Azvmss `
    -ResourceGroupName "myResourceGroup" `
    -Name "myScaleSet" `
    -UpgradePolicyMode "Manual" `
    -VirtualMachineScaleSet $vmss
```

### [ARM Template](#tab/template)

Update the properties section of your ARM template with the upgrade policy mode. 

If using a rolling upgrade policy mode, see [configure rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) for more configuration options and suggestions.


```json
"properties": {
        "upgradePolicy": {
            "mode": "automatic",
        }
    }
```
---


## Next steps
Learn how to [configure rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) on Virtual Machine Scale Sets. 
