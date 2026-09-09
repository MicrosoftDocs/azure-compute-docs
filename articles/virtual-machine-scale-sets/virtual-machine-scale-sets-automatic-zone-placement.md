---
title: Automatic zone placement for Virtual Machine Scale Sets (Preview)
description: Learn how automatic zone placement selects availability zones and how to configure it for Azure Virtual Machine Scale Sets.
author: hilaryw29
ms.author: hilarywang
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 08/11/2026
ms.reviewer: fisteele
ms.custom: devx-track-arm-template, devx-track-azurecli, devx-track-azurepowershell
ai-usage: ai-assisted
---

# Configure automatic zone placement for a scale set (preview)

Automatic zone placement for Virtual Machine Scale Sets lets Azure select availability zones for your scale set based on capacity and SKU availability. Instead of specifying a fixed list of zones for every region, you configure your placement requirements and Azure selects the zones that best satisfy them.

Automatic zone placement is supported for Virtual Machine Scale Sets with Uniform and Flexible orchestration modes.

> [!IMPORTANT]
> Automatic zone placement for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change before general availability (GA).

## How automatic zone placement works

Automatic zone placement evaluates SKU availability, capacity, and any placement constraints that you configure to select availability zones and place VM instances.

By default, the scale set works toward spreading instances across three availability zones. If you configure `maxZoneCount`, the scale set instead works toward spreading instances across that number of zones and never exceeds the configured limit. Actual placement depends on SKU availability, capacity, the scale set's current zone distribution, and your other placement constraints. When `maxZoneCount` isn't configured and the region has more than three availability zones, the scale set can expand beyond its default three-zone spread, such as into a fourth zone, if its current zones can't satisfy a scale-out request.

By default, automatic zone placement:

- Targets an even distribution across three availability zones.
- Requires instances to be distributed across at least two availability zones.
- Limits the number of instances in any single zone to 50 percent of the scale set.
- Can expand to additional availability zones if the current zones don't have enough capacity for more instances and another zone is available in the region.

If capacity isn't available to satisfy all placement requirements, Azure allocates as many instances as the policy allows and the remaining instances may fail to provision. This behavior is a partial allocation failure, not a failure of the entire request.

> [!NOTE]
> Automatic zone placement selects zones when new instances are created. It doesn't move or recreate existing instances to correct a zone imbalance. To proactively correct an existing imbalance, see [Automatic Zone Balance](auto-zone-balance-overview.md).

## When to use automatic zone placement

Use automatic zone placement when:

- You want a zonal or zone-spanning scale set but don't require instances to be placed in specific availability zones. You can define your zone-spreading requirements and allow Azure to select the zones.
- You deploy across regions where zonal SKU availability differs. Automatic zone placement helps you apply a consistent configuration instead of maintaining region-specific zone lists.
- You want Azure to consider current capacity signals when selecting availability zones.

Use [customer-selected availability zones](virtual-machine-scale-sets-configure-customer-selected-zones.md) instead when your workload must use specific zones for every placement operation.

## Prerequisites

Before you configure automatic zone placement, ensure that:

- You're using Compute API version `2026-03-01` or later.
- You deploy the scale set in a [public Azure region that supports availability zones](/azure/reliability/availability-zones-region-support).

### Register the subscription with AFEC

Your subscription must be registered with the Azure Feature Exposure Control (AFEC) flag `Microsoft.Compute/VmssAutomaticZonePlacement`. This registration is required to enable automatic zone placement feature for your subscription.

### [Portal](#tab/portal-register)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. In the search box, enter *subscriptions*, and then select **Subscriptions**.
1. Select your subscription.
1. Under **Settings**, select **Preview features**.
1. Search for **VmssAutomaticZonePlacement**, and then select the feature.
1. Select **Register**.

### [Azure CLI](#tab/cli-register)

Register the AFEC using Azure CLI:

```azurecli
az feature register \
  --namespace Microsoft.Compute \
  --name VmssAutomaticZonePlacement
```

Check the registration status:

```azurecli
az feature show \
  --namespace Microsoft.Compute \
  --name VmssAutomaticZonePlacement
```

### [Azure PowerShell](#tab/powershell-register)

Register the AFEC using Azure PowerShell:

```azurepowershell
Register-AzProviderFeature `
  -ProviderNamespace Microsoft.Compute `
  -FeatureName VmssAutomaticZonePlacement
```

Check the registration status:

```azurepowershell
Get-AzProviderFeature `
  -ProviderNamespace Microsoft.Compute `
  -FeatureName VmssAutomaticZonePlacement
```

---

## Limitations

- A scale set can use either the `zones` property for [customer-selected zones](virtual-machine-scale-sets-configure-customer-selected-zones.md) or `placement.zonePlacementPolicy` for automatic zone placement, but not both.
- Proximity placement groups aren't supported.
- Capacity reservation groups aren't supported.
- [Instance Mix for Virtual Machine Scale Sets](instance-mix-overview.md) isn't supported during public preview. Expected support by GA.
- Standby pools aren't supported.
- Dedicated host groups aren't supported.
- Edge zones aren't supported.
- Overprovisioning isn't supported. Ensure `overprovision` is set to `false` before you add `placement.zonePlacementPolicy`; otherwise, the deployment fails.
- Service Fabric scale sets aren't supported.
- [Creating and attaching a new VM](virtual-machine-scale-sets-attach-detach-vm.md#attach-a-new-virtual-machine-to-a-virtual-machine-scale-set) isn't supported. You can [attach an existing zonal VM](virtual-machine-scale-sets-attach-detach-vm.md#attach-an-existing-virtual-machine-to-a-virtual-machine-scale-set) if its zone complies with the scale set's placement configuration.

## Configuration properties

The following example shows where automatic zone placement properties appear in a scale set model. Only `placement.zonePlacementPolicy` is required. All other properties are optional.

```json
{
  "placement": {
    "zonePlacementPolicy": "auto",
    "includeZones": [
      "1",
      "2",
      "4"
    ]
  },
  "properties": {
    "zoneBalance": false,
    "resiliencyPolicy": {
      "zoneAllocationPolicy": {
        "maxZoneCount": 2,
        "maxInstancePercentPerZonePolicy": {
          "enabled": true,
          "value": 70
        }
      }
    }
  }
}
```

> [!NOTE]
> Use either `includeZones` or `excludeZones`. You can't configure both properties on the same scale set.

The following properties control automatic zone placement behavior:

| Property | Description |
| --- | --- |
| `placement.zonePlacementPolicy` | **Required.** Set to `"auto"` to enable automatic zone placement. |
| `placement.includeZones` | **Optional.** A list of availability zones that Azure can consider for placement. Azure selects zones only from this list, but might not deploy instances in every listed zone. You can't use this property with `excludeZones`. |
| `placement.excludeZones` | **Optional.** A list of zones that Azure must exclude from placement. You can't use this property with `includeZones`. |
| `zoneBalance` | **Optional.** Controls zone balancing during scaling operations. The default is `false`, which uses best-effort balancing. When you enable `zoneBalance` with automatic zone placement, you must also specify `maxZoneCount`. When `zoneBalance` is `false`, `maxZoneCount` is optional. |
| `resiliencyPolicy.zoneAllocationPolicy.maxZoneCount` | **Optional.** The maximum number of zones that the scale set can use. If you don't specify this property, Azure targets three zones and can expand to additional zones when needed to satisfy a scaling request. |
| `resiliencyPolicy.zoneAllocationPolicy.maxInstancePercentPerZonePolicy.enabled` | **Optional.** Enables or disables the maximum-instance-percent-per-zone policy. This policy is supported only when `zoneBalance` is `false`. |
| `resiliencyPolicy.zoneAllocationPolicy.maxInstancePercentPerZonePolicy.value` | **Optional.** The maximum percentage of a scale set's target capacity that can be allocated to a single zone. If satisfying a request would exceed this value, Azure allocates the instances that comply with the policy, and the remaining instances may fail to provision. |

## Create a scale set with automatic zone placement

The following scenarios show how to configure automatic zone placement and explain the expected placement behavior. Complete the operating system, storage, and network profiles before you deploy the scale set.

### Use default placement behavior

### [REST API](#tab/rest-api-create)

Add `placement.zonePlacementPolicy` to the scale set resource. Use Compute API version `2026-03-01` or later.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2026-03-01
```

```json
{
  "placement": {
    "zonePlacementPolicy": "auto"
  }
}
```

Add this configuration to a complete Virtual Machine Scale Set request.

### [Azure CLI](#tab/cli-create)

Use [az vmss create](/cli/azure/vmss#az-vmss-create) with `zone-placement-policy` set to `Auto`. The command creates the scale set and its associated resources with default settings in an existing resource group.

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 3 \
  --zone-placement-policy Auto
```

### [Azure PowerShell](#tab/powershell-create)

Use [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig) with `ZonePlacementPolicy` set to `Auto`. Configure the operating system, storage, and network profiles before you deploy the scale set.

```azurepowershell
$resourceGroupName = "myResourceGroup"
$scaleSetName = "myScaleSet"
$location = "EastUS"

$vmssConfig = New-AzVmssConfig `
  -Location $location `
  -SkuCapacity 3 `
  -SkuName "Standard_D2s_v5" `
  -ZonePlacementPolicy "Auto"

# Configure the operating system, storage, and network profiles before deployment.

New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

---

#### Expected behavior

If you don't specify any placement constraints, Azure distributes instances evenly across three zones. For example, a scale set with 100 instances might have a distribution of 34, 33, and 33.

If an even three-zone distribution isn't possible, Azure requires instances to span at least two zones and applies a default limit of 50 percent of instances in any one zone. The platform applies this limit even when `maxInstancePercentPerZonePolicy` isn't explicitly configured.

For example, suppose you request 100 instances and available capacity across three zones is 70, 30, and 0. Azure attempts a distribution of 50, 30, and 20. Because the third zone has no capacity, the final distribution is 50, 30, and 0, and 20 instances fail to provision. To prioritize allocation success over the default distribution limit, configure a higher `maxInstancePercentPerZonePolicy.value`.

### Configure eligible zones

Use either `includeZones` or `excludeZones` to control which zones Azure considers during placement. You can't configure both properties on the same scale set.

### [REST API](#tab/rest-api-zones)

The following example restricts Azure to only consider zones `1` and `2` for placement.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2026-03-01
```

```json
{
  "placement": {
    "zonePlacementPolicy": "auto",
    "includeZones": [
      "1",
      "2"
    ]
  }
}
```

Alternatively, use `excludeZones` to prevent Azure from selecting specific zones. The following example excludes zone `3`:

```json
{
  "placement": {
    "zonePlacementPolicy": "auto",
    "excludeZones": [
      "3"
    ]
  }
}
```

### [Azure CLI](#tab/cli-zones)

Use either `include-zones` or `exclude-zones` with [az vmss create](/cli/azure/vmss#az-vmss-create). The following example assumes that `myResourceGroup` exists and restricts Azure to only consider zones `1` and `2` for placement:

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 3 \
  --zone-placement-policy Auto \
  --include-zones 1 2
```

To exclude zone `3` instead, replace `--include-zones 1 2` with `--exclude-zones 3`.

### [Azure PowerShell](#tab/powershell-zones)

Use either `IncludeZone` or `ExcludeZone` with [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig). Configure the operating system, storage, and network profiles before you deploy the scale set.

```azurepowershell
$resourceGroupName = "myResourceGroup"
$scaleSetName = "myScaleSet"
$location = "EastUS"

$vmssConfig = New-AzVmssConfig `
  -Location $location `
  -SkuCapacity 3 `
  -SkuName "Standard_D2s_v5" `
  -ZonePlacementPolicy "Auto" `
  -IncludeZone @("1", "2")

# Configure the operating system, storage, and network profiles before deployment.

New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

To exclude zone `3` instead, replace `-IncludeZone @("1", "2")` with `-ExcludeZone @("3")`.

---

#### Expected behavior

When you configure `includeZones`, Azure selects zones only from the specified list. This setting doesn't guarantee that Azure places instances in every listed zone. Azure continues to consider capacity, SKU availability, the current distribution, and your other placement constraints when placing instances.

When you configure `excludeZones`, Azure doesn't select or place new instances in the excluded zones. All other supported zones remain eligible. You can't combine `includeZones` and `excludeZones` on the same scale set.

### Limit the percentage of instances in one zone

Use `maxInstancePercentPerZonePolicy` to control the maximum percentage of a scale set's target capacity that can be allocated to any one zone. A higher percentage gives Azure more flexibility to satisfy allocation requests but allows more instances to be concentrated in one zone.

The policy is supported only when `zoneBalance` is `false`.

#### Minimum instance percentage

The minimum value depends on the number of zones eligible for placement after Azure applies `includeZones`, `excludeZones`, and `maxZoneCount`:

`minimum instance percentage = ceil(100 / number of eligible zones)`

For example, suppose `includeZones` contains zones `1`, `2`, and `4`, and `maxZoneCount` is `2`. The effective number of eligible zones is two, so the minimum valid percentage is 50.

#### Maximum number of instances in one zone

Azure calculates the maximum number of instances allowed in one zone based on the scale set's target capacity:

`maximum instances per zone = ceil(maximum instance percentage * target capacity / 100)`

Azure rounds the result up to the nearest whole number. For example, if the target capacity is 101 instances and the maximum instance percentage is 85, the scale set can place up to 86 instances in a single zone.

### [REST API](#tab/rest-api-instance-percent)

The following example allows up to 85 percent of instances in one zone:

```http
PUT or PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2026-03-01
```

```json
{
  "placement": {
    "zonePlacementPolicy": "auto"
  },
  "properties": {
    "resiliencyPolicy": {
      "zoneAllocationPolicy": {
        "maxInstancePercentPerZonePolicy": {
          "enabled": true,
          "value": 85
        }
      }
    }
  }
}
```

### [Azure CLI](#tab/cli-instance-percent)

Use `instance-percent-policy` and `max-instance-percent` to allow up to 85 percent of instances in one zone.

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 3 \
  --zone-placement-policy Auto \
  --instance-percent-policy true \
  --max-instance-percent 85
```

### [Azure PowerShell](#tab/powershell-instance-percent)

Use `EnableMaxInstancePercentPerZone` and `MaxInstancePercentPerZoneValue` to allow up to 85 percent of instances in one zone. Configure the operating system, storage, and network profiles before you deploy the scale set.

```azurepowershell
$resourceGroupName = "myResourceGroup"
$scaleSetName = "myScaleSet"
$location = "EastUS"

New-AzResourceGroup -Name $resourceGroupName -Location $location

$vmssConfig = New-AzVmssConfig `
  -Location $location `
  -SkuCapacity 3 `
  -SkuName "Standard_D2s_v5" `
  -ZonePlacementPolicy "Auto" `
  -EnableMaxInstancePercentPerZone `
  -MaxInstancePercentPerZoneValue 85

# Configure the operating system, storage, and network profiles before deployment.

New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

---

#### Expected behavior

Azure first attempts to distribute instances evenly across three zones. If an even distribution isn't possible, Azure can allocate up to 85 percent of the instances to one zone. This setting gives Azure more flexibility to satisfy allocation requests but allows a higher concentration of instances in one zone.

After a zone reaches the calculated maximum, Azure attempts to place the remaining instances in other eligible zones. If those zones don't have enough capacity, the remaining instances may fail to provision.

For example, suppose you request 100 instances and available capacity across three zones is 90, 10, and 0. Azure attempts a distribution of 85, 10, and 5. Because the third zone has no capacity, the final distribution is 85, 10, and 0, and five instances fail to provision.

### Use strict zone balancing

Set `zoneBalance` to `true` to require a balanced distribution across the selected zones. You must specify `maxZoneCount` when strict zone balancing is enabled.

### [REST API](#tab/rest-api-zone-balance)

The following example enables strict zone balancing across three zones.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2026-03-01
```

```json
{
  "placement": {
    "zonePlacementPolicy": "auto"
  },
  "properties": {
    "zoneBalance": true,
    "resiliencyPolicy": {
      "zoneAllocationPolicy": {
        "maxZoneCount": 3
      }
    }
  }
}
```

### [Azure CLI](#tab/cli-zone-balance)

Use `zone-balance` with `max-zone-count` to require strict balancing across three zones.

```azurecli
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 3 \
  --zone-placement-policy Auto \
  --zone-balance true \
  --max-zone-count 3
```

### [Azure PowerShell](#tab/powershell-zone-balance)

Use `ZoneBalance` with `MaxZoneCount` to require strict balancing across three zones. Configure the operating system, storage, and network profiles before you deploy the scale set.

```azurepowershell
$resourceGroupName = "myResourceGroup"
$scaleSetName = "myScaleSet"
$location = "EastUS"

New-AzResourceGroup -Name $resourceGroupName -Location $location

$vmssConfig = New-AzVmssConfig `
  -Location $location `
  -SkuCapacity 3 `
  -SkuName "Standard_D2s_v5" `
  -ZonePlacementPolicy "Auto" `
  -ZoneBalance `
  -MaxZoneCount 3

# Configure the operating system, storage, and network profiles before deployment.

New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

---

#### Expected behavior

Azure requires the instance counts across the selected zones to [remain within one instance of each other](virtual-machine-scale-sets-zone-balancing.md#balanced-and-unbalanced-scale-sets). When you set `maxZoneCount` to `3`, Azure selects up to three zones and maintains strict balance across the zones in use.

If Azure can't maintain strict balance because of capacity or zone availability, the instances that can't be placed within the policy fail to provision. Strict zone balancing can therefore reduce allocation success compared to best-effort balancing.

### Enable automatic zone placement on an existing scale set

You can enable automatic zone placement on an existing scale set that uses customer-selected zones or regional placement.

> [!WARNING]
> After you enable automatic zone placement on a regional scale set, you can't return the scale set to regional placement. Existing regional instances remain regional, but Azure places new instances in availability zones.

Before you enable automatic zone placement on a regional scale set, review [Prepare for zone expansion](virtual-machine-scale-sets-configure-customer-selected-zones.md#prepare-for-zone-expansion).

For a scale set that uses customer-selected zones, clear the `zones` property and set `placement.zonePlacementPolicy` to `auto` in the same update operation. For a regional scale set, set `placement.zonePlacementPolicy` to `auto`.

### [REST API](#tab/rest-api-migrate)

Send a `PATCH` request to the following endpoint:

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2026-03-01
```

For a scale set that uses customer-selected zones, clear `zones` and enable automatic zone placement in the same request:

```json
{
  "zones": [],
  "placement": {
    "zonePlacementPolicy": "auto"
  }
}
```

For a regional scale set, enable automatic zone placement without specifying `zones`:

```json
{
  "placement": {
    "zonePlacementPolicy": "auto"
  }
}
```

### [Azure CLI](#tab/cli-migrate)

For a scale set that uses customer-selected zones, use one [az vmss update](/cli/azure/vmss#az-vmss-update) command to clear `zones` and enable automatic zone placement:

```azurecli
az vmss update \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --zone-placement-policy Auto \
  --set zones=[]
```

For a regional scale set, enable automatic zone placement without clearing `zones`:

```azurecli
az vmss update \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --zone-placement-policy Auto
```

### [Azure PowerShell](#tab/powershell-migrate)

Retrieve the scale set model, update the placement configuration locally, and submit the changes in one [Update-AzVmss](/powershell/module/az.compute/update-azvmss) operation:

```azurepowershell
$resourceGroupName = "myResourceGroup"
$scaleSetName = "myScaleSet"

$vmss = Get-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -VMScaleSetName $scaleSetName

# For a scale set that uses customer-selected zones, clear the zones property.
# Omit this line for a regional scale set.
$vmss.Zones = [System.Collections.Generic.List[string]]::new()

$vmss.Placement = [Microsoft.Azure.Management.Compute.Models.Placement]::new()
$vmss.Placement.ZonePlacementPolicy = "Auto"

Update-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -VMScaleSetName $scaleSetName `
  -VirtualMachineScaleSet $vmss
```

---

#### Expected behavior

The update enables automatic zone placement for new instances. Existing instances aren't moved or recreated.

For a scale set that uses customer-selected zones, existing instances remain in their current availability zones. For a regional scale set, existing regional instances remain regional. Azure uses automatic zone placement when it places new instances.

If your placement requirements change, you can switch from automatic zone placement back to customer-selected zones. In the same update operation, clear `placement.zonePlacementPolicy` and set the `zones` property. This update doesn't move or recreate existing instances.

## Next steps

- Review [availability zone options for Virtual Machine Scale Sets](virtual-machine-scale-sets-use-availability-zones.md).
- Learn about [zone balancing in Virtual Machine Scale Sets](virtual-machine-scale-sets-zone-balancing.md).
- Learn about [Automatic Zone Balance](auto-zone-balance-overview.md).