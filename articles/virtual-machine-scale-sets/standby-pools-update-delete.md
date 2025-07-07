---
title: Delete or update a standby pool for Virtual Machine Scale Sets
description: Learn how to delete or update a standby pool for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: ju-shim
# Customer intent: As a cloud administrator, I want to update or delete standby pools in Virtual Machine Scale Sets, so that I can manage resource allocation and optimize performance based on operational requirements.
---


# Update or delete a standby pool

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

You can update the state of the instances and the max ready capacity of your standby pool at any time. The standby pool name can only be set during standby pool creation. If updating the provisioning state to hibernated, ensure that the scale set is properly configured to use hibernated VMs. For more information see, [Hibernation overview](../virtual-machines/hibernate-resume.md). 

When changing the provisioning state of your standby pool, transitioning between the following states below are supported. Transitioning between a Stopped (deallocated) state and a hibernated state is not supported. If using a Stopped (deallocated) pool and you want to instead use a hibernated pool, first transition to a running pool then update the provisioning state to hibernated. 

|Initial state | Updated state | 
|---|---|
| Running | Stopped (deallocated) | 
| Running | Hibernated | 
| Stopped (deallocated) | Running | 
| Hibernated | Running | 
| Hibernated | Stopped (deallocated) | 

## Update a standby pool


### [Portal](#tab/portal-2)

> [!NOTE]
> Setting the standby pool VM state to hibernated is not yet available in the Azure portal. To configure a standby pool with a hibernated VM state use an alternative SDK such as CLI or PowerShell.

1) Navigate to Virtual Machine Scale set the standby pool is associated with. 
2) Under **Availability + scale** select **Standby pool**. 
3) Select **Manage pool**. 
4) Update the configuration and save any changes.  

:::image type="content" source="media/standby-pools/managed-standby-pool-after-vmss-create.png" alt-text="A screenshot of the Networking tab in the Azure portal during the Virtual Machine Scale Set creation process.":::


### [CLI](#tab/cli-2)
Update an existing standby pool using [az standby-vm-pool update](/cli/azure/standby-vm-pool).

```azurecli-interactive
az standby-vm-pool update \
   --resource-group myResourceGroup \
   --name myStandbyPool \
   --max-ready-capacity 20 \
   --min-ready-capacity 5 \
   --vm-state "Deallocated" 
```
### [PowerShell](#tab/powershell-2)
Update an existing standby pool using [Update-AzStandbyVMPool](/powershell/module/az.standbypool/update-azstandbyvmpool).

```azurepowershell-interactive
Update-AzStandbyVMPool `
   -ResourceGroup myResourceGroup `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -MinReadyCapacity 5 `
   -VMState "Deallocated" 
```

### [ARM template](#tab/template)
Update an existing standby pool deployment. Deploy the updated template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```JSON
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
           "defaultValue": 10
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


### [Bicep](#tab/bicep-2)
Update an existing standby pool deployment. Deploy the updated template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param location string = resourceGroup().location
param standbyPoolName string = 'myStandbyPool'
param maxReadyCapacity int = 10
param minReadyCapacity int = 5
@allowed([
  'Running'
  'Deallocated'
  'Hibernated'
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

### [REST](#tab/rest-2)
Update an existing standby pool using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update).

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


## Delete a standby pool

### [Portal](#tab/portal-3)

1) Navigate to Virtual Machine Scale set the standby pool is associated with. 
2) Under **Availability + scale** select **Standby pool**. 
3) Select **Delete pool**. 
4) Select **Delete**. 

:::image type="content" source="media/standby-pools/delete-standby-pool-portal.png" alt-text="A screenshot showing how to delete a standby pool in the portal.":::


### [CLI](#tab/cli-3)
Delete an existing standby pool using [az standbypool delete](/cli/azure/standby-vm-pool).

```azurecli-interactive
az standby-vm-pool delete \
    --resource-group myResourceGroup \
    --name myStandbyPool
```
### [PowerShell](#tab/powershell-3)
Delete an existing standby pool using [Remove-AzStandbyVMPool](/powershell/module/az.standbypool/remove-azstandbyvmpool).

```azurepowershell-interactive
Remove-AzStandbyVMPool `
    -ResourceGroup myResourceGroup `
    -Name myStandbyPool `
    -Nowait
```

### [REST](#tab/rest-3)
Delete an existing standby pool using [Delete](/rest/api/standbypool/standby-virtual-machine-pools/delete).

```HTTP
DELETE https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2025-03-01
```

---

## Next steps
Review the most [frequently asked questions](standby-pools-faq.md) about standby pools for Virtual Machine Scale Sets.
