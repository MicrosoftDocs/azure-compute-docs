---
title: Enable MSP on existing Virtual Machine or Virtual Machine Scale Sets
description: MSP on preexisting VMs
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Enable MSP on existing VM or Virtual Machine Scale Sets
This page explains the different ways 
Metadata Security Protocol (MSP) can be enabled on existing Virtual Machines (VM) or Virtual Machine Scale Sets. MSP can be enabled via Portal, ARM template, or the REST API in preexisting VMs. By default, for Windows VMs if ProxyAgentSettings.Enabled is true the `Microsoft.CPlat.ProxyAgent.ProxyAgentWindows` extension will be installed. For Linux VMs, setting ProxyAgentSettings.Enabled to true will not implicitly install Proxy Agent Extension. There must be an explicit PUT Rest API call on the `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` extension to enable Proxy Agent through the Proxy Agent Extension. The purpose of the Proxy Agent Extension is to enable, auto-update, and report status of Proxy Agent. 

## Prerequisites

-  Ensure your image of choice is [compatible](./overview.md#compatibility).
- Familiarize yourself with the [basic configuration](./configuration.md#configuration) options.

## Enable MSP on a VM

### With Azure portal

See [examples](./other-examples/portal.md).

### With ARM template

### With REST API

Enable with both services protected in `Audit` mode:

```http
PATCH https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01

{
  "properties": {
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "wireServer": {
          "mode": "Audit"
        },
        "imds": {
          "mode": "Audit"
        }
      }
    },
  }
}
```

#### Validating the linked rules were applied to your VM

Check the VM Instance View to confirm the status of the `Microsoft.CPlat.ProxyAgent.ProxyAgentWindows` extension for Windows and `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` extension for Linux.
The `ComponentStatus/ProxyAgentStatus/succeeded` status would have an `imdsRuleId` & `wireServerRuleId` which should map to the reference ID of the `InVMAccessControlProfile`.

```json
"extensions": [
    {
      "name": "AzureGuestProxyAgentExtension",
      "type": "Microsoft.CPlat.ProxyAgent.ProxyAgentWindows",
      "typeHandlerVersion": "1.0.21",
      "substatuses": [
        {
          "code": "ComponentStatus/ProxyAgentConnectionSummary/succeeded",
          "level": "Info",
          "displayStatus": "Provisioning succeeded",
          "message": "[{\"userName\":\"SYSTEM\",\"ip\":\"169.254.169.254\",\"port\":80,\"processCmdLine\":\"\\\"C:\\\\Packages\\\\Plugins\\\\Microsoft.Azure.Geneva.GenevaMonitoring\\\\2.45.0.4\\\\GenevaMonitoringExtension.exe\\\" collectLogs\",\"responseStatus\":\"200 OK\",\"count\":344,\"userGroups\":[],\"processFullPath\":\"C:\\\\Packages\\\\Plugins\\\\Microsoft.Azure.Geneva.GenevaMonitoring\\\\2.45.0.4\\\\GenevaMonitoringExtension.exe\"]"
        },
        {
          "code": "ComponentStatus/ProxyAgentStatus/succeeded",
          "level": "Info",
          "displayStatus": "Provisioning succeeded",
          "message": "{\"version\":\"1.0.21\",\"status\":\"SUCCESS\",\"monitorStatus\":{\"status\":\"RUNNING\",\"message\":\"proxy_agent_status thread started.\"},\"keyLatchStatus\":{\"status\":\"RUNNING\",\"message\":\"Found key details from local and ready to use. - 122\",\"states\":{\"imdsRuleId\":\"/SUBSCRIPTIONS/A53F7094-A16C-47AF-ABE4-B05C05D0D79A/RESOURCEGROUPS/HUIYARG-EASTUS2EUAP/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSIMDS/VERSIONS/1.3.0\",\"secureChannelState\":\"WireServer Audit -  IMDS Audit\",\"keyGuid\":\"7512289d-6e5c-449b-9cbf-06728c1c1e4a\",\"wireServerRuleId\":\"/SUBSCRIPTIONS/A53F7094-A16C-47AF-ABE4-B05C05D0D79A/RESOURCEGROUPS/HUIYARG-EASTUS2EUAP/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSWIRESERVER/VERSIONS/1.2.0\"}},\"ebpfProgramStatus\":{\"status\":\"RUNNING\",\"message\":\"Started Redirector with eBPF maps - 79\"},\"proxyListenerStatus\":{\"status\":\"RUNNING\",\"message\":\"Started proxy listener 127.0.0.1:3080, ready to accept request - 9\"},\"telemetryLoggerStatus\":{\"status\":\"RUNNING\",\"message\":\"Telemetry event logger thread started.\"},\"proxyConnectionsCount\":259356}"
        },
        {
          "code": "ComponentStatus/EbpfStatus/succeeded",
          "level": "Info",
          "displayStatus": "Provisioning succeeded",
          "message": "Ebpf Drivers successfully queried."
        }
      ],
      "statuses": [
        {
          "code": "ProvisioningState/succeeded",
          "level": "Info",
          "displayStatus": "Provisioning succeeded",
          "message": "Update Proxy Agent command output successfully",
          "time": "2024-09-24T16:42:59+00:00"
        }
      ]
    }
```

## Enable MSP on a Virtual Machine Scale Sets

Steps above also apply to the Virtual Machine Scale Sets model.
