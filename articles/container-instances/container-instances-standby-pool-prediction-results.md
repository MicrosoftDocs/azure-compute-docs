---
title: Get prediction results for standby pools for Azure Container Instances (Preview)
description: Learn how to use prediction results to help right-size your standby pools for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: concept-article
ms.date: 11/17/2025
ms.update-cycle: 1095-days
ms.reviewer: tomvcassidy
# Customer intent: As a cloud administrator, I want to retrieve prediction results for my standby pool, so that I can optimize resource allocation and improve operational efficiency based on forecasted demand.
---

# Get prediction results for standby pools for Azure Container Instances (Preview)

> [!IMPORTANT]
> Prediction results for standby pools is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).


To effectively manage and optimize your standby pool for Azure Container Instances, you can use the Standby Pool runtime view APIs to retrieve prediction results. These results are **available 2-3 weeks after creating the standby pool** and provide insights into the predicted number of instances that will be requested from the pool for each hour over a 12-hour period. The predictions include the accuracy of the forecast and a historical view of instances requested from the pool over the past 12 hours, helping you make informed decisions to right-size your standby pool and improve operational efficiency.

While the prediction results provide valuable insights, they are not a guarantee and should be treated as a suggested size for your pool. The actual number of instances requested from the pool may vary depending on the specific demands of your workload. Additionally, the longer the prediction engine is able to monitor and analyze your workload trends, the more accurate and reliable the prediction results will become over time.

## Prediction information

> [!NOTE]
> Prediction values will not show unless the standby pool has been provisioned for 2-3 weeks. 

The prediction results provide detailed insights into the expected behavior of your standby pool. These results include metadata about the forecast, such as accuracy, time intervals, and historical data, as well as the predicted number of instances requested during the forecast period. Use this information to analyze trends, optimize your standby pool size, and improve operational efficiency. Below is a description of the key values included in the prediction results:

| Value | Description | 
|---|---|
| `forecastInfo` | A JSON string containing additional metadata about the forecast, such as accuracy, time intervals, and recent history of requested instances. |
| `SeriesUnitIntervalInMins` | The time interval, in minutes, between each data point in the forecast. For example, a value of 60 indicates hourly intervals. |
| `InstancesRequestedCountRecentHistory` | An array of historical data showing the number of instances requested from the standby pool over the past 12 hours. |
| `ForecastAccuracy` | The accuracy of the forecast as a percentage. A higher value indicates a more reliable prediction. |
| `forecastStartTime` | The timestamp indicating when the forecast period begins. |
| `instancesRequestedCount` | An array of predicted instance counts for each time interval in the forecast period. |


## Retrieve prediction results

### [CLI](#tab/cli)

```azurecli
az standby-container-group-pool status --resource-group myResourceGroup --name myStandbyPool

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
  "instanceCountsByState": [
    {
      "state": "Creating",
      "count": 0
    },
    {
      "state": "Starting",
      "count": 0
    },
    {
      "state": "Running",
      "count": 12
    },
    {
      "state": "Deleting",
      "count": 0
    }
  ],
  "name": "latest",
  "prediction": {
    "forecastInfo": "{\"SeriesUnitIntervalInMins\":60,\"InstancesRequestedCountRecentHistory\":[10,11,9,11,12,11,10,9,7,11,10,12],\"ForecastAccuracy\":90.0}",
    "forecastStartTime": "2025-05-09T17:00:00-07:00",
    "forecastValues": {
      "instancesRequestedCount": [ 10,10,11,12,9,11,11,12,13,10,10,8]}},
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "status": {
    "code": "HealthState/healthy"
  },
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews"
}
```


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyContainerGroupPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

{
Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest
InstanceCountSummary: {
  "instanceCountsByState": [
    {
      "state": "Creating",
      "count": 0
    },
    {
      "state": "Starting",
      "count": 0
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
}
Name                                 : latest
PredictionForecastInfo               : {"SeriesUnitIntervalInMins":60,"InstancesRequestedCountRecentHistory":[10,11,10,11,12,8,9,11,10,12,7,11],"ForecastAccuracy":85.0}
forecastValues                       : { "instancesRequestedCount": [11,10,10,12,14,16,11,19,10,10,11]}
PredictionForecastStartTime          : 5/10/2025 12:00:00 AM
ProvisioningState                    : Succeeded
ResourceGroupName                    : myResourceGroup
StatusCode                           : HealthState/healthy
Type                                 : Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews
```


### [REST](#tab/rest)

```HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyContainerPoolName}/runtimeViews/{runtimeView}?api-version=2025-03-01

{
  "properties": {
    "instanceCountSummary": [
      {
        "state": "Creating",
        "count": 0
      },
      {
        "state": "Starting",
        "count": 0
      },
      {
        "state": "Running",
        "count": 20
      },
      {
        "state": "Deleting",
        "count": 0
      }
    ],
    "provisioningState": "Succeeded",
    "status": {
      "code": "HealthState/healthy",
      "message": "The pool is healthy."
    },
    "prediction": {
      "forecastValues": {
        "instancesRequestedCount": [ 24,10,20,12,15,10,15,23,14,16,17,19 ]
      },
      "forecastStartTime": "2025-02-14T01:34:59.228Z",
      "forecastInfo": "{\"forecastAccuracy\": 85, \"seriesUnitIntervalInMins\": 60, \"instancesRequestedCount_recentHistory\": \"[9,4,2,8,8,2,3,6,5,3,2,6]\"}"
    }
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyContainerPoolName}/runtimeViews/latest",
  "name": "pool",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews"
}

```

---


## Next steps

Review the most [frequently asked questions](container-instances-standby-pool-faq.md) about standby pools for Azure Container Instances.
