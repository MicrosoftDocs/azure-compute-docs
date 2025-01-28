---
title: Update or delete a standby pool for Azure Container Instances (Preview)
description: Learn how to update or delete a standby pool for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/6/2024
ms.reviewer: tomvcassidy
---


# Update or delete a standby pool for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

This article steps through updating or deleting a standby pool for Azure Container Instances. 

## Prerequisites

Before utilizing standby pools, complete the feature registration and configure role based access controls listed in the [Standby pools for Azure Container Instances](container-instances-standby-pool-overview.md#prerequisites) overview page. 


## Update a standby pool
A standby pool can be updated at any point in time. The settings that are adjustable after creation include `maxReadyCapacity` and the associated `containerGroupProfile`. If you update the container group profile of the standby pool, the new profile must also be in the same subscription and location as the standby pool. Once the profile has been updated, the pool will drain all existing instances and replaced them with new ones. 

### [CLI](#tab/cli)
Update an existing standby pool using [az standby-container-group-pool update](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az standby-container-group-pool update \
   --resource-group myResourceGroup 
   --location WestCentralUS \
   --name myStandbyPool \
   --max-ready-capacity 20 \
   --refill-policy always \
   --container-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```
### [PowerShell](#tab/powershell)
Update an existing standby pool using [Update-AzStandbyContainerGroupPool](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
Update-AzStandbyContainerGroupPool `
   -ResourceGroup myResourceGroup `
   -Location WestCentralUS `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -RefillPolicy always `
   -ContainerProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```

### [ARM template](#tab/template)
Update an existing a standby pool. Update your template and deploy it using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
           "type": "string",
           "defaultValue": "West Central US"    
        },
        "name": {
           "type": "string",
           "defaultValue": "myStandbyPool"
        },
        "maxReadyCapacity" : {
           "type": "int",
           "defaultValue": 10
        },
        "refillPolicy" : {
           "type": "string",
           "defaultValue": "always"
        },
        "containerGroupProfile" : {
           "type": "string",
           "defaultValue": "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
        }
    },
    "resources": [ 
        {
            "type": "Microsoft.StandbyPool/standbyContainerGroupPools",
            "apiVersion": "2024-03-01",
            "name": "[parameters('name')]",
            "location": "[parameters('location')]",
            "properties": {
               "elasticityProfile": {
                   "maxReadyCapacity": "[parameters('maxReadyCapacity')]",
                   "refillPolicy": "[parameters('refillPolicy')]"
               },
               "containerGroupProfile": "[parameters('containerGroupProfile')]"
            }
        }
    ]
   }

```


### [REST](#tab/rest)
Update an existing standby pool using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update).

```HTTP
PUT https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool?api-version=2024-03-01 
 
Request Body
{
    "properties": {
        "elasticityProfile": {
            "maxReadyCapacity": 20,
            "refillPolicy": "always"
        },
        "containerGroupProperties": {
            "containerGroupProfile": {
                "id": "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile",
                "revision": 1
            }
        }
    },
    "location": "West Central US"
}
```

---

## Delete a standby pool

### [CLI](#tab/cli-1)
Delete an existing standby pool using [az standby-container-group-pool delete](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az standby-container-group-pool delete \
   --resource-group myResourceGroup \
   --name myStandbyPool 
```
### [PowerShell](#tab/powershell-1)
Delete an existing standby pool using [Update-AzStandbyContainerGroupPool](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
Remove-AzStandbyContainerGroupPool `
   -ResourceGroup myResourceGroup `
   -Name myStandbyPool 
```


### [REST](#tab/rest-1)
Delete an existing standby pool using [Delete](/rest/api/standbypool/standby-virtual-machine-pools/delete).

```HTTP
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyContainerGroupPoolName}?api-version=2024-03-01
```

---


## Next steps

[Get standby pool and container details using the standby pool runtime view APIs](container-instances-standby-pool-get-details.md).
