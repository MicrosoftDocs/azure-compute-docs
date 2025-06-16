---
title: Understand the health state of your standby pool
description: Learn how to get and understand the health state of your standby pool using the runtime view API.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: conceptual
ms.date: 5/6/2025
ms.reviewer: ju-shim
---

# Understand the health state of your standby pool

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

The health state of your standby pool provides critical insights into its operational status. By using the Standby Pool runtime view API, you can retrieve the current health state of your standby pool, including details about instance counts, provisioning state, and overall health status. This information allows you to monitor the pool's performance and take proactive measures to address any issues.

## Health state overview

The health state of a standby pool is determined by analyzing various metrics, such as the number of instances in different states (for example, running, deallocated, creating, etc.), provisioning status, and system health indicators. The health state of the pool can be in either Healthy or Degraded. It is also important to review the provisioning state of your pool. The provisioning state can be in a creating, deleting, updating, succeeded or failed state. 

| State | Description | 
|---|---|
| Healthy | The standby pool is functioning as expected, with all instances in the desired state and no issues detected. The pool is ready to provide instances to the scale set as needed. |
| Degraded | The standby pool is experiencing issues provisioning instances successfully. This may be caused by resource constraints, permission issues, or configuration problems such as extension errors. The pool temporarily pauses instance creation for 30 seconds to allow investigation and resolution. After this pause, the pool will attempt to create resources again. For more information on troubleshooting pool errors, see [Use Azure Log Analytics to monitor standby pool events](standby-pools-monitor-pool-events.md). |

## Retrieve health state using the runtime view API

You can use the runtime view API to retrieve the health state of your standby pool. Below are examples of how to query the API using different tools.

### [CLI](#tab/cli)

Run the following Azure CLI command to get the health state of your standby pool:

```azurecli
az standby-vm-pool status --resource-group myResourceGroup --name myStandbyPool

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool/runtimeViews/latest",
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 1,
          "state": "Running"
        },
        {
          "count": 3,
          "state": "Deallocating"
        },
        {
          "count": 7,
          "state": "Deallocated"
        },
         {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ]
   }
  "name": "latest",
  "resourceGroup": "myResourceGroup",
  "provisioningState": "Succeeded",
    "status": {
      "code": "HealthState/healthy",
      "message": "The pool is healthy."
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews"
}
}

```

### [PowerShell](#tab/powershell)
Use the following PowerShell command to retrieve the health state:

```azurepowershell

Get-AzStandbyVMPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool

/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/mmyStandbyPool/runtimeViews/latest
InstanceCountSummary: {{

        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 0
        },
        {
            "state": "Starting",
            "count": 5
        },
        {
            "state": "Running",
            "count": 0
        },
        {
            "state": "Deallocating",
            "count": 10
        },
        {
            "state": "Deallocated",
            "count": 5
        },
         {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
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
Status                       : HealthState/healthy
Message                      : The pool is healthy
ResourceGroupName            : myResourceGroup
Type                         : Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews

```



### [REST API](#tab/rest)
```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyVirtualMachinePoolName}/runtimeViews/{runtimeView}?api-version=2025-03-01

{
  "properties": {
    "instanceCountSummary": [
      {
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 5
          },
          {
            "state": "running",
            "count": 0
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 5
          },
          {
            "state": "starting",
            "count": 0
          },
         {
            "state": "Hibernating",
            "count": 0
        },
        {
            "state": "Hibernated",
            "count": 0
        },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      }
    ]
  },
  "provisioningState": "Succeeded",
    "status": {
      "code": "HealthState/healthy",
      "message": "The pool is healthy."
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/pool/runtimeViews/latest",
  "name": "myStandbyPool",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews",
  "systemData": {
    "createdBy": "pooluser@microsoft.com",
    "createdByType": "User",
    "createdAt": "2024-02-14T23:31:59.679Z",
    "lastModifiedBy": "pooluser@microsoft.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2024-02-14T23:31:59.679Z"
  }
}

```

---

### Interpreting the health state
The health state includes the following key components:

- Instance counts by state: Shows the number of instances in each state (for example, running, deallocated, creating). This helps you understand the current capacity and utilization of the pool.
- Provisioning state: Indicates whether the pool is successfully provisioned or if there are issues that need attention.
- Health status: Provides an overall assessment of the pool's health, such as "healthy" or "unhealthy," along with a message explaining the status.


### Next steps
Learn how to get [prediction results for your standby pool](standby-pools-prediction-results.md).
Review the [frequently asked questions about standby pools for Virtual Machine Scale Sets](standby-pools-faq.md).
