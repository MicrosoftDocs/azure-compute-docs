---
title: Standby pools for Azure Container Instances
description: Learn how to utilize standby pools to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: concept-article
ms.date: 10/22/2025
ms.reviewer: tomvcassidy
# Customer intent: As a cloud architect, I want to implement standby pools for Azure Container Instances, so that I can reduce scale-out latency and effectively manage container resources in response to fluctuating traffic demands.
---

# Standby pools for Azure Container Instances

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](container-instances-standby-pool-configure-permissions.md)**.


Standby pools for Azure Container Instances enable you to create a pool of pre-provisioned container groups that can be used in response to incoming traffic. The container groups in the pool are fully provisioned, initialized, and ready to receive work.

:::image type="content" source="media/container-instances-standby-pools/standby-pool-aci-workflow-diagram.png" alt-text="Diagram of the workflow of creating a container using the traditional path vs the standby pool path.":::

## Limitations
Creating and managing a standby pool for Azure Container Instances isn't yet available in the Azure portal. 

## Prerequisites

### Provider Registration 
Register the standby pool resource provider with your subscription using Azure Cloud Shell. Registration can take up to 30 minutes to successfully show as registered. You can rerun the below commands to determine when the feature is successfully registered.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.StandbyPool
```

### Configure Role-based Access Control Permissions
To allow standby pools to create and manage container instances in your subscription, assign the appropriate permissions to the standby pool resource provider. For more detailed steps and information, see [configure role permissions for standby pools in Azure Container Instances](container-instances-standby-pool-configure-permissions.md).

## Using a container from the standby pool

When you require a new container group, you can immediately pull one from the standby pool that is provisioned and running. 

Standby pools only give out container groups from the pool that are fully provisioned and ready to receive work. For example, when the instances in your pool are still being initialized, they aren't in the running state and are't given out when a container is requested. If no instances in the pool are available, Azure Container Instances will default back to net new container group creation. 

## Standby pool size
The number of container groups in a standby pool is determined by setting the `maxReadyCapacity` parameter. When a container group is consumed from the pool, the standby pool automatically begins to refill ensuring that your standby pool maintains the set maximum ready capacity.

Currently, the only available refill policy for standby pools on Azure Container instances is `Always`. 

| Setting | Description | 
|---|---|
| maxReadyCapacity | The maximum number of container groups you want deployed in the pool.|
| refillPolicy | Tells the standby pool to immediately replenish container groups to maintain the maxReadyCapacity. | 

## Container group profile

A container group profile tells the standby pool how to configure the containers in the pool. If you make any changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.


```json
{
    "location":"{location}",
    "properties":{
        "containers": [
        {
            "name":"[mycontainergroupprofile]",
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

## Config maps

A [config map](container-instances-config-map.md) is a property that can be associated with a container group profile and be used to apply container configurations similar to environment variables and secret volumes. However, when using environment variables or secret volumes, restarting the pod is required for the changes to take effect. By using config maps, the configurations can be applied without restarting the container. This enables out of band updates so containers can read the new values without restarting. 

Azure Container Instances can be created with or without config maps and can be updated at any point in time post creation using config maps. Updating config maps in an existing running container group can be accomplished quickly and without causing the container to reboot. 

For more information, see [use config maps](container-instances-config-map.md).

```json
{
    "properties": {
        "containers": [
            {
                "name": "{mycontainergroupprofile}",
                "properties": {
                    "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
                    "ports": [
                        {
                            "port": 80,
                            "protocol": "TCP"
                        }
                    ],
                    "resources": {
                        "requests": {
                            "memoryInGB": 0.5,
                            "cpu": 0.5
                        }
                    },
                    "configMap": {
                        "keyValuePairs": {
                            "key1": "value1",
                            "key2": "value2"
                        }
                    }
                }
            }
        ],
        "osType": "Linux",
        "ipAddress": {
            "type": "Public",
            "ports": [
                {
                    "protocol": "tcp",
                    "port": 80
                }
            ]
        }
    },
    "location": "{location}"
}

```

## Confidential containers
Standby pools for Azure container instances support confidential containers. To utilize [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` in the container group profile.

> [!IMPORTANT] 
> Values passed using config maps are not included in the security policy or validated by the runtime before the file mount is made available to the container. Any values that could have an impact to data or application security can't be trusted by the application during execution and instead should be made available to the container using environment variables.


```json
{
    "location":"{location}",
    "properties":{
        "containers": [
        {
            "name":"{mycontainergroupprofile}",
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
            "sku":"Confidential"
            }
        }


```

## Availability zones

Standby pools for Azure Container Instances supports creating and requesting containers across availability zones. To create a standby pool with instances in specific zones, specify the `zones` parameter in the standby pool create request. 

### [CLI](#tab/cli)

```azurecli-interactive
az standby-container-group-pool create \
   --resource-group myResourceGroup \
   --location WestCentralUS \
   --name myStandbyPool \
   --max-ready-capacity 20 \
   --refill-policy always \
   --zones 1,2,3 \
   --container-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```
### [PowerShell](#tab/powershell)

```azurepowershell-interactive
New-AzStandbyContainerGroupPool `
   -ResourceGroup myResourceGroup `
   -Location "WestCentralUS" `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -RefillPolicy always `
   -Zones 1,2,3 `
   -ContainerProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/mycontainergroupprofile"
```

### [REST](#tab/rest)

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
          },
          "subnetIds": [
            {
              "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVNET/subnets/mySubnet"
            }
          ]
        },
        "zones": [
          "1",
          "2",
          "3"
        ]
      },

    "location": "West Central US"
}
```
---

## Next steps

[Create a standby pool for Azure Container Instances](container-instances-standby-pool-create.md).
