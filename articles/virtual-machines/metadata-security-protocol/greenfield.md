---
title: Deploy a VM or Virtual Machine Scale Set with MSP
description: Learn about deploying a virtual machine or a virtual machine scale set with Metadata Security Protocol (MSP).
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/17/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As a cloud administrator, I want to deploy Virtual Machines with the Metadata Security Protocol enabled, so that I can enhance security during the provisioning process."
---

# Deploy a VM or virtual machine scale set with MSP

This article explains how to enable the Metadata Security Protocol (MSP) feature while provisioning a new virtual machine (VM) or virtual machine scale set.

> [!WARNING]
> Please note that on Windows, the users can enable GuestProxyAgent (GPA) from an ARM template. However, in Linux it's a two-step process. The VM/VMSS is provisioned first and only then can the GPA be configured. 

## Prerequisites

- Ensure that your image of choice is [compatible](./overview.md#compatibility).
- Familiarize yourself with the [basic configuration](./configuration.md#msp-feature-configuration) options.

## Deploy a VM with MSP

### Use an ARM template

To deploy a VM with MSP via an Azure Resource Manager template (ARM template), update `proxyAgentSettings` and ensure that the minimum API version is `2024-03-01`. See the [ARM template examples](./other-examples/arm-templates.md).

### Use the REST API

To use the REST API, deploy a VM as you normally would, but with the MSP configuration also applied. Here's an example that references a linked [advanced configuration](./advanced-configuration.md):

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

### Validate linked rules

After you deploy the VM, validate that the linked rules were applied to your VM. The steps for this validation in a newly provisioned VM are the same as the steps for an [existing VM](./brownfield.md#validate-linked-rules).

## Deploy a virtual machine scale set with MSP

Using the same steps for a virtual machine scale set applies MSP to every VM within that scale set.
