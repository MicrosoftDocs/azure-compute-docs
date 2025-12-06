---
title: Request a container from a Standby pool for Azure Container Instances
description: Learn how to scale out from a standby pool with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/17/2025
ms.reviewer: tomvcassidy
# Customer intent: As a DevOps engineer, I want to request a container group from a standby pool in Azure Container Instances, so that I can effectively manage container deployments and optimize resource usage during scaling.
---


# Request a container from a standby pool for Azure Container Instances

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](container-instances-standby-pool-configure-permissions.md)**.


This article steps through requesting a container group from a standby pool for Azure Container Instances.   

## Prerequisites

Before utilizing standby pools, complete the feature registration and configure role based access controls. For more information see [Configure role permissions for standby pools in Azure Container Instances](container-instances-standby-pool-configure-permissions.md)


## Request a container from the standby pool

### [CLI](#tab/cli)
Request a container group from a standby pool using [az container create](/cli/azure/container) and specifying the standby pool and container group profile. For more information on using config maps during container requests, see [use config maps](container-instances-config-map.md). 

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \ 
    --location WestCentralUS \
    --config-map key1=value1 key2=value2 \
    --container-group-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile" \
    --container-group-profile-revision 1 \
    --standby-pool-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool" 


```
### [PowerShell](#tab/powershell)
Request a container group from a standby pool using [New-AzContainerGroup](/powershell/module/az.containerinstance/new-AzContainerGroup) and specifying the standby pool and container group profile. For more information on using config maps during container requests, see [use config maps](container-instances-config-map.md). 

```azurepowershell-interactive
$container = New-AzContainerInstancenoDefaultObject -Name mycontainer -ConfigMapKeyValuePair @{"key1"="value1"}

New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name mycontainergroup`
    -Container $container `
    -Location "WestCentralUS" `
    -ContainerGroupProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile" `
    -ContainerGroupProfileRevision 1 `
    -StandbyPoolProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool" 

```

### [ARM template](#tab/template)
Request a container group from a standby pool using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) and specifying the standby pool and container group profile. For more information on using config maps during container requests, see [use config maps](container-instances-config-map.md). 


```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "subscriptionId": {
      "type": "string",
      "metadata": {
        "description": "The subscription ID."
      }
    },
    "resourceGroup": {
      "type": "string",
      "metadata": {
        "description": "The name of the resource group."
      }
    },
    "location": {
      "type": "string",
      "metadata": {
        "description": "Location for the resource."
      },
      "defaultValue": "West Central US"
    },
    "containerGroupName": {
      "type": "string",
      "metadata": {
        "description": "The name of the container group."
      }
    },
    "standbyPoolName": {
      "type": "string",
      "metadata": {
        "description": "The name of the standby pool."
      }
    },
    "containerGroupProfileName": {
      "type": "string",
      "metadata": {
        "description": "The name of the container group profile."
      }
    },
    "myContainerProfile": {
      "type": "string",
      "metadata": {
        "description": "The name of the container profile."
      }
    },
    "newKey": {
      "type": "string",
      "metadata": {
        "description": "The new key for the config map."
      }
    },
    "newValue": {
      "type": "string",
      "metadata": {
        "description": "The new value for the config map."
      }
    },
    "revisionNumber": {
      "type": "int",
      "metadata": {
        "description": "The revision number for the container group profile."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2024-05-01-preview",
      "name": "[parameters('containerGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "standByPoolProfile": {
          "id": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.StandbyPool/standbyContainerGroupPools/', parameters('standbyPoolName'))]"
        },
        "containerGroupProfile": {
          "id": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.ContainerInstance/mycontainergroupprofile/', parameters('containerGroupProfileName'))]",
          "revision": "[parameters('revisionNumber')]"
        },
        "containers": [
          {
            "name": "[parameters('mycontainergroupprofile')]",
            "properties": {
              "configMap": {
                "keyValuePairs": {
                  "[parameters('newKey')]": "[parameters('newValue')]"
                }
              }
            }
          }
        ]
      }
    }
  ],
  "outputs": {
    "containerGroupId": {
      "type": "string",
      "value": "[resourceId('Microsoft.ContainerInstance/containerGroups', parameters('containerGroupName'))]"
    }
  }
}

```


### [REST](#tab/rest)
Request a container group from a standby pool using [Create or Update](/rest/api/container-instances/container-groups/create-or-update) and specifying the standby pool and container group profile. For more information on using config maps during container requests, see [use config maps](container-instances-config-map.md). 

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2024-05-01-preview 

Request Body
{
   "location": "{location}",
   "properties": {
       "standByPoolProfile":{
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool"
           },
           "containerGroupProfile": {
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile",
               "revision": {revisionNumber}
           },
          "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
              "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identity}": {}
            },
           "containers": [
               {
                   "name": "{mycontainergroupprofile}",
                   "properties": {
                       "configMap": {
                           "keyValuePairs": {
                               "{newKey}": "{newValue}"
                           }
                       }
                   }
               }
           ]
       }
    }    
}
```

---



## Next steps

- [Get standby pool and container details using the standby pool runtime view APIs](container-instances-standby-pool-get-details.md).
- [Update or delete your standby pool for Azure Container Instances](container-instances-standby-pool-update-delete.md).
