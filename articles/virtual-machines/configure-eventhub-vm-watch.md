---
title:       Configure Event Hubs for VM watch
description: Get started to configure Event Hubs for VM watch.
author:      ofemifowode
ms.author:   ofemifowode 
ms.service:   azure-virtual-machines
ms.topic:     get-started
ms.date:     02/05/2025
# Customer intent: "As a system administrator, I want to configure Event Hubs for VM watch, so that I can efficiently collect and analyze signal data from my virtual machines in real-time."
---

# Configure Event Hubs for VM watch

VM watch can send signal data to a preconfigured [Event Hub](/azure/event-hubs/event-hubs-about).

This article provides instructions on configuring Event Hubs to access signals collected by VM watch

### Prerequisites
This article assumes that you're familiar with:
- [Azure Event Hubs](/azure/event-hubs/event-hubs-about)
- [VM watch checks, metrics, and logs](/azure/virtual-machines/azure-vm-watch)
- [Install VM watch to virtual machines and scale sets](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2) 

### Enable Event Hubs Output

#### 1: Prepare Event Hubs for VM watch
- [Deploy an Event Hub](/azure/event-hubs/event-hubs-create)
- [Authorize access to the Azure Event Hub](/azure/event-hubs/authorize-access-event-hubs)
> [!IMPORTANT]
> VM watch supports managed identity, SAS token, and connection string as authentication methods. When multiple authentication methods are provided, managed identity is prioritized as the highest, while the connection string is assigned the lowest priority.

#### 2: Enable Event Hubs output for VM watch

For each Event Hub authentication method, you need to combine both the common and authentication specific parameter settings. Instructions are given for each authentication scenario. For virtual machines and virtual machine scale sets, specify the following settings within `vmWatchSettings` in the JSON configurations. 
See [Configure VM watch](/azure/virtual-machines/configure-vm-watch) for instructions on how to access `vmWatchSettings` using [ARM template](/azure/azure-resource-manager/templates/), [Azure CLI](/cli/azure/), or [PowerShell](/powershell/).

##### Common parameters for Event Hubs output

For all authentication methods, the following parameter set applies:

|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_NAMESPACE`|Yes|Event hub name space name, without the domain name `".servicebus.windows.net"`|
|`EVENT_HUB_OUTPUT_NAME`|Yes|Event hub name within the given namespace|
|`EVENT_HUB_OUTPUT_DOMAIN_NAME`|No|Event hub domain name. Default value `"servicebus.windows.net"`|
|`EVENT_HUB_OUTPUT_CLOSE_TIMEOUT`|No|Client close time-out. Default is 30s|
|`EVENT_HUB_OUTPUT_PARTITION_ID`|No|Metric tag or field name to use for the event partition key. Default is null|
|`EVENT_HUB_OUTPUT_MAX_MESSAGE_SIZE`|No|The maximum batch message size in bytes. Setting this parameter to 0 means using the default size from the Azure Event Hubs Client library (1,000,000 bytes). Default is 0|
|`SEND_INTERNAL_TELEMETRY_TO_EVENT_HUB`|No|To receive VM watch internal metrics (startup and heartbeat events), set this value to "true." Default is "false"|

##### Authentication specific parameters for Event Hubs output

#### [Managed Identity](#tab/managedidentity-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY`|No|Set this value to "true." Default is "false"|
|`EVENT_HUB_OUTPUT_MANAGED_IDENTITY_CLIENT_ID`|No|If you're using a specific managed identity to authenticate, specify this value|

For example, the following VM watch JSON configuration sets the environment variables `EVENT_HUB_OUTPUT_NAMESPACE`, `EVENT_HUB_OUTPUT_NAME`, and `EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY`. This allows Event Hubs to use managed identity as the authentication method without needing to specify a managed identity client ID.

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY": "true"
    }
  }
}
```

#### [SAS Token](#tab/sastoken-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_SAS_TOKEN_BASE64`|No|If using SAS token to authenticate, specify this value|

For example, the following VM watch JSON configuration enables Event Hubs as an output by using a SAS token for authentication.

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_SAS_TOKEN_BASE64": "<base 64 encoded SAS token>"
    }
  }
}
```

#### [Connection String](#tab/connectionstring-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_CONNECTION_STRING_BASE64`|No|If using connection string token to authenticate, encode it with base64. The connection string should follow the following format: `Endpoint=sb://<NamespaceName>.<DomainName>/;SharedAccessKeyName=<KeyName>;SharedAccessKey=<KeyValue>`. This should **not** include the entity path `;EntityPath=<EventHubName>` in the connection string.|

For example, the following VM watch JSON configuration enables Event Hubs as an output by using a connection string for authentication

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_CONNECTION_STRING_BASE64": "<base 64 encoded connection string>"
    }
  }
}
```
---


#### 3: Examine events in Event Hubs

Once VM watch settings are successfully configured to use Event Hubs as the output, VM watch restarts. Events start flowing into Event Hubs within a few minutes. You can use the [Azure portal](https://portal.azure.com/) to observe the incoming messages. 

The following screenshot shows data flowing into the Event Hub

:::image type="content" source="./media/vm-watch-data-view.png" alt-text="Screenshot that shows VM watch data flowing into Event Hubs." lightbox="./media/vm-watch-data-view.png":::

Also, you can use the [Event Hubs Data Explorer](/azure/event-hubs/event-hubs-data-explorer) feature to view incoming event and content. 

The following screenshot shows Event Hubs Data Explorer

:::image type="content" source="./media/vm-watch-data-explorer-view.png" alt-text="Screenshot that shows Event Hubs data explorer." lightbox="./media/vm-watch-data-explorer-view.png":::



#### Event Hubs Events schema

Each Event Hub event has the following schema:

|**Field Name**|**Data Type**|**Description**|
| -------- | -------- | -------- |
|`DateTime`|time|The time the signal was emitted|
|`SignalType`|string|The type of the signal, which can be either "StartUp," "Heartbeat," "Check," "Metric," or "EventLog"|
|`SignalName`|string|The name of the signal|
|`SubscriptionId`|string|The VM subscription ID|
|`ResourceGroup`|string|The VM resource group name|
|`ResourceId`|string|The Azure resource URI of the VM|
|`VmId`|string|The unique ID of the VM queried from IMDS endpoint within the VM|
|`Vmss`|string|The virtual machine scale set name, if applicable|
|`Offer`|string|The Azure VM offer|
|`VmSize`|string|The VM size|
|`MeasurementTarget`|string|The target that the signal is measuring at. `Name` field and `MeasurementTarget` are used together for raw signal aggregation|
|`SignalValue`|json|The value of this signal, where the schema depends on the SignalType|
|`Version`|string|The version of the Event Hub output event schema|

#### Debug Event Hubs connection issues

If there are no events in Event Hubs after several minutes, check the VM watch logs in the following directories on the virtual machine or virtual machine scale set to diagnose the issue:

#### [Linux](#tab/linux-1)
```
/var/log/azure/Microsoft.ManagedServices.ApplicationHealthLinux/vmwatch.log
```
#### [Windows](#tab/windows-1)
```
C:/WindowsAzure/Logs/Plugins/Microsoft.ManagedServices.ApplicationHealthWindows/vmwatch.log
```
---

### Next steps

- [Configure VM watch](/azure/virtual-machines/configure-vm-watch) 

- [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite)

- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)

- [VM watch overview](/azure/virtual-machines/azure-vm-watch)
