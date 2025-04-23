---
title: MSP Feature Configuration
description: Configure MSP
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/22/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# MSP Feature Configuration

Metadata Security Protocol (MSP) offers customization to maximally restrict metadata server access in your workload. This page introduces the fundamental concepts that can be quickly enabled on any supported Virtual Machine (VM) or Virtual Machine Scale Sets.

Users that are more familiar how their workload uses metadata services can harden access further by following the [Advanced Configuration guide](./advanced-configuration.md).

## Register the feature flags

To use MSP in preview, register the following flag using the `az feature register` command.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name ProxyAgentPreview
```

Verify the registration status by using the `az feature show` command. It takes a few minutes for the status to show *Registered*:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name ProxyAgentPreview
```

When the status reflects *Registered*, refresh the registration of the *Microsoft.Compute* resource provider by using the `az provider register` command:

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```
Once you are registered in the feature flag you can configure MSP via: 

- [ARM Templates](./other-examples/arm-templates.md)
- REST API 
- PowerShell
- [Azure portal](./other-examples/portal.md)


## Concepts

| Term | Definition |
|--|--|
| "Metadata Services" | A general term for the industry practice of offering private, non-internet anonymous HTTP APIs available at a fixed location. Typically `169.254.169.254`. While the exact features, details, and service count vary by cloud provider, they all share a similar architecture and purpose. |
| [IMDS](https://aka.ms/azureimds) | A user facing metadata service that comes with support and full API documentation. It's intended for both Azure and third party integrations. It offers a mix of metadata and credentials. |
| [WireServer](https://aka.ms/azurewireserver) | A platform facing metadata service, used to implement core IaaS functionality. It isn't intended for general consumption and Azure offers no support for doing so. While the consumers of the service are Azure implementation related, the service must still be considered in security assessments. WireServer resources are treated as privileged by default. However, Azure Instance Metadata Service (IMDS) resources shouldn't be viewed as nonprivileged. Depending on your workload, IMDS can also provide sensitive information. |
| Guest Proxy Agent (GPA) | An in-guest agent that enables MSP protections. |

## ARM Fields

All configuration is managed via new properties in the `securityProfile` section of your resource's configuration.

> The minimum API version to configure MSP is `2024-03-01`

### General Configuration

`ProxyAgentSettings` is the new property in the VM model for Azure VM and Virtual Machine Scale Sets to configure MSP feature.

| Parameter Name | Type | Default | Details  |
|--|--|--|--|
| `enabled`| `Bool` | `false` | Specifies whether MSP feature should be enabled on the virtual machine or virtual machine scale set. Default is `false`. This property controls: <ul><li>Applying other settings</li><li>Platform latched key generation</li><li>Automatic GPA Installation (when `true`) / Uninstallation (when `false`) on Windows VM/Virtual Machine Scale Sets</li></ul> |
| `imds` | HostEndpointSettings | N/A | IMDS specific configuration. See [Per-Metadata Service Configuration](#per-metadata-service-configuration). |
| `wireServer` | HostEndpointSettings | N/A | Settings for the Wireserver endpoint. See [Per-Metadata Service Configuration](#per-metadata-service-configuration). |
| `keyIncarnationId` | `Integer` | `0` | A counter for key generation. Increasing this value instructs MSP to reset the key used for securing communication channel between guest and host. The GPA notices the key reset and automatically acquires a new one. (*This property is only for used recovery/troubleshooting purposes.*) |

### Per-Metadata Service Configuration

Protections for each metadata service can be individually configured. The schema is the same for each service.

> Inline and linked configurations are mutually exclusive on a per-service basis.


#### Inline Configuration

An inline configuration can be defined using the `mode` property of `HostEndpointSettings`. Inline configurations don't support customization.

| `mode` (Enum; expressed as `String`) | GPA Behavior | Service Behavior |
|--|--|--|
| `"Disabled"` | The GPA doesn't set up eBPF interception of requests to this service. Requests go directly to the service. | Unchanged. The service does **not** require that requests are endorsed (signed) by the GPA. |
| `"Audit"` | The GPA intercepts requests to this service and determines if they're authorized by the current configuration. The request is always forwarded to the service, but the result + caller information is logged. | Unchanged. The service does **not** require that requests are endorsed (signed) by the GPA. |
| `"Enforce"` | The GPA intercepts requests to this service and determines if they are authorized by the current configuration. If the caller is authorized, GPA signs the request to endorse it. If the caller isn't authorized, the GPA responds with status code `401` or `403`. All outcomes are recorded in the audit log. | Requests to the service **must** be endorsed (signed) by the GPA. Unsigned requests are rejected with a `401` status code. |

> This property cannot be used at the same time as `inVMAccessControlProfileReferenceId`!

> In WireServer `Enforce` mode, the GPA will implicitly require that a process runs as `Administrator` / `root` to access WireServer. This is equivalent to the in-guest firewall rule that exists even when MSP is disabled but with a more secure implementation.

#### Linked configuration

The `inVMAccessControlProfiles` resource type defines a per-service configuration which can be linked against VM or Virtual Machine Scale Sets. These support full customizations. See [Advanced Configuration](./advanced-configuration.md) for full details.

| `HostEndpointSettings` property | Type | Details |
|--|--|--|
| `inVMAccessControlProfileReferenceId` | `String` | The resource ID of the configuration to apply. |

> Must be a full ARM ID, which takes the form: `/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}`

> This property cannot be used at the same time as `mode`!

#### Full Example

- VM: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01`
- Virtual Machine Scale Sets: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSet-name}/virtualMachines/{instance-id}?api-version=2024-03-01`

##### Using `mode`

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

##### Using `inVMAccessControlProfileReferenceId`

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


## Get Audit Logs

In `Audit` and `Enforce` modes, audit logs are generated on the local disk.

| OS Family | Audit Log Location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |

## Recommended Progression

Customers often aren't aware of the full extent WireServer + IMDS usage in their workload. Your VM's usage is a combination of any custom integrations you create and the implementation details of any other software you run. We recommend starting with a simpler configuration, and iterating over time as you gain more information. In general, it's best to go in this order:

1. Enable MSP for Wireserver and IMDS, in `Audit` mode. This allows you to confirm compatibility, learn more about your workload, and generates valuable telemetry for detecting threat actors.
1. Review the audit telemetry, paying close attention to any flagged requests. There can be signs of an ongoing attack or a workload incompatibility.
1. Remediate any issues. Under the default setup, anything being flagged is a misuse of the metadata services. Contact your security team and/or the corresponding service owners as needed.
1. Switch from `Audit` to `Enforce` mode. Flagged requests are rejected, protecting your workload.
1. Experiment with [Advanced Configuration](./advanced-configuration.md). The default configuration mirror's pre-MSP security rules, which are often overly permissive. Defining custom configuration for your workload enables you to further enhance security by applying the principle of least privileged access.

*Don't make the perfect enemy of the good*. **Any** level of MSP enablement greatly improves your security and protects against most previous real world attacks.

## Next Steps

- [Deploy a VM or Virtual Machine Scale Sets With MSP](./greenfield.md)
- [Enable MSP on Existing VM or Virtual Machine Scale Sets](./brownfield.md)

## Related Content

- [Advanced Configuration guide](./advanced-configuration.md)
