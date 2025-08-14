---
title: Enable MSP on an Existing Virtual Machine or Virtual Machine Scale Set
description: Learn about the methods for enabling Metadata Security Protocol (MSP) on existing VMs or virtual machine scale sets.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As an IT administrator managing existing Virtual Machines, I want to enable the Metadata Security Protocol on my VMs or Scale Sets so that I can enhance security and compliance for my cloud infrastructure."
---

# Enable MSP on an existing VM or virtual machine scale set

This article explains the ways that you can enable Metadata Security Protocol (MSP) on an existing virtual machine (VM) or virtual machine scale set. You can enable MSP on a VM by using the Azure portal, an Azure Resource Manager template (ARM template), or the REST API.

## Prerequisites

- Ensure that your image of choice is [compatible](./overview.md#compatibility).
- Familiarize yourself with the [basic configuration](./configuration.md#msp-feature-configuration) options.

## Enable MSP on a VM

### Use the Azure portal

See [Configure MSP via the portal](./other-examples/portal.md).

### Use an ARM template

### Use the REST API

Enable MSP with both WireServer and Azure Instance Metadata Service protected in `Audit` mode:

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

For Windows VMs, if `ProxyAgentSettings.Enabled` is `true`, the `Microsoft.CPlat.ProxyAgent.ProxyAgentWindows` extension is installed by default.

For Linux VMs, setting `ProxyAgentSettings.Enabled` to `true` doesn't implicitly install the Proxy Agent extension. To enable the Proxy Agent through the Proxy Agent extension, there must be an explicit `PUT` REST API call on the `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` extension. The purpose of the Proxy Agent extension is to enable, automatically update, and report the status of the Proxy Agent.

### Validate linked rules

To validate that the linked rules were applied to your VM, check the VM instance view to confirm the status of the `Microsoft.CPlat.ProxyAgent.ProxyAgentWindows` extension for Windows and the `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` extension for Linux.

The `ComponentStatus/ProxyAgentStatus/succeeded` status should have an `imdsRuleId` value and a `wireServerRuleId` value. These values should map to the reference ID of `InVMAccessControlProfile`.

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

## Enable MSP on a virtual machine scale set

The preceding steps for enabling MSP on a VM also apply to a virtual machine scale set.
