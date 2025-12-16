---
title: Reset a Latched Key
description: Learn about key reset for virtual machines.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As a virtual machine administrator, I want to reset the latched key for my VM, so that I can restore its access to necessary services after a key mismatch or loss."
---

# Reset a latched key

If a virtual machine (VM) loses its copy of the latched key, a disk is migrated to a new VM. If any other key mismatch occurs, the VM can't access WireServer or Azure Instance Metadata Service. Resetting the key brings the VM back to a healthy state if the key is lost or unmatched between host and guest.

> [!NOTE]
> The VM owner must request the key reset. Metadata services can't distinguish between an attacker or the Guest Proxy Agent requesting a reset when the key is lost, so resets can't be issued from within the VM.

## Reset a VM's key

The platform always ensures that the `keyIncarnationId` value in the VM model matches the actual key in storage. Incrementing this value triggers a key reset. For more information, see [MSP feature configuration](../configuration.md).

```http
PATCH https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine_Name}?api-version=2024-03-01
{
  "properties": {
    "securityProfile": {
      "proxyAgentSettings": {
        "keyIncarnationId": 10
      }
    }
  }
}
```

### Confirm the reset

Check the `AzureProxyAgentExtension` value of the VM instance view to confirm that the new key is generated. Your new `keyIncarnationId` value appears after the change propagates end to end.

```json
"keyLatchStatus":{
    "status":"RUNNING",
    "message":"Found key details from local and ready to use. - 122",
    "states":{
        "imdsRuleId":"/SUBSCRIPTIONS/{subscription_id}/RESOURCEGROUPS/{resource_group}/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSIMDS/VERSIONS/{data_version}",
        "secureChannelState":"WireServer Enforce - IMDS Enforce",
        "keyIncarnationId":"10",
        "keyGuid":"e3882f98-da8d-4410-8394-06c23462781c",
        "wireServerRuleId":"/SUBSCRIPTIONS/{subscription_id}/RESOURCEGROUPS/{resource_group}/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSWIRESERVER/VERSIONS/{data_version}"
        }
    }
```

> [!NOTE]
> These requests are idempotent. However, if multiple requests are made with multiple `keyIncarnationId` values, there's no guarantee on the number and order of `keyIncarnationId` values that you observe. The final state reflects whichever request used the largest value.

## Reset a virtual machine scale set's key

Key data is unique for each instance in a virtual machine scale set. The key must be reset on a per-instance basis.

You can reset a key only for a particular scale set's VM instance. You can't reset a key for all the scale set instances in a single API.

See the [earlier instructions for resetting a VM's key](#reset-a-vms-key), and substitute in your scale set instance's resource ID. For example:

`https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSet_name}/virtualMachines/{instance_id}?api-version=2024-03-01`
