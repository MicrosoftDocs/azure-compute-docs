---
title: Configure role permissions for standby pools
description: Learn how to configure role-based access control (RBAC) permissions for standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/6/2025
ms.update-cycle: 180-days
ms.custom: portal
# Customer intent: "As a cloud administrator managing standby pools, I want to configure role-based access control permissions, so that I can ensure proper functionality and prevent issues with resource creation and management in my subscription."
---

# Configure role permissions for standby pools

Standby pools require specific permissions to create and manage resources in your subscription. Without the correct permissions, standby pools will not function properly. This article explains how to configure role-based access control (RBAC) permissions for standby pools and provides guidance for scenarios where additional permissions may be required.


## Basic permissions 
To allow standby pools to create and manage virtual machines in your subscription, assign the appropriate permissions to the standby pool resource provider.

> [!IMPORTANT]
> These permissions may not fully encompass all scenarios. If your standby pool uses specific resources, such as compute gallery images in other subscriptions, ensure the standby pool resource provider has access to those resources.

To cover as many scenarios as possible, it is suggested to provide the following permissions to the standby pool resource provider: 

- **Virtual Machine Contributor**
- **Network Contributor**
- **Managed Identity Contributor**
- **Compute Gallery Sharing Admin**
- **Compute Gallery Artifacts Publisher**

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

## Cross-subscription resources
If your standby pool uses resources, such as compute gallery images, stored in a different subscription, ensure the standby pool resource provider has access to those resources. You may need to assign roles in the other subscription to grant the required permissions.

## Troubleshoot permission issues

Permission issues are a common cause of problems with standby pools. These issues can manifest in several ways, such as the pool continually creating and deleting instances, failing to create instances, or no instances appearing in the pool. Use the following steps to troubleshoot and resolve these issues:

### Use Log Analytics to Investigate

If your pool is not functioning as expected, use Log Analytics to analyze the logs and identify missing permissions:

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. Go to your Log Analytics workspace associated with the standby pool. Before using log analytics, you first need to configure a log analytics workspace. For more information, see [use Azure Log Analytics to monitor standby pool events](standby-pools-monitor-pool-events.md).
3. Query the `SVMPoolExecutionLog` table to review events related to instance creation and deletion:

```kusto
   SVMPoolExecutionLog
   | where TimeGenerated > ago(24h)
   | project TimeGenerated, EventName, EventStatus, ResourceId, FailureDetails
```

4. Look for errors or failures in the FailureDetails column to identify missing permissions or other issues.


### Check the Runtime View API for Degraded Mode

If your pool is in degraded mode, resource creation will be paused briefly. Use the Runtime View API to check the health status of your pool and determine the reason for degraded mode:

1. Send a GET request to the Runtime View API or other SDKs such as PowerShell or CLI. 

```rest
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPool}/runtime?api-version=2025-03-01
```

2. Review the response for the healthStatus field. If the pool is in degraded mode, the response will include the reason for the degraded state.

3. Address the identified issue, such as missing permissions or resource constraints, to resolve the degraded mode.

### Review Required Permissions for Pooled Instances

If you notice pooled instances recycling or no instances appearing in the pool, review the resources required to create an individual virtual machine. Ensure the standby pool resource provider has the necessary permissions for all required resources, including but not limited to:

- Compute Gallery images
- Virtual networks and subnets
- Managed identities
- Storage accounts

Verify that the roles assigned to the standby pool resource provider include all permissions needed to create and manage virtual machines.

## Next Steps
- Use the [Azure Monitor Logs documentation to explore and analyze logs further.
- Refer to the Runtime View API documentation for more details on checking pool health and status.
- Ensure all required roles are assigned as described in the Basic permissions section.
