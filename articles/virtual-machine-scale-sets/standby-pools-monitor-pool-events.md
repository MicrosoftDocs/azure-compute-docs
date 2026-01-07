---
title: Use Azure Log Analytics to monitor standby pool events
description: Learn how to use Azure Log Analytics to monitor and analyze events from standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: cynthn
# Customer intent: As an IT administrator managing Virtual Machine Scale Sets, I want to configure Azure Log Analytics to monitor events from standby pools, so that I can track metrics, analyze trends, and receive alerts for critical operational issues.
---

# Use Azure Log Analytics to monitor standby pool events

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

Azure Log Analytics provides a powerful platform for monitoring and analyzing events from standby pools in Virtual Machine Scale Sets. By integrating your standby pools with a Log Analytics workspace, you can track key metrics, analyze trends, and set up alerts for critical events.

## Available metrics and tables

There are two main tables where you can view logs associated with your standby pool: `SVMPoolRequestLog` and `SVMPoolExecutionLog`. 

| Table name | Description | 
|---|---|
| `SVMPoolRequestLog` | Contains logs for user-initiated events, such as updates to pool settings. |
| `SVMPoolExecutionLog` | Contains logs for system-initiated events, such as standby pool operations like degraded mode, VM reuse, and pool refills. |

Within the above tables, you can query on specific pool related events as described below: 

| Event name | Description | 
|---|---|
| `StandbyPoolExhaustedPool` | Triggered when the standby pool instance count reaches zero and can't create more VMs because the pool's max ready capacity is less than or equal to the Virtual Machine Scale Set instance count. This typically occurs when no minimum ready capacity is configured.|
| `StandbyPoolReuseSuccess` | Triggered when a virtual machine is successfully moved from the standby pool into the scale set. |
| `StandbyPoolReuseFailure` | Triggered when the scale set requests a VM from the standby pool but is unable to provide one, causing the scale set to create a new VM directly. |
| `StandbyPoolSettingsUpdated` | Triggered when a setting is changed on the standby pool resource, such as adjusting the min/max ready capacity or the VM state. |
| `StandbyPoolMaxReadyPool` | Triggered when the number of instances in the standby pool are replenished enough to meet the maximum ready capacity set by the customer. |
| `StandbyPoolDegradedPool` | Triggered when the instances within the standby pool are unable to successfully provision the requested resources, causing the pool to enter a degraded mode for 30 seconds. |
| `StandbyPoolExitDegradedPool` | Triggered when the time out on degraded mode expires, and the pool is now attempting to create resources again. |

## Configure Log Analytics for standby pools
A Log Analytics workspace is a centralized data repository in Azure Monitor that allows you to collect, analyze, and query telemetry data from various Azure resources and services.

### Create a log analytics workspace
Before configuring monitoring for standby pools, ensure you have a Log Analytics workspace set up. 

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. In the search bar, type **Log Analytics workspaces** and select it from the results.
3. Click **+ Create**.
4. Fill in the required fields:
   - **Subscription**: Select the subscription to associate with the workspace.
   - **Resource group**: Choose an existing resource group or create a new one.
   - **Name**: Enter a unique name for the workspace.
   - **Region**: Select the region for the workspace.
5. Click **Review + Create**, then **Create** to deploy the workspace.

### Configure diagnostic settings for standby pools
To send information to the log analytics workspace configured, set up a diganostic settings for your standby pool resource. After successfully configuring the diagnostic settings, it takes about 30 minutes before any logs begin showing up in the log analytics workspace. Events that occurred before configuring the log analytics workspace won't be included. 

> [!NOTE]
> Enabling a diagnostic setting for a standby pool resource is not yet available from the Azure portal. Instead enable a diagnostics setting using an alterative SDK such as PowerShell or CLI. 

#### [CLI](#tab/cli)
```azurecli
az monitor diagnostic-settings create \
  --name "standbyPoolLogs" \
  --resource "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPool}" \
  --workspace "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.OperationalInsights/workspaces/{logAnalyticsWorkspace}" \
  --logs '[{"categoryGroup": "allLogs", "enabled": true}]'
```


#### [PowerShell](#tab/powershell)
```azurepowershell
# Create log settings object
$log = New-AzDiagnosticSettingLogSettingsObject -Enabled $true -CategoryGroup allLogs  

# Create a diagnostic setting
New-AzDiagnosticSetting -Name 'standbyPoolLogs' `
  -ResourceId "/subscriptions/{subscrptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPool}" `
  -WorkspaceId "/subscriptions/{subscriptionId}>/resourceGroups/{resourceGroup}/providers/Microsoft.OperationalInsights/workspaces/{logAnalyticsWorkspace}" `
  -Log $log
```

#### [REST](#tab/rest)
```rest
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyPool}/providers/microsoft.insights/diagnosticSettings/standbyPoolLogs?api-version=2021-05-01-preview

{
  "properties": {
    "workspaceId": "/subscriptions/{subscriptionId}>/resourceGroups/{resourceGroup}/providers/Microsoft.OperationalInsights/workspaces/{logAnalyticsWorkspace}>",
    "logs": [
      {
        "categoryGroup": "allLogs",
        "enabled": true
      }
    ]
  }
}
```
---

## Query standby pool events 

1. Go to the [Azure portal](https://portal.azure.com/).
2. In the search bar at the top, type **Log Analytics workspaces** and select it from the results.
3. Select the Log Analytics workspace you configured for your standby pool.
4. In the workspace menu, click **Logs** under the **General** section to open the query editor.

### Query standby pool events

Use the following queries to analyze events from the `SVMPoolRequestLog` and `SVMPoolExecutionLog` tables:

#### View user-initiated events from `SVMPoolRequestLog`
```kusto
SVMPoolRequestLog
| where TimeGenerated > ago(24h)
| project TimeGenerated, EventName, ResourceId, Details
| order by TimeGenerated desc
```

#### View system-initiated events from `SVMPoolExecutionLog`

```kusto
SVMPoolExecutionLog
| where TimeGenerated > ago(24h)
| project TimeGenerated, EventName, ResourceId, Details
| order by TimeGenerated desc
```

#### Count events by type

```kusto
SVMPoolRequestLog
| summarize Count = count() by EventName
| union (
    SVMPoolExecutionLog
    | summarize Count = count() by EventName
)
| order by Count desc
```

## Set up alerts for specific events

To ensure you're notified of critical events, you can set up alerts in Azure Monitor based on the events in the `SVMPoolRequestLog` and `SVMPoolExecutionLog` tables. 

### Create an alert for failed standby pool actions

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. In the search bar, type **Monitor** and select it from the results.
3. In the **Monitor** menu, select **Alerts** under the **Monitoring** section.
4. Click **+ New alert rule**.
5. Configure the alert:
   - **Scope**: Select your Log Analytics workspace.
   - **Condition**: Use the following custom log query:
     ```kusto
     SVMPoolExecutionLog
     | where EventName == "StandbyPoolReuseFailure"
     ```
   - **Action group**: Create or select an action group to define how you want to be notified.
   - **Alert rule details**: Provide a name  for the alert and set the severity level.

6. Click **Create alert rule** to save the alert.

### Create an alert for exhausted standby pools

1. Follow steps 1–4 from the previous example.
2. Configure the alert:
   - **Scope**: Select your Log Analytics workspace.
   - **Condition**: Use the following custom log query:
     ```kusto
     SVMPoolExecutionLog
     | where EventName == "StandbyPoolExhaustedPool"
     ```
   - **Action group**: Create or select an action group for notifications.
   - **Alert rule details**: Provide a name for the alert and set the severity level.

3. Click **Create alert rule** to save the alert.

### Create an alert for frequent pool setting updates

1. Follow steps 1–4 from the first example.
2. Configure the alert:
   - **Scope**: Select your Log Analytics workspace.
   - **Condition**: Use the following custom log query:
     ```kusto
     SVMPoolRequestLog
     | where EventName == "StandbyPoolSettingsUpdated"
     | summarize Count = count() by bin(TimeGenerated, 1h)
     | where Count > 5
     ```
     This query triggers an alert if more than 5 pool setting updates occur within an hour.
   - **Action group**: Create or select an action group for notifications.
   - **Alert rule details**: Provide a name for the alert and set the severity level.

3. Click **Create alert rule** to save the alert.

### Next steps

- Test your alerts by simulating the events in your standby pool.
- Review the [Azure Monitor Alerts documentation](/azure/azure-monitor/alerts/alerts-overview) for more advanced alerting options.
