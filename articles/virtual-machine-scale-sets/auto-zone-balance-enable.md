---
title: Enable Automatic Zone Balance on Virtual Machine Scale Sets
description: Guide to enable the Automatic Zone Balance feature for your virtual machine scale set.
author: hilaryw29
ms.author: hilarywang
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 04/24/2025
---
# Enable Automatic Zone Balance on Virtual Machine Scale Sets

This guide provides instructions to enable Automatic Zone Balance on your Virtual Machine Scale Sets.

## Prerequisites

Before enabling automatic zone balance on your scale set, ensure the following prerequisites are met:

**Enable application health monitoring for the scale set**

The scale set must have application health monitoring enabled to use automatic zone balance. Health monitoring can be done using either [Application Health Extension](./virtual-machine-scale-sets-health-extension.md) or [Load Balancer Health Probes](/azure/load-balancer/load-balancer-custom-probe-overview), where only one can be enabled at a time. 

The application health status is used to ensure that new virtual machines (VM) created during the rebalance process are successful and "healthy," before the original VMs are removed. 

**Configure the scale set with at least two availability zones**

The Virtual Machine Scale Set must be zonal with at least two availability zones configured (for example, `zones = [1, 2]`). This ensures that VMs can be distributed across multiple zones for resiliency.

**Specify a SKU for the scale set**

The Virtual Machine Scale Set must have a SKU configured. The SKU determines the VM size and capabilities, and is required for creating and rebalancing VMs.

**Use a supported Compute API version**

Automatic zone balance is supported for Compute API version 2024-07-01 or higher. 

**Register the subscription with AFEC**

The subscription must be registered with the Azure Feature Exposure Control (AFEC) flag `Microsoft.Compute.AutomaticZoneRebalancing`. This registration is required to enable the automatic zone balance feature for your subscription.

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

## Enable Automatic Zone Balance on your scale set

Follow the steps below to enable Automatic Zone Balance on a new or existing virtual machine scale set.

### [REST API](#tab/rest-api)

Add the `resiliencyPolicy` property to your scale set model. Use API version 2024-07-01 or higher.

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

### [Azure CLI](#tab/CLI-1)

Provide Azure CLI instructions, once available

### [Azure PowerShell](#tab/PowerShell-1)

Provide PowerShell instructions, once available

## Next Steps
Learn more about [automatic zone balance for Virtual Machine Scale Sets](./virtual-machine-scale-sets-auto-zone-balance.md).

