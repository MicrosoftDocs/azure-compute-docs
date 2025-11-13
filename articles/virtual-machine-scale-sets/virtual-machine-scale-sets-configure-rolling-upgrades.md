---
title: Configure rolling upgrades on Virtual Machine Scale Sets
description: Learn about how to configure rolling upgrades on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 03/21/2025
ms.reviewer: ju-shim
ms.custom: upgradepolicy, maxsurge, ignite-2024
# Customer intent: As a cloud administrator, I want to configure rolling upgrades for Virtual Machine Scale Sets, so that I can apply updates safely without downtime and maintain high availability of my applications.
---
# Configure rolling upgrades on Virtual Machine Scale Sets

Using a rolling upgrade policy is the safest way to apply updates to instances in a Virtual Machine Scale Set. Performing updates in batches ensures that your scale set maintains a set number of instances available to take traffic, meaning you don't need to take down your entire workload to make a change. 

Rolling upgrade policy is best suited for production workloads.

## Requirements

- When using a rolling upgrade policy on Virtual Machine Scale Sets with Flexible Orchestration, the scale set must also use the [application health extension](virtual-machine-scale-sets-health-extension.md) to monitor application health.

- When using a rolling upgrade policy on Virtual Machine Scale Sets with Uniform Orchestration, the scale set must also have a [health probe](/azure/load-balancer/load-balancer-custom-probe-overview) or use the [application health extension](virtual-machine-scale-sets-health-extension.md) to monitor application health. 

- If attaching a virtual machine to a Virtual Machine Scale Set, the attached virtual machine should also have an application health extension. If an attached virtual machine does not have a health extension configured and reporting application health, the rolling upgrade may be impacted. 

## Concepts

### Upgrade policy mode vs rolling upgrade policy
The **upgrade policy mode** and the **rolling upgrade policy** are nested, but separate settings of the **upgrade policy**. This means that while your scale set may use an automatic or manual upgrade policy mode, there is still a rolling upgrade policy that is associated. This rolling upgrade policy comes with default settings but can also be configured to meet your business needs. 

```json
"properties": {
        "upgradePolicy": {
            "mode": "<Automatic, Manual, Rolling>",
            "rollingUpgradePolicy": {
            "maxBatchInstancePercent": 20,
            "maxUnhealthyInstancePercent": 20,
            "maxUnhealthyUpgradedInstancePercent": 20,
            "pauseTimeBetweenBatches": "PT2S",
            "MaxSurge": "false"
            }
        }
    }
```

For scale set model updates, the upgrade policy mode will determine how the instances in the scale set are brought up to date. 

> [!IMPORTANT]
> If you are using **[automatic OS image upgrades](virtual-machine-scale-sets-automatic-upgrade.md)** or **[automatic extension upgrades](../virtual-machines/automatic-extension-upgrade.md)**, these feature don't use the upgrade policy mode. Instead these features use the rolling upgrade policy configuration settings to perform rolling upgrades.

### Rolling upgrade policy configuration settings


|Setting | Description |
|---|---|
|**Rolling upgrade batch size %** | Specifies how many of the total instances of your scale set you want to be upgraded at one time. <br><br>Example: A batch size of 20% when you have 10 instances in your scale set results in upgrade batches with two instances each. |
|**Pause time between batches (sec)** | Specifies how long you want your scale set to wait between finishing an upgrade batch and starting a new one.<br><br> Example: A pause time of 10 seconds means that once a batch is successfully completed, the scale set will wait 10 seconds before moving onto the next batch. |
|**Max unhealthy instance %** | Specifies the total number of instances allowed to be marked as unhealthy before and during the rolling upgrade. <br><br>Example: A max unhealthy instance % of 20 means if you have a scale set of 10 instances and more than two instances in the entire scale set report back as unhealthy, the rolling upgrade stops. |
| **Max unhealthy upgrade %**| Specifies the total number of instances allowed to be marked as unhealthy after being upgraded. <br><br>Example: A max unhealthy upgrade % of 20 means if you have a scale set of 10 instances and more than two instances in the entire scale set report back as unhealthy after being upgraded, the rolling upgrade is canceled. |
|**Prioritize unhealthy instances** | Tells the scale set to upgrade instances reporting as unhealthy before upgrading instances reporting as healthy. <br><br>Example: If some instances in your scale are failed or unhealthy when a rolling upgrade begins, the scale set updates those instances first. |
| **Enable cross-zone upgrade** | Allows the scale set to ignore Availability Zone boundaries when determining batches. This essentially lets the rolling upgrade treat your scale set as a regional deployment instead of a zone-aware deployment. |
| **MaxSurge** | New instances are created in batches using the latest scale model. Once the batch of new instances is successfully created and marked as healthy, the scale set then deletes instances in batches matching the old scale set model. This continues until all instances are brought up-to-date. rolling upgrades with MaxSurge can help improve service uptime during upgrade events. <br><br>For more information see [MaxSurge rolling upgrades](virtual-machine-scale-sets-maxsurge.md). |
| **Rollback Failed Instances On Policy Breach (Uniform Orchestration only)** | Rollback failed instances to previous model if the rolling upgrade policy is violated. |


## Setting or updating the rolling upgrade policy

Rolling upgrade policy can be configured during scale set creation. Because rolling upgrade policy requires successfully monitoring application health and there are specific settings that determine how upgrades are completed, it's suggested to first create your scale set using manual upgrade mode. Once you have confirmed the application health is being successfully reported, update your upgrade policy from manual to rolling.

### [Portal](#tab/portal1)

Select the Virtual Machine Scale Set you want to change the upgrade policy for. In the menu under **Settings**, select **Upgrade Policy** and from the drop-down menu, select **Rolling - Upgrades roll out in batches with optional pause**. 

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/rolling-upgrade-policy-portal.png" alt-text="Screenshot showing changing the upgrade policy and enabling MaxSurge in the Azure portal.":::

### [CLI](#tab/cli1)
Update an existing Virtual Machine Scale Set using [az vmss update](/cli/azure/vmss#az-vmss-update). 

```azurecli-interactive
az vmss update \
	--name myScaleSet \
	--resource-group myResourceGroup \
     --set upgradePolicy.mode=Rolling \
	--max-batch-instance-percent 10 \
	--max-unhealthy-instance-percent 20 \
	--max-unhealthy-upgraded-instance-percent 20 \
	--prioritize-unhealthy-instances true \
	--pause-time-between-batches PT2S \
	--max-surge true 

```

### [PowerShell](#tab/powershell1)
Update an existing Virtual Machine Scale Set using [Update-AzVmss](/powershell/module/az.compute/update-azvmss). 

```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

Set-AzVmssRollingUpgradePolicy `
   -VirtualMachineScaleSet $VMSS `
   -MaxBatchInstancePercent 20 `
   -MaxUnhealthyInstancePercent 20 `
   -MaxUnhealthyUpgradedInstancePercent 20 `
   -PauseTimeBetweenBatches "PT30S" `
   -EnableCrossZoneUpgrade True `
   -PrioritizeUnhealthyInstance True `
   -MaxSurge True

Update-Azvmss -ResourceGroupName "myResourceGroup" `
    -Name "myScaleSet" `
    -UpgradePolicyMode "Rolling" `
    -VirtualMachineScaleSet $vmss
```

### [ARM Template](#tab/template1)

Update the properties section of your ARM template and set the upgrade policy to rolling and various rolling upgrade options.  


```json
"properties": {
    "singlePlacementGroup": false,
        "upgradePolicy": {
            "mode": "Rolling",
            "rollingUpgradePolicy": {
            "maxBatchInstancePercent": 20,
            "maxUnhealthyInstancePercent": 20,
            "maxUnhealthyUpgradedInstancePercent": 20,
            "pauseTimeBetweenBatches": "PT2S",
	        "MaxSurge": "true"
            }
        }
    }
```
---


## Get rolling upgrade Status

### [Portal](#tab/portal2)

When a rolling upgrade is triggered in the Azure portal, a banner appears at the top of your scale set resource informing you that a rolling upgrade is in progress. You can click on view details to see the status of the rolling upgrade. When all updates are complete, the banner disappears. 

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/rolling-upgrade-banner.png" alt-text="Screenshot showing banner when a rolling upgrade is taking place.":::

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/rolling-upgrade-status-portal.png" alt-text="Screenshot showing details of the rolling upgrade in the Azure portal.":::

Additionally, you can view exactly what changes are being rolled out in the Activity Log. The rolling upgrade event is under **Create or Update Virtual Machine Scale Set**. Select **Change History** to review what is being updated. 

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/rolling-upgrade-activity-log.png" alt-text="Screenshot showing the rolling upgrade details in the Activity Log.":::


### [CLI](#tab/cli2)

You can get the status of a rolling upgrade in progress using [az vmss rolling-upgrade get-latest](/cli/azure/vmss#az-vmss-rolling-upgrade).

```azurecli
az vmss rolling-upgrade get-latest \
    --name myScaleSet \
    --resource-group myResourceGroup

{
  "location": "eastus",
  "policy": {
    "maxBatchInstancePercent": 20,
    "maxSurge": false,
    "maxUnhealthyInstancePercent": 20,
    "maxUnhealthyUpgradedInstancePercent": 20,
    "pauseTimeBetweenBatches": "PT0S",
    "prioritizeUnhealthyInstances": true,
    "rollbackFailedInstancesOnPolicyBreach": false
  },
  "progress": {
    "failedInstanceCount": 0,
    "inProgressInstanceCount": 2,
    "pendingInstanceCount": 0,
    "successfulInstanceCount": 8
  },
  "runningStatus": {
    "code": "RollingForward",
    "lastAction": "Start",
    "lastActionTime": "2024-01-12T20:20:04.1863772+00:00",
    "startTime": "2024-01-12T20:20:04.4363788+00:00"
  },
  "type": "Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades"
}
```

### [PowerShell](#tab/powershell2)

You can get the status of a rolling upgrade in progress using [Get-AzVmssRollingUpgrade](/powershell/module/az.compute/get-azvmssrollingupgrade).

```azurepowershell
Get-AzVMssRollingUpgrade `
    -ResourceGroupName myResourceGroup `
    -VMScaleSetName myScaleSet

Policy                                  : 
  MaxBatchInstancePercent               : 20
  MaxUnhealthyInstancePercent           : 20
  MaxUnhealthyUpgradedInstancePercent   : 20
  PauseTimeBetweenBatches               : PT0S
  PrioritizeUnhealthyInstances          : True
  RollbackFailedInstancesOnPolicyBreach : False
  MaxSurge                              : False
RunningStatus                           : 
  Code                                  : RollingForward
  StartTime                             : 1/12/2024 8:20:04 PM
  LastAction                            : Start
  LastActionTime                        : 1/12/2024 8:20:04 PM
Progress                                : 
  SuccessfulInstanceCount               : 8
  FailedInstanceCount                   : 0
  InProgressInstanceCount               : 2
  PendingInstanceCount                  : 0
Type                                    : Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades
Location                                : eastus
Tags                                    : {}
```
---

## Cancel a rolling upgrade

### [Portal](#tab/portal3)
You can cancel a rolling upgrade in progress using the Azure portal by selecting the **view details** in the banner above your scale set. In the pop-up window, you can view the current status and at the bottom is a **cancel upgrade** option. 

:::image type="content" source="../virtual-machine-scale-sets/media/upgrade-policy/rolling-upgrade-cancel-portal.png" alt-text="Screenshot showing the rolling upgrade details in the Activity Log.":::


### [CLI](#tab/cli3)
You can stop a rolling upgrade in progress using [az vmss rolling-upgrade cancel](/cli/azure/vmss#az-vmss-rolling-upgrade). If you don't see any output after running the command, it means the cancel request was successful.

```azurecli
az vmss rolling-upgrade cancel \
    --name myScaleSet \
    --resource-group myResourceGroup
```

### [PowerShell](#tab/powershell3)
You can stop a rolling upgrade in progress using [Stop-AzVmssRollingUpgrade](/powershell/module/az.compute/stop-azvmssrollingupgrade).

```azurepowershell
Stop-AzVmssRollingUpgrade `
    -ResourceGroupName myResourceGroup `
    -VMScaleSetName myScaleSet
```
---

## Restart a rolling upgrade

If you decide to cancel a rolling upgrade or the upgrade has stopped due to any policy breach, any more changes that result in another scale set model change trigger a new rolling upgrade. If you want to restart a rolling upgrade, trigger a generic model update. This tells the scale set to check if all the instances are up to date with the latest model.

### [CLI](#tab/cli4)
To restart a rolling upgrade after it has been canceled, trigger the scale set to check if the instances in the scale set are up to date with the latest scale set model. You can do this by running [az vmss update](/cli/azure/vmss#az-vmss-update).

```azurecli
az vmss update \
    --name myScaleSet \
    --resource-group myResourceGroup
```

### [PowerShell](#tab/powershell4)
To restart a rolling upgrade after it's been canceled, you need to trigger the scale set to check if the instances in the scale set are up to date with the latest scale set model. You can do this by running [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

```azurepowershell
$VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

Update-AzVmss `
    -ResourceGroupName myResourceGroup `
    -Name myScaleSet `
    -VirtualMachineScaleSet $VMSS
```    
---


## Next steps
Learn how to [perform manual upgrades](virtual-machine-scale-sets-perform-manual-upgrades.md) on Virtual Machine Scale Sets. 
