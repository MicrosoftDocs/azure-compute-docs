---
title: Migrate virtual machines from availability sets to Virtual Machine Scale Sets
description: Learn how to migrate your Azure Virtual Machines from availability sets to Virtual Machine Scale Sets with Flexible orchestration for improved availability and scale.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 01/30/2026
---

# Migrate virtual machines from availability sets to Virtual Machine Scale Sets

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

This article describes how to migrate your Azure Virtual Machines (VMs) from availability sets to Virtual Machine Scale Sets with Flexible orchestration mode.

> [!IMPORTANT]
> The availability set to Virtual Machine Scale Sets migration feature is currently in **Public Preview**. Preview features should be tested in non-production environments before migrating production workloads.

## Why migrate to Virtual Machine Scale Sets?

Virtual Machine Scale Sets with Flexible orchestration provide several advantages over availability sets:

| Feature | Virtual Machine Scale Sets (Flexible) | Availability sets |
|---------|--------------------------------------|-------------------|
| **Maximum instances** | Up to 1,000 VMs | Up to 200 VMs |
| **Availability Zones support** | Yes | No |
| **Autoscaling** | Yes | No |
| **Rolling upgrades** | Yes | No |
| **Instance protection** | Yes | No |
| **Availability SLA** | 99.95% (fault domains) or 99.99% (availability zones) | 99.95% |
| **Mixed VM sizes** | Yes | Yes |
| **Azure Backup support** | Yes | Yes |

For a complete comparison, see [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes#a-comparison-of-flexible-uniform-and-availability-sets).

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the migration preview feature registered
- **Contributor** role or higher on the resource group containing the availability set
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az module** installed
- An existing **availability set** with VMs you want to migrate
- A target **Virtual Machine Scale Set** with Flexible orchestration mode (or create one during migration)

### Register the preview feature

The migration feature requires registration. Register the `MigrateToVmssFlex` preview feature for your subscription:

# [Azure CLI](#tab/cli)

```azurecli
# Set your subscription context
az account set --subscription "<subscription-name-or-id>"

# Register the migration feature
az feature register --namespace Microsoft.Compute --name MigrateToVmssFlex

# Check registration status (wait for "Registered" state)
az feature show --namespace Microsoft.Compute --name MigrateToVmssFlex --query properties.state -o tsv
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
# Set your subscription context
Set-AzContext -Subscription "<subscription-name-or-id>"

# Register the migration feature
Register-AzProviderFeature -FeatureName MigrateToVmssFlex -ProviderNamespace Microsoft.Compute

# Check registration status (wait for "Registered" state)
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

> [!NOTE]
> Feature registration may take several minutes to complete. Wait until the state shows **Registered** before proceeding.

## Migration overview

The migration process consists of the following phases:

1. **Create target Virtual Machine Scale Set** - Create a new scale set with Flexible orchestration mode (if you don't have one)
2.* *Validate migration** - Verify the availability set can be migrated
3. **Start migration** - Put the availability set into migration mode
4. **Migrate VMs** - Move each VM individually to the scale set
5. **Start VMs** - Power on the migrated VMs
6. **Clean up** - Delete the empty availability set

> [!IMPORTANT]
> - VMs are deallocated during migration. Plan for downtime accordingly.
> - Once migration starts, the availability set is locked. No other changes can be made until migration completes or is canceled.
> - Migration is a one-way operation. You cannot migrate back to availability sets after completion.

## Supported configurations

### Migration paths

| Source | Target | Description |
|--------|--------|-------------|
| Availability set | Regional Virtual Machine Scale Set | VMs distributed across fault domains within the region |
| Availability set | Zonal Virtual Machine Scale Set | VMs placed in specific availability zones (1, 2, or 3) |

### Limitations

The following configurations aren't supported for migration:

- VMs with **Basic public IP addresses** - Upgrade to Standard SKU before migration
- VMs behind a **Basic Load Balancer** - Upgrade to Standard Load Balancer before migration
- VMs with **unmanaged disks** - Convert to managed disks before migration
- Availability sets with VMs in different resource groups

### Supported disk types

All managed disk types are supported and preserved during migration:

- Standard HDD (Standard_LRS)
- Standard SSD (StandardSSD_LRS)
- Premium SSD (Premium_LRS)
- Premium SSD v2 (PremiumV2_LRS)
- Ultra Disk (UltraSSD_LRS)

## Step 1: Create a target Virtual Machine Scale Set

If you don't have an existing scale set, create one as the migration target. The scale set must use **Flexible orchestration mode**.

For detailed instructions on creating a scale set, see [Create virtual machines in a scale set using Azure portal](/azure/virtual-machine-scale-sets/flexible-virtual-machine-scale-sets-portal).

When creating the scale set for migration:

- Select **Flexible** orchestration mode
- For **zonal deployment** (recommended for highest availability), select one or more availability zones (1, 2, 3)
- For **regional deployment**, don't select any availability zones
- Set **Instance count** to **0** to create an empty scale set for migration

## Step 2: Validate migration

Before starting migration, validate that your availability set can be migrated to the target scale set.

# [Azure CLI](#tab/cli)

```azurecli
az vm availability-set validate-migration-to-vmss \
  --resource-group "<resource-group-name>" \
  --name "<availability-set-name>" \
  --vmss-flexible-id "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scale-set-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vmss = Get-AzVmss -ResourceGroupName "<resource-group-name>" -VMScaleSetName "<scale-set-name>"
Invoke-AzAvailabilitySetValidateMigrationToVmss `
    -ResourceGroupName "<resource-group-name>" `
    -AvailabilitySetName "<availability-set-name>" `
    -VirtualMachineScaleSetId $vmss.Id
```

# [REST API](#tab/rest)
```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets/{availabilitySetName}/validateMigrationToVirtualMachineScaleSet?api-version=2024-11-01

{
  "virtualMachineScaleSetFlexible": {
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}"
  }
}
```

---

A successful validation returns HTTP status code **200**. If validation fails, review the error message for required actions.

## Step 3: Start migration

After successful validation, start the migration process. This action puts the availability set into migration mode.

# [Azure CLI](#tab/cli)

```azurecli
az vm availability-set start-migration-to-vmss \
  --resource-group "<resource-group-name>" \
  --name "<availability-set-name>" \
  --vmss-flexible-id "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scale-set-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vmss = Get-AzVmss -ResourceGroupName "<resource-group-name>" -VMScaleSetName "<scale-set-name>"
Start-AzAvailabilitySetMigrationToVmss `
    -ResourceGroupName "<resource-group-name>" `
    -AvailabilitySetName "<availability-set-name>" `
    -VirtualMachineScaleSetId $vmss.Id
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets/{availabilitySetName}/startMigrationToVirtualMachineScaleSet?api-version=2024-11-01

{
  "virtualMachineScaleSetFlexible": {
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}"
  }
}
```

---

> [!CAUTION]
> After starting migration, the availability set is locked. You can only migrate VMs, complete migration, or cancel migration until the process finishes.

## Step 4: Migrate individual VMs

Migrate each VM from the availability set to the scale set. You can choose between **regional** or **zonal** migration for each VM.

### Regional migration

For regional migration, VMs are distributed across fault domains automatically:

# [Azure CLI](#tab/cli)

```azurecli
az vm migrate-to-vmss \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Move-AzVMToVmss `
    -ResourceGroupName "<resource-group-name>" `
    -VMName "<vm-name>"
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/migrateToVirtualMachineScaleSet?api-version=2024-11-01

```

---

### Zonal migration

For zonal migration, specify the target availability zone:

# [Azure CLI](#tab/cli)

```azurecli
az vm migrate-to-vmss \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>" \
  --target-zone "1"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Move-AzVMToVmss `
    -ResourceGroupName "<resource-group-name>" `
    -VMName "<vm-name>" `
    -TargetZone "1"

```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/migrateToVirtualMachineScaleSet?api-version=2024-11-01

{
  "targetZone": "1"
}
```

---

### Optional: Change VM size during migration

You can optionally change the VM size during zonal migration:

```json
{
  "targetZone": "1",
  "targetVMSize": "Standard_D4s_v5"
}

```

> [!TIP]
> To distribute VMs across availability zones for maximum resilience, assign different zones to different VMs. For example, migrate VM1 to Zone 1, VM2 to Zone 2, and VM3 to Zone 3.

## Step 5: Start migrated VMs

After migration, VMs are in a **Stopped (deallocated)** state. Start each VM to make it available:

# [Azure CLI](#tab/cli)

```azurecli
az vm start --resource-group "<resource-group-name>" --name "<vm-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Start-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/start?api-version=2024-11-01
```

---

> [!NOTE]
> You can start VMs immediately after they reach the **Stopped (deallocated)** state, even if background data migration is still in progress. The data migration continues transparently.

## Step 6: Clean up the empty availability set

After all VMs are migrated and verified, delete the empty availability set:

# [Azure CLI](#tab/cli)

```azurecli
az vm availability-set delete \
  --resource-group "<resource-group-name>" \
  --name "<availability-set-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Remove-AzAvailabilitySet `
    -ResourceGroupName "<resource-group-name>" `
    -Name "<availability-set-name>" `
    -Force
```

# [REST API](#tab/rest)

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets/{availabilitySetName}?api-version=2024-11-01
```

---

## Cancel migration

If you need to cancel an in-progress migration before all VMs are migrated:

# [Azure CLI](#tab/cli)

```azurecli
az vm availability-set cancel-migration-to-vmss \
  --resource-group "<resource-group-name>" \
  --name "<availability-set-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Stop-AzAvailabilitySetMigrationToVmss `
    -ResourceGroupName "<resource-group-name>" `
    -AvailabilitySetName "<availability-set-name>"
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/availabilitySets/{availabilitySetName}/cancelMigrationToVirtualMachineScaleSet?api-version=2024-11-01
```

---

> [!WARNING]
> VMs that have already been migrated remain in the scale set. Only VMs that haven't yet been migrated stay in the availability set.

## Troubleshooting

### Feature not registered

**Error:**`The subscription is not registered to use feature 'MigrateToVmssFlex'`

**Solution:** Register the preview feature and wait for registration to complete. See [Register the preview feature](#register-the-preview-feature).

### VM size not available in zone

**Error:**`The requested VM size is not available in the specified location/zone`

**Solution:** Check available VM sizes in the target zone:

```azurecli
az vm list-skus --location "<region>" --zone "<zone>" --resource-type virtualMachines --output table
```

### Availability set already in migration

**Error:**`The availability set is already in migration mode`

**Solution:** Continue with VM migration steps, or cancel the migration if you need to make changes.

### Insufficient permissions

**Error:**`The client does not have authorization to perform action`

**Solution:** Ensure you have **Contributor** role or higher on the resource group.

### Basic public IP or Load Balancer

**Error:**`VMs with Basic public IP addresses or Basic Load Balancers are not supported`

**Solution:** Upgrade to Standard SKU resources before migration:

- [Upgrade a public IP address](/azure/virtual-network/ip-services/public-ip-upgrade-portal)
- [Upgrade a Basic Load Balancer](/azure/load-balancer/upgrade-basic-standard-with-powershell)

## Next steps

- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
- [Flexible orchestration mode for Virtual Machine Scale Sets](/azure/virtual-machines/flexible-virtual-machine-scale-sets)
- [Autoscale Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
