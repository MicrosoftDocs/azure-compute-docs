---
title: Enable Automatic Zone Balance on Virtual Machine Scale Sets (Preview)
description: Guide to enable the Automatic Zone Balance feature for your virtual machine scale set.
author: hilaryw29
ms.author: hilarywang
ms.topic: article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 12/10/2025
---
# Enable Automatic Zone Balance on Virtual Machine Scale Sets (Preview)

This guide provides instructions to enable Automatic Zone Balance on your Virtual Machine Scale Sets.

> [!IMPORTANT]
> Automatic Zone Balance for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature can change before general availability (GA).

## Prerequisites

Before enabling automatic zone balance on your scale set, ensure the following prerequisites are met:

**Enable application health monitoring for the scale set**

The scale set must have application health monitoring enabled to use automatic zone balance. Health monitoring can be done using either [Application Health Extension](./virtual-machine-scale-sets-health-extension.md) or [Load Balancer Health Probes](/azure/load-balancer/load-balancer-custom-probe-overview), where only one can be enabled at a time. 

The application health status is used to ensure that new virtual machines (VM) created during the rebalance process are successful and "healthy," before the original VMs are removed. 

**Configure the scale set with at least two availability zones**

The Virtual Machine Scale Set must be zone-spanning with at least two [availability zones](./virtual-machine-scale-sets-use-availability-zones.md) configured (for example, `zones = [1, 2]`). All VMs in the scale set must be assigned to an availability zone. Scale sets that contain regional (non-zonal) VMs don't qualify for automatic zone balance.  This ensures that the VMs can be distributed across multiple zones for resiliency.

**Use best-effort zone balancing mode**

The scale set must use best-effort zone balancing mode (`zoneBalance = false`). Automatic zone balance can't be enabled on scale sets with strict zone balancing (`zoneBalance = true`). For more information on zone balance modes, see [Zone balance modes](./virtual-machine-scale-sets-zone-balancing.md#zone-balance-modes).

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

Follow the steps to enable Automatic Zone Balance on a new or existing virtual machine scale set.

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
1. Select the checkbox for **Automatic zone balancing**.
:::image type="content" source="media/virtual-machine-scale-sets-auto-zone-balance/enable-rebalance-create-vmss.png" alt-text="Screenshot of the Health tab in Azure portal showing application health monitoring enabled and automatic zone balancing checkbox selected under Resiliency section.":::

1. Complete additional setup for your scale set. When ready, select **Review and Create**.
1. Select **Create**.

**To enable automatic zone balance on an existing scale set**

1. Navigate to your Virtual Machine Scale Set.
1. Under **Availability + scale** dropdown, select **Availability**. 
1. Ensure that at least 2 zones are selected in the **Availability zone** dropdown.
1. Under **Operations** dropdown, select **Health and repair**. 
1. Ensure that **[application health monitoring](./virtual-machine-scale-sets-health-extension.md)** is enabled.
1. Under **Resiliency** section, select the checkbox for **Automatic zone balancing**.
1. Select **Save**.

:::image type="content" source="media/virtual-machine-scale-sets-auto-zone-balance/enable-rebalance-existing-vmss.png" alt-text="A screenshot showing how to enable automatic zone balance on an existing Virtual Machine Scale Set in the portal.":::

### [Azure CLI](#tab/CLI-2)

The following example shows how to configure automatic zone balance and Application Health extension when creating a new Virtual Machine Scale Set through Azure CLI. For more information, see [az vmss create](/cli/azure/vmss#az-vmss-create) and [az vmss update](/cli/azure/vmss#az-vmss-update).

```azurecli
# Step 1: Create VMSS with zones configured
az vmss create \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --location eastus \
    --vm-sku Standard_D2s_v3 \
    --instance-count 3 \
    --zones 1 2 3 \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys

# Step 2: Add Application Health extension (required for automatic zone balance)
az vmss extension set \
    --resource-group myResourceGroup \
    --vmss-name myVMScaleSet \
    --name ApplicationHealthLinux \
    --publisher Microsoft.ManagedServices \
    --version 2.0 \
    --settings '{"protocol": "http", "port": 80, "requestPath": "/healthEndpoint"}'

# Step 3: Upgrade instances to install the extension
az vmss update-instances \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --instance-ids "*"

# Step 4: Enable automatic zone balance
az vmss update \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --enable-automatic-zone-balancing true \
    --automatic-zone-balancing-behavior CreateBeforeDelete \
    --automatic-zone-balancing-strategy Recreate
```

### [Azure PowerShell](#tab/PowerShell-2)

The following example shows how to configure automatic zone balance and Application Health extension when creating a new Virtual Machine Scale Set through Azure PowerShell. You must also configure an OS, storage, and networking profile to complete the deployment. For more information, see [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig#description).

```azurepowershell
# Set variables
$resourceGroupName = "myResourceGroup"
$vmssName = "myVMScaleSet"
$location = "EastUS"

# Create VMSS config with automatic zone balance parameters
$vmssConfig = New-AzVmssConfig `
    -Location $location `
    -SkuCapacity 3 `
    -SkuName "Standard_D2s_v3" `
    -Zone @("1", "2", "3") `
    -EnableAutomaticZoneRebalance `
    -AutomaticZoneRebalanceBehavior "CreateBeforeDelete" `
    -AutomaticZoneRebalanceStrategy "Recreate"
    
# Add Application Health extension (required for automatic zone balance)
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"; "gracePeriod" = 600}
$vmssConfig = Add-AzVmssExtension `
    -VirtualMachineScaleSet $vmssConfig `
    -Name "HealthExtension" `
    -Publisher "Microsoft.ManagedServices" `
    -Type "ApplicationHealthWindows" `
    -TypeHandlerVersion "2.0" `
    -AutoUpgradeMinorVersion $true `
    -Setting $publicConfig

# Configure OS, storage, and networking profiles (not shown) before deployment.

# Create the VMSS
New-AzVmss -ResourceGroupName $resourceGroupName -Name $vmssName -VirtualMachineScaleSet $vmssConfig
```

---

## Enable Automatic Zone Balance Without Instance Repairs

By default, enabling automatic zone balance also enables [automatic instance repairs](./virtual-machine-scale-sets-automatic-instance-repairs.md) to provide high availability for your scale set. If you prefer to use automatic zone balance without automatic instance repairs, you can disable instance repairs separately.

> [!NOTE]
> Disabling automatic instance repairs means your scale set will only benefit from zone-level resiliency, not instance-level health monitoring, and repair.

### [Azure CLI](#tab/cli)

The following example shows how to disable automatic instance repairs while enabling automatic zone balance when creating a new Virtual Machine Scale Set. For more information, see [az vmss create](/cli/azure/vmss#az-vmss-create) and [az vmss update](/cli/azure/vmss#az-vmss-update).

```azurecli
# Step 1: Create VMSS with zones configured
az vmss create \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --location eastus \
    --vm-sku Standard_D2s_v3 \
    --instance-count 3 \
    --zones 1 2 3 \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys

# Step 2: Add Application Health extension (required for automatic zone balance)
az vmss extension set \
    --resource-group myResourceGroup \
    --vmss-name myVMScaleSet \
    --name ApplicationHealthLinux \
    --publisher Microsoft.ManagedServices \
    --version 2.0 \
    --settings '{"protocol": "http", "port": 80, "requestPath": "/healthEndpoint"}'

# Step 3: Upgrade instances to install the extension
az vmss update-instances \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --instance-ids "*"

# Step 4: Enable automatic zone balance and disable automatic instance repairs
az vmss update \
    --resource-group myResourceGroup \
    --name myVMScaleSet \
    --enable-automatic-zone-balancing true \
    --automatic-zone-balancing-behavior CreateBeforeDelete \
    --automatic-zone-balancing-strategy Recreate \
    --enable-auto-repairs false
```

### [Azure PowerShell](#tab/powershell)

The following example shows how to disable automatic instance repairs while enabling automatic zone balance when creating a new Virtual Machine Scale Set. You must also configure an OS, storage, and networking profile to complete the deployment. For more information, see [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig#description).

```azurepowershell
# Set variables
$resourceGroupName = "myResourceGroup"
$vmssName = "myVMScaleSet"
$location = "EastUS"

# Create VMSS config with automatic zone balance parameters and disable automatic repairs
$vmssConfig = New-AzVmssConfig `
    -Location $location `
    -SkuCapacity 3 `
    -SkuName "Standard_D2s_v3" `
    -Zone @("1", "2", "3") `
    -EnableAutomaticZoneRebalance `
    -AutomaticZoneRebalanceBehavior "CreateBeforeDelete" `
    -AutomaticZoneRebalanceStrategy "Recreate" `
    -EnableAutomaticRepair $false
    
# Add Application Health extension (required for automatic zone balance)
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"; "gracePeriod" = 600}
$vmssConfig = Add-AzVmssExtension `
    -VirtualMachineScaleSet $vmssConfig `
    -Name "HealthExtension" `
    -Publisher "Microsoft.ManagedServices" `
    -Type "ApplicationHealthWindows" `
    -TypeHandlerVersion "2.0" `
    -AutoUpgradeMinorVersion $true `
    -Setting $publicConfig

# Configure OS, storage, and networking profiles (not shown) before deployment.

# Create the VMSS
New-AzVmss -ResourceGroupName $resourceGroupName -Name $vmssName -VirtualMachineScaleSet $vmssConfig
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

### How can I monitor automatic zone balance operations on my scale set?

**View rebalancing operations in Azure Monitor activity logs:**

You can monitor automatic zone balance operations through Azure Monitor activity logs. Each rebalancing operation generates activity log entries that show when VMs are created and deleted during the rebalancing process.

1. In the Azure portal, navigate to your Virtual Machine Scale Set.
2. Select **Activity log** from the left menu.
3. Filter for operations with the name `BalanceVMsAcrossZones` or `Microsoft.Compute/virtualMachineScaleSets/rebalanceVmsAcrossZones/action`.

For more information about Azure Monitor activity logs, see [Azure Monitor activity log](/azure/azure-monitor/essentials/activity-log).

**View orchestration service state (VMSS Uniform only):**

For Virtual Machine Scale Sets with Uniform orchestration mode, you can also use the [Get Instance View API](/rest/api/compute/virtual-machine-scale-sets/get-instance-view?tabs=HTTP) to check the automatic zone balance orchestration service state:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}/instanceView?api-version=2025-04-01
```

The response includes an `orchestrationServices` array with automatic zone balance status:

```json
"orchestrationServices": [
  {
    "serviceName": "AutomaticZoneRebalancing",
    "serviceState": "Running",
    "latestOperationStatus": "InProgress / Completed",
    "lastStatusChangeTime": ""
  }
]
```

The `lastStatusChangeTime` field indicates when the status last changed, helping you track when rebalancing operations occurred.

### How do I know if automatic zone balance is enabled on my scale set?
You can check your scale set’s configuration for the `resiliencyPolicy` property. If `AutomaticZoneRebalancingPolicy.Enabled` is set to `true`, automatic zone balance is enabled.

### Why is my scale set not balanced?
Automatic zone balance only runs if the VMs in your scale set are imbalanced across availability zones and all [safety conditions](./auto-zone-balance-overview.md#safety-features) are met. For example, there must be no ongoing or recently completed operations on the scale set, and no rebalancing operation from the past 12 hours.

Rebalancing also depends on available capacity—if there isn’t enough capacity in the under-provisioned zone, automatic zone balance can’t create a new VM and the rebalance operation won’t start. 

### How often does automatic zone balance run?

Automatic zone balance is constantly checking your scale set for zone imbalance. When an imbalance is detected and there’s an opportunity to rebalance—all safety conditions are met, available capacity in under-provisioned zone;  a rebalance operation starts right away. 

If a rebalance operation already occurred in the past 12 hours, another rebalance won’t happen until that window passes. These limits help ensure that changes to your scale set are gradual and controlled.

### What happens if my subscription doesn’t have enough quota for a new VM during rebalancing?
If there isn’t enough quota to create a new VM, the rebalance operation won’t proceed. You need to increase your quota to allow rebalancing to occur.

### Can I control which VMs are excluded from rebalancing?
Yes. You can apply an [instance protection policy](./virtual-machine-scale-sets-instance-protection.md) to specific VMs to prevent them from being selected for rebalancing.

## Next Steps
- Learn more about [automatic zone balance for Virtual Machine Scale Sets](./auto-zone-balance-overview.md).
- Learn more about [zone balancing for Virtual Machine Scale Sets](./virtual-machine-scale-sets-zone-balancing.md).

