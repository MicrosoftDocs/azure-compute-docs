---
title: Config Maps for Azure Container Instances
description: Learn how to use config maps with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 05/19/2025
ms.reviewer: tomvcassidy
# Customer intent: As a container orchestrator user, I want to implement config maps in Azure Container Instances so that I can modify container configurations dynamically without restarting the instances to ensure high availability and minimize downtime.
---
# Config maps for Azure Container Instances

A config map is a property that you can use to apply container configurations similar to environment variables and secret volumes. However, the process is unlike using environment variables or secret volumes where you must restart the pod to apply the settings. Using a config map to bpply settings doesn't require any restarts for the changes to take effect.

You can use Azure Container Instances to create container instances with or without config maps. You can also update them at any point after creation by using config maps. Updating config maps in an existing running container group is a task you can accomplish quickly without compromising uptime of the container.

## How it works

You can include a config map in the container properties or in a container group profile. Creating a container group profile with the config map settings makes applying those settings simple and easy to automate.

### Create a container group profile with config map settings

### [CLI](#tab/cli)

Create a container group profile with config map settings by using [az container container-group-profile create](/cli/azure/container).

```azurecli-interactive
az container container-group-profile create \
    --resource-group myResourceGroup \
    --name myContainerGroupProfile \
    --location WestCentralUS \
    --image nginx \
    --os-type Linux \ 
    --ip-address Public \ 
    --ports 8000 \ 
    --cpu 1 \
    --memory 1.5 \
    --restart-policy never \
    --config-map key1=value1 key2=value2
```

### [PowerShell](#tab/powershell)

Create a container group profile with config map settings by using [New-AzContainerInstanceContainerGroupProfile](/powershell/module/az.containerinstance).

```azurepowershell-interactive
$port1 = New-AzContainerInstancePortObject -Port 8000 -Protocol TCP
$port2 = New-AzContainerInstancePortObject -Port 8001 -Protocol TCP

$container = New-AzContainerInstanceObject -Name myContainer -Image nginx -RequestCpu 1 -RequestMemoryInGb 1.5 -Port @($port1, $port2) -ConfigMapKeyValuePair @{"key1"="value1"}

New-AzContainerInstanceContainerGroupProfile `
    -ResourceGroupName myResourceGroup `
    -Name myContainerGroupProfile `
    -Location WestCentralUS `
    -Container $container `
    -OsType Linux `
    -RestartPolicy "Never" `
    -IpAddressType Public
```

### [ARM template](#tab/template)

Create a container group profile with config map settings by deploying the template by using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2024-05-01-preview",
      "name": "[parameters('profileName')]",
      "location": "[parameters('location')]",
      "properties": {
        "containers": [
          {
            "name": "myContainer",
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
              "configMap": {
                  "keyValuePairs": {
                        "key1": "value1",
                        "key2": "value2"
                                   }                   
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
      "defaultValue": "myContainerGroupProfile",
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
        "description": "The container image used"
      }
    }
  }
}


```

### [REST](#tab/rest)

Create a container group profile with config map settings by using [Create or Update](/rest/api/container-instances/container-groups/create-or-update).

```HTTP
PUT https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile?api-version=2024-05-01-preview

Request Body
{
    "location": "West Central US",
    "properties":{
        "containers": [
        {
            "name":"myContainerGroupProfile",
            "properties": {
                "command":[],
                "configMap": {
                  "keyValuePairs": {
                        "key1": "value1",
                        "key2": "value2"
                                   }
                            },     
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

### Apply config map settings by using the container group profile

Applying the config map settings stored in a container group profile requires you to update the container and specify the container group profile that should be associated with the update.

### [CLI](#tab/cli)

Apply the config map settings stored in the container group profile by using [az container create](/cli/azure/container).

```azurecli-interactive
az container create 
        --resource-group myResourceGroup \ 
        --name myContainer \ 
        --location WestCentralUS \
        --container-group-profile-id "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" \
        --container-group-profile-revision 1 

```

### [PowerShell](#tab/powershell)

Apply the config map settings stored in the container group profile by using [New-AzContainerGroup](/powershell/module/az.containerinstance/new-azcontainergroup).

```azurepowershell-interactive
$container = New-AzContainerInstancenoDefaultObject -Name myContainer

New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name myContainer`
    -Container $container `
    -Location WestCentralUS `
    -ContainerGroupProfileId "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" `
    -ContainerGroupProfileRevision 1 
                    
```

### [ARM template](#tab/template)

Apply the config map settings stored in the container group profile by using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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
    },
    "containerGroupProfileName": {
      "type": "string",
      "metadata": {
        "description": "The name of the container group profile."
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
        "containerGroupProfile": {
          "id": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.ContainerInstance/containerGroupProfiles/', parameters('containerGroupProfileName'))]",
          "revision": "[parameters('revisionNumber')]"
        },
        "containers": [
          {
            "name": "[parameters('myContainerProfile')]",
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

Apply the config map settings stored in the container group profile by using [Create or Update](/rest/api/container-instances/container-groups/create-or-update).

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2024-05-01-preview 

Request Body
{
   "location": "{location}",
   "properties": {
           "containerGroupProfile": {
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile",
               "revision": {revisionNumber}
           },
           "containers": [
               {
                   "name": "{myContainerGroupProfile}",
                   "properties": {
                   }
               }
           ]
   }
}
```

---

### Apply config map settings without the container group profile

You can also apply config map settings directly to the instance by specifying the config map settings in the create commands.

### [CLI](#tab/cli)

Apply the config map settings by using [az container create](/cli/azure/container).

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \ 
    --name myContainer \
    --location WestCentralUS \ 
    --config-map key1=value1 key2=value2 
        

```

### [PowerShell](#tab/powershell)

Apply the config map settings by using [New-AzContainerGroup](/powershell/module/az.containerinstance/new-azcontainergroup).

```azurepowershell-interactive
$container = New-AzContainerInstancenoDefaultObject -Name myContainer -ConfigMapKeyValuePair @{"key1"="value1"}

New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name myContainerGroup ` 
    -Container $container `
    -Location WestCentralUS 

```

### [ARM template](#tab/template)

Apply the config map settings by using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

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
  },
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2024-05-01-preview",
      "name": "[parameters('containerGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "containers": [
          {
            "name": "[parameters('myContainerProfile')]",
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

Apply the config map settings by using [Create or Update](/rest/api/container-instances/container-groups/create-or-update).

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2024-05-01-preview 

Request Body
{
   "location": "{location}",
   "properties": {
           "containers": [
               {
                   "name": "{myContainerGroupProfile}",
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
```

---

### ConfigMaps in Linux containers

After the update is applied to an existing container, you see the values mounted in the Linux container without requiring a restart.

```
/mnt/configmap/<containername>/key1 with value as “value1”

/mnt/configmap/<containername>/key2 with value as “value2”
```

### Config maps in Windows containers

After the update is applied to an existing container, you can fetch the config map key/value pairs in the Windows container by making the following call, without requiring a restart. These values aren't mounted anywhere for Windows containers, as is the case for Linux.

```
Invoke-Expression "$Env:ConfigMapURI"
```

## Related content

- Learn how to use config maps with [standby pools to increase scale and availability](container-instances-standby-pool-get-details.md).
