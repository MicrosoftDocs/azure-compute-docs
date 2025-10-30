---
title: Configure role permissions for standby pools in Azure Container Instances
description: Learn how to configure role-based access control (RBAC) permissions for standby pools in Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 5/19/2025
ms.reviewer: tomvcassidy
# Customer intent: "As a cloud admin, I want to configure role-based access control for standby pools in Azure Container Instances, so that I can ensure proper permissions for managing resources and prevent operational issues."
---

# Configure role permissions for standby pools in Azure Container Instances

Standby pools for Azure Container Instances require specific permissions to create and manage resources in your subscription. Without the correct permissions, standby pools will not function properly. This article explains how to configure role-based access control (RBAC) permissions for standby pools and provides guidance for scenarios where additional permissions may be required.


## Basic permissions 
To allow standby pools to create and manage container instances in your subscription, assign the appropriate permissions to the standby pool resource provider.

> [!NOTE]
> These permissions may not fully encompass all scenarios. If your standby pool uses specific resources, such as container images stored in Azure Container Registry or other subscriptions, ensure the standby pool resource provider has access to those resources.

To cover as many scenarios as possible, it is suggested to provide the following permissions to the standby pool resource provider:

- **Azure Container Instances Contributor**
- **Standby Container Group Pool Contributor**
- **Network Contributor**
- **Managed Identity Contributor**
- **Managed Identity Operator**
- **Storage Blob Data Contributor** (if using Azure Storage for container data)
- **Container Registry Repository Reader** (if using images stored in Azure Container Registry)

1. In the Azure portal, navigate to your subscriptions.
1. Select the subscription you want to adjust permissions for.
1. Select Access Control (IAM).
1. Select Add and Add role assignment.
1. Under the Role tab, search for **Azure Container Instances Contributor** and select it.
1. Move to the Members tab.
1. Select + Select members.
1. Search for **Standby Pool Resource Provider** and select it.
1. Move to the Review + assign tab.
1. Apply the changes.
1. Repeat the above steps and assign the **Network Contributor** and **Managed Identity Contributor** roles to the Standby Pool Resource Provider. If you're using Azure Container Registry or Azure Storage, assign the **Container Registry Repository Reader** and **Storage Blob Data Contributor** roles as well.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).

## Cross-subscription resources
If your standby pool uses resources, such as container images stored in Azure Container Registry, in a different subscription, ensure the standby pool resource provider has access to those resources. You may need to assign roles in the other subscription to grant the required permissions.

## Troubleshoot permission issues

Permission issues are a common cause of problems with standby pools. These issues can manifest in several ways, such as the pool continually creating and deleting instances, failing to create instances, or no instances appearing in the pool. Use the following steps to troubleshoot and resolve these issues:

### Use Log Analytics to investigate

If your pool is not functioning as expected, use Log Analytics to analyze the logs and identify missing permissions:

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. Go to your Log Analytics workspace associated with the standby pool. Before using log analytics, you first need to configure a log analytics workspace. For more information, see [use Azure Log Analytics to monitor standby pool events](container-instances-standby-pools-monitor-pool-events.md).
3. Query the `SCGPoolExecutionLog` table to review events related to instance creation and deletion:

```kusto
   SCGPoolExecutionLog
   | where TimeGenerated > ago(24h)
   | project TimeGenerated, EventName, EventStatus, ResourceId, FailureDetails
```

4. Look for errors or failures in the FailureDetails column to identify missing permissions or other issues.


### Check the runtime view API for degraded mode

If your pool is in degraded mode, resource creation will be paused briefly. Use the Runtime View API to check the health status of your pool and determine the reason for degraded mode:

1. Send a GET request to the Runtime View API or other SDKs such as PowerShell or CLI. 

```rest
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyPool}/runtime?api-version=2025-03-01
```

2. Review the response for the healthStatus field. If the pool is in degraded mode, the response will include the reason for the degraded state.

3. Address the identified issue, such as missing permissions or resource constraints, to resolve the degraded mode.

### Review required permissions for pooled instances

If you notice pooled container instances recycling or no instances appearing in the pool, review the resources required to create an individual container instance. Ensure the standby pool resource provider has the necessary permissions for all required resources, including but not limited to:

- **Container Instance Contributor**
- **Network Contributor**
- **Managed Identity Contributor**
- **Storage Blob Data Contributor** (if using Azure Storage for container data)
- **Azure Container Registry Reader** (if using images stored in Azure Container Registry)

Verify that the roles assigned to the standby pool resource provider include all permissions needed to create and manage container instances.

## Next steps
- Use the Azure Monitor Logs documentation to explore and analyze logs further.
- Refer to the Runtime View API documentation for more details on checking pool health and status.
- Ensure all required roles are assigned as described in the Basic permissions section.
