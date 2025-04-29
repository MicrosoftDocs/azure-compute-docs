---
title: Enable Automatic Zone Balance on Virtual Machine Scale Sets
description: Guide to enable the Automatic Zone Balance feature for your VMSS.
author: hilaryw29
ms.author: hilarywang
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.subservice: resiliency
ms.date: 04/24/2025
---
# Enable Automatic Zone Balance on Virtual Machine Scale Sets

This guide provides instructions to enable Automatic Zone Balance on your Azure Virtual Machine Scale Sets (VMSS).

## Prerequisites

Before enabling Automatic Zone Balance, ensure the following requirements are met:

- **AFEC Registration:** Your subscription must be registered with the Azure Feature Exposure Control (AFEC) flag `Microsoft.Compute.AutomaticZoneRebalancing`.
- **API Version:** Use Azure Compute API version `2024-07-01` or higher.
- **Zonal VMSS:** Your scale set must be deployed across at least two availability zones (e.g., `zones = [1, 2]`).
- **SKU:** The VMSS must have a SKU specified.
- **Application Health Monitoring:** Application health signals must be enabled using either the Application Health Extension or a Load Balancer Health Probe.

## Register the AFEC Feature

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

## Enable Automatic Zone Balance via REST API

Add or update the `resiliencyPolicy` property in your VMSS resource template:

```json
"resiliencyPolicy": {
  "AutomaticZoneRebalancingPolicy": {
    "Enabled": true,
    "RebalanceStrategy": "Recreate",
    "RebalanceBehavior": "CreateBeforeDelete"
  }
}
```

### Example: Update VMSS with Automatic Zone Balance (REST API)

Send a PATCH request to your VMSS resource:

```
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2024-07-01
```

**Request body:**

```json
{
  "properties": {
    "resiliencyPolicy": {
      "AutomaticZoneRebalancingPolicy": {
        "Enabled": true,
        "RebalanceStrategy": "Recreate",
        "RebalanceBehavior": "CreateBeforeDelete"
      }
    }
  }
}
```

## Next Steps
- Monitor your VMSS for zone balance and health signals.
- For more details, see [Automatic zone balance for Azure Virtual Machine Scale Sets](./virtual-machine-scale-sets-auto-zone-balance.md).

