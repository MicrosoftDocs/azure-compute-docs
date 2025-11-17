---
title: View instance mix configurations
description: How to view instance mix configurations on a scale set. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 08/19/2025
ms.reviewer: jushiman
# Customer intent: As a cloud administrator, I want to view the instance mix configurations of a virtual machine scale set, so that I can assess VM sizes and allocation strategies for effective resource management.
---

# View instance mix configurations

This article details how to view your instance mix configuration on a virtual machine scale set, including the virtual machine (VM) sizes and the allocation strategy.

## Prerequisites
- Reader, or higher, role on the scale set resource, or the `Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read` permission.
- A scale set using Flexible Orchestration Mode and instance mix.
- For REST deployments, use API Version `2024-11-01` or later.

## View the instance mix configurations
### [Azure portal](#tab/portal-1)
1. Go to the virtual machine scale set you want to view.
2. In the **Overview** blade, locate the **Properties** section.
3. Under **Size**, view the VM sizes available in the scale set.
4. Under **Management**, view the **Allocation strategy**.

### [Azure CLI](#tab/cli-1)
Replace placeholders, such as `<scaleSetName>`, with your actual values.

#### View all instance mix properties
To display all properties in the `skuProfile`, run:
```azurecli-interactive
az vmss show --resource-group <resourceGroupName> --name <scaleSetName> --query "skuProfile"
```

**Example output:**
```json
{
  "allocationStrategy": "CapacityOptimized",
  "vmSizes": [
    {
      "name": "Standard_D4s_v5"
    },
    {
      "name": "Standard_D4as_v5"
    }
  ]
}
```

#### View VM sizes
To display only the VM sizes, run:
```azurecli-interactive
az vmss show --resource-group <resourceGroupName> --name <scaleSetName> --query "skuProfile.vmSizes"
```

#### View allocation strategy
To display only the allocation strategy, run:
```azurecli-interactive
az vmss show --resource-group <resourceGroupName> --name <scaleSetName> --query "skuProfile.allocationStrategy"
```

### [PowerShell](#tab/powershell-1)
#### View all instance mix properties
```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "<resourceGroupName>" -VMScaleSetName "<scaleSetName>" | Select-Object -ExpandProperty skuProfile
```

#### View VM sizes
```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "<resourceGroupName>" -VMScaleSetName "<scaleSetName>" | Select-Object -ExpandProperty skuProfile | Select-Object -ExpandProperty vmSizes
```

#### View allocation strategy
```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "<resourceGroupName>" -VMScaleSetName "<scaleSetName>" | Select-Object -ExpandProperty skuProfile | Select-Object -ExpandProperty AllocationStrategy
```

---

## Next steps
Learn how to [update](instance-mix-update.md) your instance mix enabled scale set.