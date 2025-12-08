---
title: Enable Automatic Zone Balance on Virtual Machine Scale Sets (Preview)
description: Guide to enable the Automatic Zone Balance feature for your virtual machine scale set.
author: hilaryw29
ms.author: hilarywang
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 04/24/2025
---
# Enable Automatic Zone Balance on Virtual Machine Scale Sets (Preview)

This guide provides instructions to enable Automatic Zone Balance on your Virtual Machine Scale Sets.

> [!IMPORTANT]
> Automatic Zone Balance for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

## Prerequisites

Before enabling automatic zone balance on your scale set, ensure the following prerequisites are met:

**Enable application health monitoring for the scale set**

The scale set must have application health monitoring enabled to use automatic zone balance. Health monitoring can be done using either [Application Health Extension](./virtual-machine-scale-sets-health-extension.md) or [Load Balancer Health Probes](/azure/load-balancer/load-balancer-custom-probe-overview), where only one can be enabled at a time. 

The application health status is used to ensure that new virtual machines (VM) created during the rebalance process are successful and "healthy", before the original VMs are removed. 

**Configure the scale set with at least two availability zones**

The Virtual Machine Scale Set must be zonal with at least two [availability zones](./virtual-machine-scale-sets-use-availability-zones.md) configured (for example, `zones = [1, 2]`). This ensures that the VMs can be distributed across multiple zones for resiliency.

**Use a supported Compute API version**

Automatic zone balance is supported for Compute API version 2024-07-01 or higher. 

**Register the subscription with AFEC**

The subscription must be registered with the Azure Feature Exposure Control (AFEC) flag `Microsoft.Compute.AutomaticZoneRebalancing`. This registration is required to enable the automatic zone balance feature for your subscription.

### [Portal](#tab/portal-1)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. In the search box, enter _subscriptions_ and select **Subscriptions**.
1. Select the link for your subscription's name.
1. From the left menu, under **Settings** select **Preview features**.
1. Filter for **AutomaticVMSSZoneRebalancing** and select it
1. Select **Register**
:::image type="content" source=".\media\virtual-machine-scale-sets-auto-zone-balance/auto-zone-balance-register-afec.png" alt-text="Screenshot of Azure portal with Register button for Automatic Zone Balancing preview flag.":::

1. Select **OK**

### [Azure CLI](#tab/CLI-1)

Register the AFEC feature using Azure CLI:

```azurecli
az feature register --namespace "Microsoft.Compute" --name "AutomaticZoneRebalancing"
```

Check the registration status:

```azurecli
az feature show --namespace "Microsoft.Compute" --name "AutomaticZoneRebalancing"
```

### [PowerShell](#tab/PowerShell-1)

Register the AFEC feature using Azure PowerShell:

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName AutomaticZoneRebalancing
```

Check the registration status:

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName AutomaticZoneRebalancing
```
---

## Enable Automatic Zone Balance on your scale set

Follow the steps below to enable Automatic Zone Balance on a new or existing virtual machine scale set.

### [REST API](#tab/rest-api-2)

Add the `resiliencyPolicy` property with the following parameters to your scale set model. Use API version 2024-07-01 or higher.

```
PUT or PATCH on '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2024-07-01'
```
```json
"properties": {
    "resiliencyPolicy": {
      "AutomaticZoneRebalancingPolicy": {
        "Enabled": true,
        "RebalanceStrategy": "Recreate",
        "RebalanceBehavior": "CreateBeforeDelete"
      }
    }
}
```
### [Portal](#tab/portal-2)

**To enable automatic zone balance during scale set creation**

1. Create a new Virtual Machine Scale Set.
1. In the **Basics** tab, ensure that at least 2 zones are selected in the **Availability zone** dropdown.
1. In the **Health** tab, enable and configure **[application health monitoring](./virtual-machine-scale-sets-health-extension.md)**.
1. Navigate to the **Advanced** tab, select the checkbox for **Automatic zone balancing**.
:::image type="content" source="media/virtual-machine-scale-sets-auto-zone-balance/enable-rebalance-create-vmss.png" alt-text="A screenshot showing how to enable automatic zone balance during the Virtual Machine Scale Set create experience in the portal.":::

1. Complete additional set up for your scale set. When ready, select **Review and Create**.
1. Select **Create**.

**To enable automatic zone balance on an existing scale set**

1. Navigate to your Virtual Machine Scale Set.
1. Under **Availability + scale** select **Availability**.
1. In **Availability zone** dropdown, ensure at least 2 zones are selected.
1. Select the checkbox for **Automatic zone balancing**.
1. Select **Apply**.

:::image type="content" source="media/virtual-machine-scale-sets-auto-zone-balance/enable-rebalance-existing-vmss.png" alt-text="A screenshot showing how to enable automatic zone balance on an existing Virtual Machine Scale Set in the portal.":::

### [Azure CLI](#tab/CLI-2)

Provide Azure CLI instructions, once available.

### [Azure PowerShell](#tab/PowerShell-2)

Provide PowerShell instructions, once available.

---

## Disable Automatic Instance Repairs

By default, enabling automatic zone balance also enables [automatic instance repairs](./virtual-machine-scale-sets-automatic-instance-repairs.md) to provide comprehensive high availability for your scale set. If you prefer to use automatic zone balance without automatic instance repairs, you can disable instance repairs separately.

> [!NOTE]
> Disabling automatic instance repairs means your scale set will only benefit from zone-level resiliency, not instance-level health monitoring and repair.

### [Azure CLI](#tab/cli)

```azurecli
# Add Azure CLI example here
```

### [Azure PowerShell](#tab/powershell)

```azurepowershell
# Add PowerShell example here
```

### [REST API](#tab/rest)

When enabling automatic zone balance, set the `automaticRepairsPolicy` property to disabled:

```json
{
  "properties": {
    "resiliencyPolicy": {
      "AutomaticZoneRebalancingPolicy": {
        "Enabled": true,
        "RebalanceStrategy": "Recreate",
        "RebalanceBehavior": "CreateBeforeDelete"
      }
    },
    "automaticRepairsPolicy": {
      "enabled": false
    }
  }
}
```

---

## Frequently Asked Questions

### Why is my scale set not balanced?
Automatic zone balance only runs if your scale set is zonally imbalanced and all [safety conditions](./auto-zone-balance-overview.md#safety-features) are met. For example, there must be no ongoing or recently completed operations on the scale set, and no rebalancing operation from the past 12 hours.

Rebalancing also depends on available capacity—if there isn’t enough capacity in the under-provisioned zone, automatic zone balance can’t create a new VM and the rebalance operation won’t occur. 

### How often does automatic zone balance run?

Automatic zone balance is constantly checking your scale set for zonal imbalance. When an imbalance is detected and there’s an opportunity to rebalance—all safety conditions are met, available capacity in under-provisioned zone;  a rebalance operation starts right away. 

If a rebalance operation already occurred in the past 12 hours, another rebalance won’t happen until that window has passed. These limits help ensure that changes to your scale set are gradual and controlled.

### How do I know if automatic zone balance is enabled on my scale set?
You can check your scale set’s configuration for the `resiliencyPolicy` property. If `AutomaticZoneRebalancingPolicy.Enabled` is set to `true`, automatic zone balance is enabled.

### What happens if my subscription doesn’t have enough quota for a new VM during rebalancing?
If there isn’t enough quota to create a new VM, the rebalance operation won’t proceed. You need to increase your quota to allow rebalancing to occur.

### Can I control which VMs are excluded from rebalancing?
Yes. You can apply an [instance protection policy](./virtual-machine-scale-sets-instance-protection.md) to specific VMs to prevent them from being selected for rebalancing.

## Next Steps
Learn more about [automatic zone balance for Virtual Machine Scale Sets](./auto-zone-balance-overview.md).

