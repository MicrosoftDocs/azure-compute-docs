---
title: Update a Virtual Machine Scale Set with instance mix
description: How to update a virtual machine scale set instance mix settings. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 06/10/2025
ms.reviewer: jushiman
# Customer intent: As a cloud system administrator, I want to update the instance mix settings on a virtual machine scale set, so that I can optimize VM sizes and allocation strategies to better meet my application's performance and cost requirements.
---

# Update instance mix settings on an existing scale set
This article explains how to update the instance mix settings on a scale set, including changing VM sizes and allocation strategies.

## Update the instance mix settings on an existing scale set
The instance mix settings can be updated on your scale set via CLI, PowerShell, and REST API. You can change either the virtual machine (VM) sizes or the allocation strategy, or both, in a single call.

> [!NOTE]
> When you change the allocation strategy, the new strategy takes effect only after the scale set scales in or out. Existing VMs are not affected until a scaling action occurs.

When changing from `Prioritized (preview)` to another allocation strategy, you must first nullify the priority ranks associated with the VM sizes. This will be covered in more detail in the supporting code snippets. 

### [Azure CLI](#tab/cli-1)
Ensure you are using Azure CLI version `2.66.0` or later.

#### Change the allocation strategy
To update the allocation strategy, for example, to `CapacityOptimized`:

```azurecli-interactive
az vmss update \
  --resource-group {resourceGroupName} \
  --name {scaleSetName} \
  --set skuProfile.allocationStrategy=CapacityOptimized
```

#### Change the VM sizes
To update the VM sizes in the `skuProfile`, for example, to Standard_D2as_v4, Standard_D2as_v5, and Standard_D2s_v5:

```azurecli-interactive
az vmss update \
  --resource-group {resourceGroupName} \
  --name {scaleSetName} \
  --skuprofile-vmsizes Standard_D2as_v4 Standard_D2as_v5 Standard_D2s_v5
```

### [Azure PowerShell](#tab/powershell-1)

#### Change the allocation strategy
To update the allocation strategy:

```azurepowershell-interactive
# Set variable values
$resourceGroupName = "resourceGroupName"
$vmssName = "scaleSetName"
$allocationStrategy = "CapacityOptimized"

# Get the scale set
$vmss = Get-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $vmssName

# Update the allocation strategy
$vmss.SkuProfile.AllocationStrategy = $allocationStrategy

# Apply the update
Update-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $vmssName -VirtualMachineScaleSet $vmss
```

#### Change the VM sizes
To update the VM sizes:

```azurepowershell-interactive
# Set variable values
$resourceGroupName = "resourceGroupName"
$vmssName = "scaleSetName"

# Create a list of new VM sizes
$vmSizeList = [System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.SkuProfileVMSize]]::new()
$vmSizeList.Add("Standard_D2as_v4")
$vmSizeList.Add("Standard_D2as_v5")
$vmSizeList.Add("Standard_D2s_v5")

# Get the scale set
$vmss = Get-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $vmssName

# Update the VM sizes
$vmss.SkuProfile.vmSizes = $vmSizeList

# Apply the update
Update-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $vmssName -VirtualMachineScaleSet $vmss
```

### [REST API](#tab/arm-1)
To update instance mix settings using the REST API, send a `PATCH` request to the scale set resource. Use API version `2023-09-01` or later:

```http
PATCH https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{yourScaleSetName}?api-version=2023-09-01
```

#### Change the allocation strategy
Specify both the VM sizes and the allocation strategy. For example, to set the allocation strategy to `capacityOptimized`:

```json
{
  "properties": {
    "skuProfile": {
      "vmSizes": [
        { "name": "Standard_D2as_v4" },
        { "name": "Standard_D2as_v5" },
        { "name": "Standard_D2s_v4" },
        { "name": "Standard_D2s_v3" },
        { "name": "Standard_D2s_v5" }
      ],
      "allocationStrategy": "capacityOptimized"
    }
  }
}
```

#### Change the VM sizes
To update only the VM sizes:

```json
{
  "properties": {
    "skuProfile": {
      "vmSizes": [
        { "name": "Standard_D2as_v4" },
        { "name": "Standard_D2as_v5" },
        { "name": "Standard_D2s_v4" },
        { "name": "Standard_D2s_v3" },
        { "name": "Standard_D2s_v5" }
      ]
    }
  }
}
```

---

## Enable instance mix on an existing scale set
To enable instance mix on a scale set that does not already use it, specify the `skuProfile` properties. You must set:
- `sku.name` to `"Mix"`
- `sku.tier` to `null`
- At least one value in `vmSizes` under `skuProfile`
- An `allocationStrategy` (if not specified, Azure defaults to `lowestPrice`)

The following examples show how to enable instance mix on an existing scale set.

### [Azure CLI](#tab/cli-2)
This example updates an existing scale set in Flexible Orchestration Mode to use instance mix with VM sizes Standard_D2as_v4, Standard_D2s_v5, and Standard_D2as_v5, and the `capacityOptimized` allocation strategy:

```azurecli-interactive
az vmss update \
  --name {scaleSetName} \
  --resource-group {resourceGroupName} \
  --set sku.name=Mix sku.tier=null \
  --skuprofile-vmsizes Standard_D2as_v4 Standard_D2s_v5 Standard_D2as_v5 \
  --sku-allocat-strat capacityOptimized
```

### [REST API](#tab/arm-2)
To enable instance mix using the REST API, send a `PATCH` request to the scale set resource. Use API version `2023-09-01` or later:

```http
PATCH https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{yourScaleSetName}?api-version=2023-09-01
```

In the request body, set `sku.name` to `"Mix"` and include the `skuProfile` with your desired `vmSizes` and `allocationStrategy`:

```json
{
  "sku": {
    "name": "Mix"
  },
  "properties": {
    "skuProfile": {
      "vmSizes": [
        { "name": "Standard_D2as_v4" },
        { "name": "Standard_D2as_v5" },
        { "name": "Standard_D2s_v4" },
        { "name": "Standard_D2s_v5" }
      ],
      "allocationStrategy": "lowestPrice"
    }
  }
}
```

---

## Next steps
Learn how to [troubleshoot](instance-mix-faq-troubleshooting.md) your instance mix-enabled scale set.