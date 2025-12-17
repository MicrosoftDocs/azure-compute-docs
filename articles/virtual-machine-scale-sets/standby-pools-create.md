---
title: Create a standby pool for Virtual Machine Scale Sets
description: Learn how to create a standby pool to reduce scale-out latency with Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: cynthn
# Customer intent: As a cloud infrastructure administrator, I want to create and manage a standby pool for Virtual Machine Scale Sets, so that I can reduce scale-out latency and ensure high availability of resources.
---


# Create a standby pool

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

This article steps through creating a standby pool for Virtual Machine Scale Sets with Flexible Orchestration.

## Create a standby pool

### [Portal](#tab/portal)

> [!NOTE]
> Setting the standby pool VM state to hibernated is not yet available in the Azure portal. To configure a standby pool with a hibernated VM state use an alternative SDK such as CLI or PowerShell.

1) Navigate to your Virtual Machine Scale Set.
2) Under **Availability + scale** select **Standby pool**. 
3) Select **Manage pool**.
4) Provide a name for your pool, select a provisioning state and set the maximum and minimum ready capacity.
5) Select **Save**.

:::image type="content" source="media/standby-pools/enable-standby-pool-after-vmss-creation.png" alt-text="A screenshot showing how to enable a standby pool on an existing Virtual Machine Scale Set in the Azure portal.":::

You can also configure a standby pool during Virtual Machine Scale Set creation by navigating to the **Management** tab and checking the box to enable standby pools. 

:::image type="content" source="media/standby-pools/enable-standby-pool-during-vmss-create.png" alt-text="A screenshot showing how to enable a standby pool during the Virtual Machine Scale Set create experience in the portal.":::


### [CLI](#tab/cli)
Create a standby pool and associate it with an existing scale set using [az standby-vm-pool create](/cli/azure/standby-vm-pool).

```azurecli-interactive
az standby-vm-pool create \
	--resource-group myResourceGroup \
	--location eastus --name myStandbyPool \
	--max-ready-capacity 20 \
	--min-ready-capacity 5 \
	--vm-state "Deallocated" \
	--vmss-id "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
```
### [PowerShell](#tab/powershell)
Create a standby pool and associate it with an existing scale set using [New-AzStandbyVMPool](/powershell/module/az.standbypool/new-azstandbyvmpool).

```azurepowershell-interactive
New-AzStandbyVMPool `
   -ResourceGroup myResourceGroup `
   -Location eastus `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -MinReadyCapacity 5 `
   -VMState "Deallocated" `
   -VMSSId "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
```

### [ARM template](#tab/template)
Create a standby pool and associate it with an existing scale set. Create a template and deploy it using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
           "type": "string",
           "defaultValue": "east us"    
        },
        "name": {
           "type": "string",
           "defaultValue": "myStandbyPool"
        },
        "maxReadyCapacity" : {
           "type": "int",
           "defaultValue": 20
        },
        "minReadyCapacity" : {
           "type": "int",
           "defaultValue": 5
        },
        "virtualMachineState" : {
           "type": "string",
           "defaultValue": "Deallocated"
        },
        "attachedVirtualMachineScaleSetId" : {
           "type": "string",
           "defaultValue": "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
        }
    },
    "resources": [ 
        {
            "type": "Microsoft.StandbyPool/standbyVirtualMachinePools",
            "apiVersion": "2025-03-01",
            "name": "[parameters('name')]",
            "location": "[parameters('location')]",
            "properties": {
               "elasticityProfile": {
                   "maxReadyCapacity": "[parameters('maxReadyCapacity')]",
                   "minReadyCapacity": "[parameters('minReadyCapacity')]" 
               },
               "virtualMachineState": "[parameters('virtualMachineState')]",
               "attachedVirtualMachineScaleSetId": "[parameters('attachedVirtualMachineScaleSetId')]"
            }
        }
    ]
   }

```


### [Bicep](#tab/bicep)
Create a standby pool and associate it with an existing scale set. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param location string = resourceGroup().location
param standbyPoolName string = 'myStandbyPool'
param maxReadyCapacity int = 20
param minReadyCapacity int = 5
@allowed([
  'Running'
  'Deallocated'
])
param vmState string = 'Deallocated'
param virtualMachineScaleSetId string = '/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet'

resource standbyPool 'Microsoft.standbypool/standbyvirtualmachinepools@2025-03-01' = {
  name: standbyPoolName
  location: location
  properties: {
     elasticityProfile: {
      maxReadyCapacity: maxReadyCapacity
      minReadyCapacity: minReadyCapacity
    }
    virtualMachineState: vmState
    attachedVirtualMachineScaleSetId: virtualMachineScaleSetId
  }
}
```

### [REST](#tab/rest)
Create a standby pool and associate it with an existing scale set using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update).

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2025-03-01
{
"type": "Microsoft.StandbyPool/standbyVirtualMachinePools",
"name": "myStandbyPool",
"location": "east us",
"properties": {
	 "elasticityProfile": {
		 "maxReadyCapacity": 20
          "minReadyCapacity": 5
	 },
	  "virtualMachineState":"Deallocated",
	  "attachedVirtualMachineScaleSetId": "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
	  }
}
```

---

## Next steps

Learn how to [update and delete a standby pool](standby-pools-update-delete.md).
