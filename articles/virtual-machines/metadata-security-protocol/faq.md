---
title: Frequently asked questions
description: FAQ section
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Frequently asked questions
This page answers the commonly asked questions by customers when working with the Metadata Security Protocol (MSP) feature. If you don't see a question that can help you, reach out to our support team. 

## Usage

### How to check if the feature is enabled?

 - Metadata Security Protocol (MSP) enablement can be checked programmatically using the [GET VM API](https://learn.microsoft.com/rest/api/compute/virtual-machines/get) to retrieve the Virtual Machine (VM) model. The `proxyAgentSettings` properties report MSP configuration.

 - Additionally, the GuestProxyAgent instance view in the VM runtime status, reports the state of MSP from its perspective in the VM. If MSP is disabled, the value shows as `"Disabled"`.

### How do we check component health?

The GuestProxyAgent extension instance view reports the status of the GPA and its dependencies. Each component's `status` value shows `RUNNING` when it's healthy.

| Component | Field |
|--|--|
| Guest Proxy Agent | `keyLatchStatus.status` |
| eBPF integration | `ebpfProgramStatus.status` |

Full example:

```json
{
  "version": "1.0.20",
  "status": "SUCCESS",
  "monitorStatus": {
    "status": "RUNNING",
    "message": "Monitor thread has not started yet."
  },
  "keyLatchStatus": {
    "status": "RUNNING",
    "message": "Found key details from local and ready to use. 122",
    "states": {
      "imdsRuleId": "/SUBSCRIPTIONS/<guid>/RESOURCEGROUPS/<rg_name>/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/...",
      "secureChannelState": "Wireserver Enforce - IMDS Enforce",
      "keyGuid": "e3882f98-da8d-4410-8394-06c23462781c",
      "WireserverRuleId": "/SUBSCRIPTIONS/<guid>/RESOURCEGROUPS/<rg_name>/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/..."
    }
  },
  "ebpfProgramStatus": {
    "status": "RUNNING",
    "message": "Started Redirector with eBPF maps - 143"
  },
  "proxyListenerStatus": {
    "status": "RUNNING",
    "message": "Started proxy listener 127.0.0.1:3080, ready to accept request - 27"
  },
  "telemetryLoggerStatus": {
    "status": "RUNNING",
    "message": "Telemetry event logger thread started."
  },
  "proxyConnectionsCount": 590
}
```

### Can I provision a new Linux VM with MSP enabled without baking the GPA into my image?

No. Linux provisioning doesn't have a step where the platform can install another software before the VM is handed over to the customer. Additionally, this model isn't something most Linux customers want, it's more in line with the Windows philosophy.

For security reasons, MSP doesn't allow a VM to successfully provision if a secure connection isn't established. The GPA is responsible for establishing the secure connection, so it is required to already be present.

If you don't wish to use an image with the GPA baked in, you must create the VM first then enable MSP afterwards. During preview, we monitor usage patterns and evaluate potential ways to support the scenario, by allowing the GPA to be installed as a VM Extension rather than baked in.

## Features

### Is ARM64 Supported?

ARM64 support is available once the MSP feature is Generally Available.

## Design

### How is the latched key created? How does it bind to the VM? Can it be impersonated?

Azure Host generates the latched key and stores it as platform data. The key is unique to each VM and only can be used on that VM's private connection to WireServer + Azure Instance Metadata Service (IMDS). 

### How is the Allowlist (policy) provisioned? Is it done in a secure way?

The Allowlist is defined as part of the VM model. The [Advanced Configuration](configuration.md) guide explains how to create an Allowlist. The Guest Proxy Agent retrieves this policy periodically from WireServer like any other VM metadata. The Allowlist allows users to centrally manage policy the same way they would other VM settings, and prevents users from within the VM from tampering with it.

### Where is the Allowlist enforced? In Guest or in Wireserver?

Both components play a role in enforcement. MSP uses a shared responsibility model:

- Wireserver / IMDS are responsible for ensuring that only the trusted delegate (the GPA) and clients that the trusted delegate endorses can access the VMs metadata and secrets.
  - Metadata services can't, and shouldn't, perform VM introspection, meaning they alone can't determine which software within a VM made a request.
- The trusted delegate (the GPA) is responsible for determining the identity of clients and endorsing requests that approve from.
  - The GPA can rely on the OS Kernel to authoritatively identify which process within the VM made a request.

The most important element is that MSP is a default closed model whereas other security measures (like in guest firewall rules) are default open. If the GPA is down or otherwise bypassed, Wireserver rejects any requests as they aren't endorsed by the latched key. 

### Why is this feature opt-in and not required or automatically enabled?

IMDS is used by nearly every IaaS VM or Virtual Machine Scale Sets in Azure. The diversity of workloads requires flexibility. While the design of MSP abstracts away most breaking changes, ultimately if a workload is dependent on a less secure configuration, enforcing stronger security breaks the workload.

### Why a new agent and not an Azure Guest Agent update?

The Guest Proxy Agent (GPA) has different responsibilities than the Guest Agent. The GPA also has higher performance requirements.
