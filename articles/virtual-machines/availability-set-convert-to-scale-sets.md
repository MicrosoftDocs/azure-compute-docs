---
title: Convert availability sets to Virtual Machine Scale Sets 
description: Learn how to convert your Azure Virtual Machines from availability sets to Virtual Machine Scale Sets with Flexible orchestration using the Convert API with zero downtime.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 03/17/2026
---
# Convert availability sets to Virtual Machine Scale Sets (preview)

This article describes how to convert an availability set and all its VMs to a Virtual Machine Scale Set with Flexible orchestration mode using the Convert API. The conversion is a single operation that requires **zero downtime**. VMs remain online throughout the process.

> [!IMPORTANT]
> The availability set to Virtual Machine Scale Sets conversion feature is currently in **preview**. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

## Choose the right migration approach

Azure provides two approaches for moving VMs from availability sets to Virtual Machine Scale Sets. Review the differences carefully before choosing an approach. The choice affects the capabilities available to your workloads after migration.

> [!CAUTION]
> The Convert API produces a scale set with a **limited subset of Flexible orchestration features** and **no availability zone support**. If you need full scale set capabilities or zone placement, use the [Migrate API](availability-set-migrate-to-scale-sets.md) instead.

| Feature                                  | Convert API                                                                             | [Migrate API](availability-set-migrate-to-scale-sets.md) |
| ---------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| **Downtime**                       | None. VMs remain online.                                                                | VMs deallocated during migration                      |
| **Migration scope**                | All VMs at once                                                                         | One VM at a time                                      |
| **scale set capabilities**         | Limited. Subset of scale set features.                                                  | Full. All sale set features available.                |
| **Availability zone support**      | No. Regional only.                                                                      | Yes. Target specific zones.                           |
| **Future zonal migration**         | Not supported                                                                           | Already zonal.                                        |
| **Change VM SKU during migration** | No                                                                                      | Yes                                                   |
| **Target scale set creation**      | Automatically created by the platform                                                   | User manually create a scale set                      |
| **Best for**                       | Zero-downtime conversion when zones aren't needed and will not be needed in the future. | Full scale set capabilities, zonal deployments        |

### When to use each approach

**Use the [Migrate API](availability-set-migrate-to-scale-sets.md)** (recommended for most scenarios) when:

- You need **availability zone** support (now or in the future)
- You want the **full Virtual Machine Scale Sets feature set**
- You want to **change VM sizes** or configurations during migration
- You can tolerate brief per-VM downtime during a maintenance window

**Use the Convert API** only when:

- You **cannot tolerate any downtime**. Not even a brief maintenance window.
- You **don't need availability zones** and won't need them in the future
- You accept the **limited feature set** of converted scale sets
- Regional fault domain distribution is sufficient for your availability requirements now and in the future

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the migration preview feature registered
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az.Compute module 11.3.0** or later
- An existing **availability set** with VMs you want to convert

### Limitations

The following configurations and scenarios aren't supported for conversion:

- Conversion isn't supported in the Azure portal. Only migrate is available in the Azure portal.
- VMs with **basic public IP addresses**. Upgrade to Standard SKU before conversion.
- VMs behind a **basic Load Balancer**. Upgrade to Standard Load Balancer before conversion.
- VMs with **unmanaged disks**. Convert to managed disks before conversion.
- **Zonal deployments**. The Convert API creates regional scale sets only. Use the [Migrate API](availability-set-migrate-to-scale-sets.md) for availability zone targeting.
- **Migration to availability zones after conversion**. Scale sets created through conversion can't be migrated to availability zones. If zonal deployment is a future requirement, use the [Migrate API](availability-set-migrate-to-scale-sets.md) instead.

### Register the preview feature

Register the `MigrateToVmssFlex` preview feature for your subscription:

# [Azure CLI](#tab/cli)

```azurecli
# Set your subscription context
az account set --subscription "<subscriptionId>"

# Register the migration feature
az feature register --namespace Microsoft.Compute --name MigrateToVmssFlex

# Check registration status
az feature show --namespace Microsoft.Compute --name MigrateToVmssFlex --query properties.state -o tsv
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
# Set your subscription context
Set-AzContext -Subscription "<subscriptionId>"

# Register the migration feature
Register-AzProviderFeature -FeatureName MigrateToVmssFlex -ProviderNamespace Microsoft.Compute

# Check registration status
Get-AzProviderFeature -FeatureName MigrateToVmssFlex -ProviderNamespace Microsoft.Compute
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/MigrateToVmssFlex/register?api-version=2021-07-01
```

Check registration status:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/MigrateToVmssFlex?api-version=2021-07-01
```

---

## Conversion overview

The Convert API performs the entire migration in a single call. After conversion, verify that all VMs are members of the new scale set.

The Convert API automatically:

- Creates a new Virtual Machine Scale Set with Flexible orchestration. You can specify a custom scale set name using the CLI or PowerShell, or the API defaults to the availability set name.
- Moves all VMs from the availability set into the scale set
- Keeps all VMs **running** throughout the process (zero downtime)
- Removes the original availability set after successful conversion

> [!CAUSION]
> Conversion is a **one-way operation**. You can't convert back to an availability set. The original availability set is deleted upon successful conversion.

## Step 1: Convert the availability set

Run the Convert API to convert your availability set and all its VMs to a Virtual Machine Scale Set in a single operation.

# [Azure CLI](#tab/cli)

```azurecli
az vm availability-set convert-to-vmss \
  --resource-group "<resource-group-name>" \
  --name "<availability-set-name>" \
  --vmss-name "<scale-set-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Convert-AzAvailabilitySet `
    -ResourceGroupName "<resource-group-name>" `
    -Name "<availability-set-name>" `
    -VirtualMachineScaleSetName "<scale-set-name>"
```

> [!NOTE]
> The `Convert-AzAvailabilitySet` cmdlet is available in **Az.Compute module version 11.3.0** or later. Update with `Update-Module Az.Compute -Force` if needed.

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets/{availabilitySetName}/convertToVirtualMachineScaleSet?api-version=2025-04-01

{}
```

> [!NOTE]
> The request body must be an empty JSON object `{}`. When no scale name is specified, the API creates the scale set using the availability set name.

---

A successful conversion returns HTTP status code **200** (synchronous completion) or **202 Accepted** (asynchronous operation started). If the conversion fails, review the error message for required actions.

## Step 2: Verify the conversion

After the conversion completes, verify that all VMs are now members of the new Virtual Machine Scale Set.

# [Azure CLI](#tab/cli)

```azurecli
# List VMs in the scale set and their status
az vmss list-instances \
  --resource-group "<resource-group-name>" \
  --name "<scale-set-name>" \
  --output table

# Confirm the availability set no longer exists
az vm availability-set list \
  --resource-group "<resource-group-name>" \
  --output table
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
# List VMs in the scale set and their status
Get-AzVmssVM -ResourceGroupName "<resource-group-name>" -VMScaleSetName "<scale-set-name>"

# Confirm the availability set no longer exists
Get-AzAvailabilitySet -ResourceGroupName "<resource-group-name>"
```

# [REST API](#tab/rest)

List VMs in the scale set:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{scaleSetName}/virtualMachines?api-version=2024-11-01
```

List availability sets in the resource group:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets?api-version=2024-11-01
```

---

## Troubleshooting

### Feature not registered

**Error:** `The subscription is not registered to use feature 'MigrateToVmssFlex'`

**Solution:** Register the preview feature and wait for registration to complete. See [Register the preview feature](#register-the-preview-feature).

### Scale set with same name already exists

**Error:** `A virtual machine scale set with the specified name already exists`

**Solution:** A scale set with the target name already exists in the resource group. Rename or delete the existing scale set, or use the [Migrate API](availability-set-migrate-to-scale-sets.md), which lets you specify a different target.

### Basic public IP or Load Balancer

**Error:** `VMs with Basic public IP addresses or Basic Load Balancers are not supported`

**Solution:** Upgrade to Standard SKU resources before conversion:

- [Upgrade a public IP address](/azure/virtual-network/ip-services/public-ip-upgrade-portal)
- [Upgrade a Basic Load Balancer](/azure/load-balancer/upgrade-basic-standard-with-powershell)

### Unmanaged disks

**Error:** `VMs with unmanaged disks are not supported for conversion`

**Solution:** Convert unmanaged disks to managed disks before running the Convert API:

- [Convert a VM from unmanaged disks to managed disks](/azure/virtual-machines/windows/convert-unmanaged-to-managed-disks)

### Conversion failed for individual VMs

**Error:** `One or more VMs failed to convert`

**Solution:** Check the Activity Log for the resource group to identify which VMs failed and the specific error. Resolve the issues and retry the conversion.

## Next steps

- [Migrate availability sets to Virtual Machine Scale Sets](availability-set-migrate-to-scale-sets.md). Alternative approach with per-VM control and availability zone support.
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
- [Flexible orchestration mode for Virtual Machine Scale Sets](/azure/virtual-machines/flexible-virtual-machine-scale-sets)
- [Autoscale Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
