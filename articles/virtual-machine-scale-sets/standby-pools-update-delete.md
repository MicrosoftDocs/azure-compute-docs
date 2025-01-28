---
title: Delete or update a standby pool for Virtual Machine Scale Sets
description: Learn how to delete or update a standby pool for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/5/2024
ms.reviewer: ju-shim
---


# Update or delete a standby pool
This article covers updating, managing and deleting a standby pool resource.  

## Prerequisites

To allow standby pools to create and manage virtual machines in your subscription, assign the appropriate permissions to the standby pool resource provider. 

1) In the Azure portal, navigate to your subscriptions.
2) Select the subscription you want to adjust permissions.
3) Select **Access Control (IAM)**.
4) Select **Add** and **Add role assignment**.
5) Under the **Role** tab, search for **Virtual Machine Contributor** and select it.
6) Move to the **Members** Tab.
7) Select **+ Select members**.
8) Search for **Standby Pool Resource Provider** and select it.
9) Move to the **Review + assign** tab.
10) Apply the changes. 
11) Repeat the above steps and assign the **Network Contributor** role and the **Managed Identity Operator** role to the Standby Pool Resource Provider. If you're using images stored in Compute Gallery assign the **Compute Gallery Sharing Admin** and **Compute Gallery Artifacts Publisher** roles as well.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).


## Update a standby pool

You can update the state of the instances and the max ready capacity of your standby pool at any time. The standby pool name can only be set during standby pool creation. 

### [Portal](#tab/portal-2)

> [!NOTE]
> To create and manage standby pools in the Azure portal, register the following feature flag:
> `Register-AzProviderFeature -FeatureName StandbyVMPoolPreview -ProviderNamespace Microsoft.StandbyPool`

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
            "apiVersion": "2024-03-01",
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
])
param vmState string = 'Deallocated'
param virtualMachineScaleSetId string = '/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet'

resource standbyPool 'Microsoft.standbypool/standbyvirtualmachinepools@2024-03-01' = {
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
PUT https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2024-03-01
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

> [!NOTE]
> To create and manage standby pools in the Azure portal, register the following feature flag:
> `Register-AzProviderFeature -FeatureName StandbyVMPoolPreview -ProviderNamespace Microsoft.StandbyPool`

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
DELETE https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2024-03-01
```

---

## Next steps
Review the most [frequently asked questions](standby-pools-faq.md) about standby pools for Virtual Machine Scale Sets.
