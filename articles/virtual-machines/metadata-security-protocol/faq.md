---
title: Frequently Asked Questions About MSP
description: Get answers to frequently asked questions about working with the Metadata Security Protocol (MSP) feature.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: faq
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Frequently asked questions about MSP

This article answers common questions from customers who work with the Metadata Security Protocol (MSP) feature. If you don't see a question that can help you, contact the support team.

## Usage

### How do I check if the feature is enabled?

You can check MSP enablement programmatically by using the [GET VM API](/rest/api/compute/virtual-machines/get) to retrieve the virtual machine (VM) model. The `proxyAgentSettings` properties report MSP configuration.

Additionally, the `GuestProxyAgent` instance view in the VM runtime status reports the state of MSP from its perspective in the VM. If MSP is disabled, the value is `"Disabled"`.

### How do I check component health?

The `GuestProxyAgent` instance view reports the status of the Guest Proxy Agent (GPA) and its dependencies. Each component's `status` value shows `RUNNING` when it's healthy.

| Component | Field |
|--|--|
| GPA | `keyLatchStatus.status` |
| eBPF integration | `ebpfProgramStatus.status` |

Here's a full example:

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

### Can I provision a new Linux VM with MSP enabled without including the GPA in my image?

No. Linux provisioning doesn't have a step where the platform can install other software before the customer receives the VM. Also, most Linux customers don't want this model. It's more in line with the Windows philosophy.

For security reasons, MSP doesn't allow you to successfully provision a VM if you didn't establish a secure connection. The GPA is responsible for establishing the secure connection, so it needs to already be present.

If you don't want to use an image with the GPA included, you must create the VM first and then enable MSP afterward. During the preview, we monitor usage patterns and evaluate potential ways to support the scenario, by allowing the GPA to be installed as a VM extension rather than directly included.

## Features

### Is ARM64 supported?

ARM64 support is not available while the MSP feature is in preview.

## Design

### How is the latched key created, and how does it bind to the VM?

An Azure host generates the latched key and stores it as platform data. The key is unique to each VM. You can use it only on that VM's private connection to WireServer and Azure Instance Metadata Service.

### How is the allowlist (policy) provisioned?

The allowlist is defined as part of the VM model. The [MSP feature configuration](configuration.md) article explains how to create an allowlist.

The Guest Proxy Agent retrieves this policy periodically from WireServer, like any other VM metadata. The allowlist allows users to centrally manage the policy in the same way that they would manage other VM settings. It also prevents users within the VM from tampering with it.

### Where is the allowlist enforced (GPA or WireServer)?

Both components play a role in enforcement. MSP uses a shared responsibility model:

- WireServer and Instance Metadata Service are responsible for ensuring that only the trusted delegate (the GPA), and clients that the trusted delegate endorses, can access the VM's metadata and secrets.
  
  Metadata services can't, and shouldn't, perform VM introspection. They alone can't determine which software within a VM made a request.

- The trusted delegate (the GPA) is responsible for determining the identity of clients and endorsing requests.
  
  The GPA can rely on the OS kernel to authoritatively identify which process within the VM made a request.

The most important element is that MSP is a *default-closed* model, whereas other security measures (like in guest firewall rules) are *default-open*. If the GPA is down or otherwise bypassed, WireServer rejects any requests because the latched key doesn't endorse them.

### Why is this feature opt-in and not required or automatically enabled?

Nearly every infrastructure as a service (IaaS) VM or virtual machine scale set in Azure uses Instance Metadata Service. The diversity of workloads requires flexibility. Although the design of MSP abstracts away most breaking changes, if a workload is ultimately dependent on a less secure configuration, enforcing stronger security breaks the workload.

### Why does MSP require a new agent instead of using an Azure Guest Agent update?

The GPA has different responsibilities than the Azure Guest Agent. The GPA also has higher performance requirements.
