---
title: Update a Virtual Machine Scale Set with instance mix
description: How to update a virtual machine scale set instance mix settings. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 1/10/2025
ms.reviewer: jushiman
---

# Update instance mix settings on an existing scale set
The article walks through how to update the instance mix settings on a scale set.

> [!IMPORTANT]
> Instance mix for Virtual Machine Scale Sets with Flexible Orchestration Mode is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change before general availability (GA). 

## Prerequisites
Before using instance mix, complete feature registration for the `FlexVMScaleSetSkuProfileEnabled` feature flag using the [az feature register](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

It takes a few moments for the feature to register. Verify the registration status by using the [az feature show](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

## Update the instance mix settings on an existing scale set
The instance mix settings can be updated on your scale set via CLI, PowerShell, and REST API. You can change either the virtual machine (VM) sizes or the allocation strategy, or both, in a single call.

When changing allocation strategies, the *new* allocation strategy won't become effective until the scale set scales in or out. That is to say, your existing VMs won't be altered based on the allocation strategy until there's a scaling action.

When changing from `Prioritized` to another allocation strategy, you must first nullify the priority ranks associated with the VM sizes. This will be covered in more detail in the supporting code snippets. 

### [Azure CLI](#tab/cli-1)
Before using CLI commands with instance mix, be sure you're using the correct CLI version. Make sure you're using version `2.66.0` or greater.

#### Change the Allocation Strategy
You can use the following basic command to update the allocation strategy. In this case, we're updating the scale set to use the `CapacityOptimized` allocation strategy:

```azurecli-interactive
az vmss update \
    --resource-group {resourceGroupName} \
    --name {scaleSetName} \
    --set skuProfile.allocationStrategy=CapacityOptimized
```
#### Change the VM Sizes
You can use the following command to update the VM sizes specified in the `skuProfile`. In this scenario, we're updating the VM sizes to be Standard D2asv4, Standard D2asv5, and Standard D2sv5:

```azurecli-interactive
az vmss update \
    --resource-group {resourceGroupName} \
    --name {scaleSetName} \
    --skuprofile-vmsizes Standard_D2as_v4 Standard_D2as_v5 Standard_D2s_v5
```

### [Azure PowerShell](#tab/powershell-1)

#### Change the Allocation Strategy
You can use the following basic command to update the allocation strategy:
 
```azurepowershell-interactive
# Set variable values
$resourceGroupName = "resourceGroupName"
$vmssName = "scaleSetName"
$allocationStrategy = "CapacityOptimized";

# Get the scale set information
$vmss = Get-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName;

# Update the allocation strategy
$vmss.SkuProfile.AllocationStrategy = $allocationStrategy

#Update the scale set
Update-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName `
-VirtualMachineScaleSet $vmss
```

#### Change the VM Sizes
To change the VM sizes specified in your scale set, you can use the following PowerShell command. In this example, we'll be updating the scale set to use Standard D2asv4, Standard D2asv5, and Standard D2sv5.

```azurepowershell-interactive
# Set variable values
$resourceGroupName = "resourceGroupName" `
$vmssName = "scaleSetName";

# Create a variable to hold the new VM Sizes values
$vmSizeList = [System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.SkuProfileVMSize]]::new() 

# Add the VM sizes to the list
$vmSizeList.Add("Standard_D2as_v5") `
$vmSizeList.Add("Standard_D2s_v5") `
$vmSizeList.Add("Standard_D2as_v4") ;

# Get the scale set information
$vmss = Get-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName

# Update the VM sizes in the scale set
$vmss.SkuProfile.vmSizes = $vmSizeList

#Update the scale set
Update-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName `
-VirtualMachineScaleSet $vmss
```

### [REST API](#tab/arm-1)
To update the instance mix settings through REST API, use a `PATCH` call to the scale set resource. Be sure to use an API version on or after `2023-09-01`:
```json
PUT https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{youScaleSetName}?api-version=2023-09-01
```

The following subsections walk through what to use if you want to change the allocation strategy or VM sizes through REST APIs.

#### Change the Allocation Strategy
You must specify both the VM sizes you'd like to use, and the allocation strategy. In this example, we're changing the allocation strategy to `capactiyOptimized`:
```json
{
	"properties": {
		"skuProfile": {
                "vmSizes": [
                {
                    "name": "Standard_D2as_v4"
                },
                {
                    "name": "Standard_D2as_v5"
                },
                {
                    "name": "Standard_D2s_v4"
                },
                {
                    "name": "Standard_D2s_v3"
                },
                {
                    "name": "Standard_D2s_v5"
                }
			],
		    "allocationStrategy": "capacityOptimized"
		}
	}
}
```

#### Change the VM Sizes
To change the VM Sizes in your deployment, you only need to change the VM Sizes in the `skuProfile`. In this example, we're changing the VM sizes specified in the scale set to use D2sv5, D2asv5, D2sv4, D2asv4, and D2sv3:
```json
{
	"properties": {
		"skuProfile": {
      		"vmSizes": [
                {
                    "name": "Standard_D2as_v4"
                },
                {
                    "name": "Standard_D2as_v5"
                },
                {
                    "name": "Standard_D2s_v4"
                },
                {
                    "name": "Standard_D2s_v3"
                },
                {
                    "name": "Standard_D2s_v5"
                }
			]
		}
    }
}
```
---

## Update an existing scale set to use instance mix
Existing scale sets that don't have instance mix can enable instance mix by specifying the `skuProfile` properties in the scale set. The `skuProfile`, `vmSizes`, and `allocationStrategy` can be specified through REST API and CLI. 

The properties that must be updated are:
* `sku.name` must be set to `"Mix"`.
* `sku.tier` must be set to `null`.
* You must define the `skuProfile` properties. At least one value must be provided in `vmSizes`. An `allocationStrategy` should be set, but if a value isn't provided, Azure defaults to `lowestPrice`.

The following sections have sample code snippets to demonstrate enabling instance mix on existing scale sets. 

### [Azure CLI](#tab/cli-2)
In this snippet, we'll update an existing scale set using Flexible Orchestration Mode to use instance mix with the VM sizes D2asv4, D2sv5, and D2asv5 and allocation strategy of `capacityOptimized`.

```azurecli-interactive
az vmss update \
--name {scaleSetName} \
--resource-group {resourceGroupName} \
--set sku.name=Mix sku.tier=null \
--skuprofile-vmsizes Standard_D2as_v4 Standard_D2s_v5 Standard_D2as_v5 \
--sku-allocat-strat capacityOptimized
```

### [REST API](#tab/arm-2)
To update the instance mix settings through REST API, use a `PATCH` call to the scale set resource. Be sure to use an API version on or after `2023-09-01`. 
```json
PUT https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{youScaleSetName}?api-version=2023-09-01
```

In the body, be sure to set `sku.name` to `"Mix"` and include the `skuProfile` with your inputs for `vmSizes` and `allocationStrategy`:
```json
{
	"sku": {
		"name": "Mix"
	},
	"properties": {
		"skuProfile": {
      		"vmSizes": [
        	{
          		"name": "Standard_D2as_v4"
        	},
        	{
          		"name": "Standard_D2as_v5"
        	},
        	{
				"name": "Standard_D2s_v4"
        	},
        	{
        	    "name": "Standard_D2s_v5"
        	}
			],
			"allocationStrategy": "lowestPrice"
		},
    }
}

```
---

## Next steps
Learn how to [troubleshoot](instance-mix-faq-troubleshooting.md) your instance mix enabled scale set.