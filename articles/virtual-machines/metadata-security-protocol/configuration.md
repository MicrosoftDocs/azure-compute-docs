---
title: MSP Feature Configuration
description: Get configuration details for the Metadata Security Protocol (MSP) feature.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 04/22/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As a system administrator, I want to configure Metadata Security Protocol (MSP) for my virtual machines, so that I can enhance metadata access security and protect my workloads from unauthorized access."
---

# MSP feature configuration

Metadata Security Protocol (MSP) offers customization to maximally restrict metadata server access in your workload. This article introduces fundamental concepts that you can quickly enable on any supported virtual machine (VM) or virtual machine scale set.

If you're familiar with how your workload uses metadata services, you can harden access further by following the [guide for advanced configuration](./advanced-configuration.md).

You can configure MSP via:

- [Azure Resource Manager templates](./other-examples/arm-templates.md)
- REST API
- PowerShell
- [Azure portal](./other-examples/portal.md)

## Concepts

| Term | Definition |
|--|--|
| Metadata services | A general term for the industry practice of offering private, non-internet, anonymous HTTP APIs available at a fixed location. The location is typically `169.254.169.254`. Although the exact features, details, and service count vary by cloud provider, they all share a similar architecture and purpose. |
| [Azure Instance Metadata Service](https://aka.ms/azureimds) | A user-facing metadata service that comes with support and full API documentation. It's intended for both Azure and non-Microsoft integrations. It offers a mix of metadata and credentials. |
| [WireServer](https://aka.ms/azurewireserver) | A platform-facing metadata service that organizations use to implement core infrastructure as a service (IaaS) functionality. It isn't intended for general consumption, and Azure offers no support for doing so. Although the consumers of the service are related to an Azure implementation, the service must still be considered in security assessments.<br><br> WireServer resources are treated as privileged by default. However, you shouldn't view Instance Metadata Service resources as nonprivileged. Depending on your workload, Instance Metadata Service can also provide sensitive information. |
| Guest Proxy Agent (GPA) | An in-guest agent that enables MSP protections. |

## Azure Resource Manager fields

You manage all configuration via new properties in the `securityProfile` section of your resource's configuration.

The minimum API version to configure MSP is `2024-03-01`.

### General configuration

`ProxyAgentSettings` is the new property in the VM model for Azure VMs and virtual machine scale sets that you use to configure the MSP feature.

| Parameter name | Type | Default | Details  |
|--|--|--|--|
| `enabled`| `Bool` | `false` | Specifies whether the MSP feature should be enabled on the VM or virtual machine scale set. Default is `false`. This property controls: <ul><li>Applying other settings.</li><li>Platform latched-key generation.</li><li>Automatic GPA installation (when `true`) or uninstallation (when `false`) on a Windows VM or virtual machine scale set.</li></ul> |
| `imds` | `HostEndpointSettings` | Not applicable | Instance Metadata Service-specific configuration. See [Configuration for each metadata service](#configuration-for-each-metadata-service) later in this article. |
| `wireServer` | `HostEndpointSettings` | Not applicable | Settings for the WireServer endpoint. See [Configuration for each metadata service](#configuration-for-each-metadata-service) later in this article. |
| `keyIncarnationId` | `Integer` | `0` | A counter for key generation. Increasing this value instructs MSP to reset the key used for securing a communication channel between guest and host. The GPA notices the key reset and automatically acquires a new one.<br><br>This property is only for recovery and troubleshooting purposes. |

### Configuration for each metadata service

You can configure individual protections for each metadata service. The schema is the same for each service.

Inline and linked configurations are mutually exclusive on a per-service basis.

#### Inline configuration

You can define an inline configuration by using the `mode` property of `HostEndpointSettings`. Inline configurations don't support customization.

In the following table, `mode` is an enumeration that's expressed as `String`.

| `mode` value | GPA behavior | Service behavior |
|--|--|--|
| `Disabled` | The GPA doesn't set up eBPF interception of requests to this service. Requests go directly to the service. | Unchanged. The service does *not* require the GPA to endorse (sign) requests. |
| `Audit` | The GPA intercepts requests to this service and determines if the current configuration authorizes them. The request is always forwarded to the service, but the result and caller information are logged. | Unchanged. The service does *not* require the GPA to endorse (sign) requests. |
| `Enforce` | The GPA intercepts requests to this service and determines if the current configuration authorizes them. If the caller is authorized, the GPA signs the request to endorse it. If the caller isn't authorized, the GPA responds with status code `401` or `403`. All outcomes are recorded in the audit log. | The GPA *must* endorse (sign) requests to the service. Unsigned requests are rejected with a `401` status code. |

> [!IMPORTANT]
> You can't use this property at the same time as `inVMAccessControlProfileReferenceId`.

In WireServer `Enforce` mode, the GPA implicitly requires that a process runs as `Administrator` or `root` to access WireServer. This requirement is equivalent to the in-guest firewall rule that exists even when MSP is disabled, but with a more secure implementation.

#### Linked configuration

The `inVMAccessControlProfiles` resource type defines a per-service configuration that can be linked against a VM or virtual machine scale set. These configurations support full customizations. For full details, see [Advanced configuration for MSP](./advanced-configuration.md).

| `HostEndpointSettings` property | Type | Details |
|--|--|--|
| `inVMAccessControlProfileReferenceId` | `String` | The resource ID of the configuration to apply. |

This property must be a full Resource Manager ID, which takes this form: `/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}`.

> [!IMPORTANT]
> You can't use this property at the same time as `mode`.

#### Full example

- VM: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01`
- Virtual machine scale set: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSet-name}/virtualMachines/{instance-id}?api-version=2024-03-01`

The following example uses `mode`:

```json
{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "wireServer": {
          "mode": "Enforce"
        },
        "imds": {
          "mode": "Audit"
        }
      }
    },
  }
}
```

The following example uses `inVMAccessControlProfileReferenceId:`

```json
{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "wireServer": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
        },
        "imds": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
        }
      }
    },
  }
}
```

## Audit logs

In `Audit` and `Enforce` modes, audit logs are generated on the local disk.

| OS family | Audit log location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |

## Recommended progression

Customers often aren't aware of the full extent of WireServer and Instance Metadata Service usage in their workload. Your VM's usage is a combination of any custom integrations that you create and the implementation details of any other software that you run. We recommend starting with a simpler configuration and iterating over time as you gain more information. In general, it's best to go in this order:

1. Enable MSP for WireServer and Instance Metadata Service, in `Audit` mode. This step enables you to confirm compatibility and learn more about your workload. It also generates valuable data for detecting threat actors.
1. Review the audit data. Pay close attention to any flagged requests. These requests might be signs of an ongoing attack or a workload incompatibility.
1. Remediate any problems. Under the default setup, anything that's flagged is a misuse of the metadata services. Contact your security team or the corresponding service owners as needed.
1. Switch from `Audit` mode to `Enforce` mode. Flagged requests are rejected to help protect your workload.
1. Experiment with [advanced configuration](./advanced-configuration.md). The default configuration mirrors pre-MSP security rules, which are often overly permissive. Defining a custom configuration for your workload enables you to further enhance security by applying the principle of least-privileged access.

Don't worry about following a perfect progression. *Any* level of MSP enablement greatly improves your security and helps protect against most real-world attacks.

## Related content

- [Deploy a VM or virtual machine scale set with MSP](./greenfield.md)
- [Enable MSP on an existing VM or virtual machine scale set](./brownfield.md)
- [Advanced configuration for MSP](./advanced-configuration.md)
