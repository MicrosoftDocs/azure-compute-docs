---
title: Get standby pool and instance details (Preview)
description: Learn how to get details about your standby pool for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/1/2024
ms.reviewer: tomvcassidy
---

# Get standby pool and instance details (Preview)
This article discusses how to retrieve information about your standby pool and the container groups within it. 

## Standby pool details
Use the standby pool runtime view APIs to get the current status of your standby pool including how many container groups are available and their current provisioning state.


### [CLI](#tab/cli)

```azurecli
az standby-container-group-pool status --resource-group myResourceGroup --name myStandbyPool

{
  "id": "/subscriptions/401ef76a-dea9-45da-b19a-db3efced675b/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
  "instanceCountSummary": [
    {
      "instanceCountsByState": [
        {
          "count": 5,
          "state": "Creating"
        },
        {
          "count": 20,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ]
    }
  ],
  "name": "latest",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews"
}

```


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyContainerGroupPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest
InstanceCountSummary         : {{
        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 20
        },
        {
            "state": "Running",
            "count": 5
        },
        {
            "state": "Deleting",
            "count": 0
        }
        ],
    }
  }
Name                         : latest
ProvisioningState            : Succeeded
ResourceGroupName            : myResourceGroup
Type                         : Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews

```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest?api-version=2024-03-01

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
  "name": "latest",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews",
  "properties": {
    "instanceCountSummary": [
      {
        "instanceCountsByState": [
          {
            "state": "Creating",
            "count": 5
          },
          {
            "state": "Running",
            "count": 20
          },
          {
            "state": "Deleting",
            "count": 0
          }
        ]
      }
    ],
    "provisioningState": "Succeeded"
  }
}
```

---

## Next steps
Learn more about [standby pools for Azure Container Instances](container-instances-standby-pool-overview.md).
