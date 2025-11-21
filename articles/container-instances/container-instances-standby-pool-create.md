---
title: Create a standby pool for Azure Container Instances
description: Learn how to create a standby pool to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/17/2025
ms.reviewer: tomvcassidy
# Customer intent: As a cloud administrator, I want to create a standby pool for Azure Container Instances, so that I can reduce scale-out latency and improve the responsiveness of my container deployment.
---


# Create a standby pool for Azure Container Instances

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](container-instances-standby-pool-configure-permissions.md)**.


This article steps through creating a container group profile and using that profile to configure a standby pool for Azure Container Instances. 

## Prerequisites

Before utilizing standby pools, complete the feature registration and configure role based access controls. For more information see [Configure role permissions for standby pools in Azure Container Instances](container-instances-standby-pool-configure-permissions.md)

## Create a container group profile
The container group profile tells the standby pool how to configure the containers in the pool. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.

To use [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` when creating your container group profile.

### [CLI](#tab/cli)
Create a container group profile using [az container container-group-profile create](/cli/azure/container). You can optionally include config map details in the container group profile. For more information on config maps, see [use config maps](container-instances-config-map.md).

```azurecli-interactive
az container container-group-profile create \
    --resource-group myResourceGroup \
    --name mycontainergroupprofile \
    --location WestCentralUS \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --os-type Linux \ 
    --ip-address Public \ 
    --ports 8000 \ 
    --cpu 1 \
    --memory 1.5 \
    --restart-policy Never

```
### [PowerShell](#tab/powershell)
Create a container group profile using [New-AzContainerInstanceContainerGroupProfile](/powershell/module/az.containerinstance). You can optionally include config map details in the container group profile. For more information on config maps, see [use config maps](container-instances-config-map.md).


```azurepowershell-interactive
$port1 = New-AzContainerInstancePortObject -Port 8000 -Protocol TCP
$port2 = New-AzContainerInstancePortObject -Port 8001 -Protocol TCP

$container = New-AzContainerInstanceObject -Name mycontainer -Image mcr.microsoft.com/azuredocs/aci-helloworld -RequestCpu 1 -RequestMemoryInGb 1.5 -Port @($port1, $port2)

New-AzContainerInstanceContainerGroupProfile `
    -ResourceGroupName myResourceGroup `
    -Name mycontainergroupprofile `
    -Location WestCentralUS `
    -Container $container `
    -OsType Linux `
    -RestartPolicy "Never" `
    -IpAddressType Public

```

### [ARM template](#tab/template)
Create a container group profile and save the template file. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). You can optionally include config map details in the container group profile. For more information on config maps, see [use config maps](container-instances-config-map.md).



```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroupProfiles",
      "apiVersion": "2024-05-01-preview",
      "name": "[parameters('profileName')]",
      "location": "[parameters('location')]",
      "properties": {
        "containers": [
          {
            "name": "mycontainergroupprofile",
            "properties": {
              "image": "[parameters('containerImage')]",
              "ports": [
                {
                  "port": 8000
                }
              ],
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGB": 1.5
                }
              },
              "command": [],
              "environmentVariables": []
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "TCP",
              "port": 8000
            }
          ]
        },
        "imageRegistryCredentials": [],
        "sku": "Standard"
      }
    }
  ],
  "parameters": {
    "profileName": {
      "type": "string",
      "defaultValue": "mycontainergroupprofile",
      "metadata": {
        "description": "Name of the container profile"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "West Central US",
      "metadata": {
        "description": "Location for the resource"
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
      "metadata": {
        "description": "The container image to use"
      }
    }
  }
}


```


### [REST](#tab/rest)
Create a container group profile using [Create or Update](/rest/api/container-instances/container-groups/create-or-update). You can optionally include config map details in the container group profile. For more information on config maps, see [use config maps](container-instances-config-map.md).


```HTTP
PUT https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile?api-version=2024-05-01-preview  

Request Body
{
    "location": "West Central US",
    "properties":{
        "containers": [
        {
            "name":"mycontainergroupprofile",
            "properties": {
                "command":[],
                "environmentVariables":[],
                "image":"mcr.microsoft.com/azuredocs/aci-helloworld:latest",
                "ports":[
                    {
                        "port":8000
                    }
                ],
                "resources": {
                    "requests": {
                        "cpu":1,
                        "memoryInGB":1.5
                                }
                            }
                        }
                    }
                ],
                "imageRegistryCredentials":[],
                "ipAddress":{
                "ports":[
                    {
                        "protocol":"TCP",
                        "port":8000
                    }
            ],
            "type":"Public"
            },
            "osType":"Linux",
            "sku":"Standard"
            }
        }
```

---

## Create a standby pool

### [CLI](#tab/cli)
Create a standby pool and associate it with a container group profile using [az standby-container-group-pool create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az standby-container-group-pool create \
   --resource-group myResourceGroup \
   --location WestCentralUS \
   --name myStandbyPool \
   --max-ready-capacity 20 \
   --refill-policy always \
   --container-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```
### [PowerShell](#tab/powershell)
Create a standby pool and associate it with a container group profile using [New-AzStandbyContainerGroupPool](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
New-AzStandbyContainerGroupPool `
   -ResourceGroup myResourceGroup `
   -Location "WestCentralUS" `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -RefillPolicy always `
   -ContainerProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```

### [ARM template](#tab/template)
Create a standby pool and associate it with a container group profile. Create a template and deploy it using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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
            "apiVersion": "2025-03-01",
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
Create a standby pool and associate it with a container group profile using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update).

```HTTP
PUT https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool?api-version=2025-03-01 
 
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

## Next steps

[Request a container from the standby pool](container-instances-standby-pool-request-container.md).
