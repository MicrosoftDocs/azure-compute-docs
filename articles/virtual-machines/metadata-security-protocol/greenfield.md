---
title: Greenfield
description: Learn more about deploying a VM with MSP
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Deploy a VM or Virtual Machine Scale Sets with MSP
This page explains how to enable the Metadata Security Protocol (MSP) feature while provisioning a new Virtual Machine (VM) or Virtual Machine Scale Sets.

## Prerequisites

-  Ensure your image of choice is [compatible](./overview.md#compatibility).
- Familiarize yourself with the [basic configuration](./configuration.md#msp-feature-configuration) options.

## Deploy VM with MSP

You can deploy a VM with MSP via an ARM template. You will update `proxyAgentSettings` and ensure the minimum API version is `2024-03-01`

See [examples](./other-examples/arm-templates.md).

### With REST API

Deploy a VM as you normally would, but with the MSP configuration also applied. Example referencing a linked [Advanced Configuration](./advanced-configuration.md):

```http
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01

{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "keyIncarnationId": 0,
        "wireServer": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{wireServerProfileName}/versions/{version}"
        },
        "imds": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{imdsProfileName}/versions/{version}"
        }
      }
    },
  }
}
```

#### Validating the linked rules were applied to your VM

Once a VM is deployed, these steps are the same as for an [existing VM](./brownfield.md#validating-the-linked-rules-were-applied-to-your-vm).

## Deploy Virtual Machine Scale Sets with MSP

Applying the same steps to the Virtual Machine Scale Sets model will apply MSP to every VM in the Scale Set.
