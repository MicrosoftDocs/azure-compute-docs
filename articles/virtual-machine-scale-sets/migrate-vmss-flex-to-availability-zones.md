---
title: Migrate from a regional to a zonal Virtual Machine Scale Set
description: Learn how to migrate VMs from a regional Virtual Machine Scale Set with Flexible orchestration to a zonal scale set while preserving VM state.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 03/03/2026
---
# Migrate from a regional to a zonal Virtual Machine Scale Set (Preview)

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

This article describes how to migrate existing VMs in a regional (non-zonal) Virtual Machine Scale Set with Flexible orchestration into specific availability zones while preserving VM names, data disks, network configuration, and other stateful properties.

> [!IMPORTANT]
> Stateful regional to zonal VM migration is currently in **Public Preview**. Preview features should be tested in non-production environments before migrating production workloads. Updating a Flexible scale set to include availability zones is generally available.

## Overview

Virtual Machine Scale Sets with Flexible orchestration allow you to combine the scalability of scale sets with the flexibility of individual VMs. If your Flexible scale set was originally deployed without availability zones (regional), you can update the scale set to include zones (GA) and then migrate the existing VMs into those zones in place.

This is a **stateful migration** — each VM retains its:

- **VM name and resource ID**
- **OS disk and all data disks** (contents preserved)
- **Network interface**, private IP address, and MAC address
- **Scale set membership** (the VM stays attached to the same Flexible scale set)

The migration process involves:

1. **Update the scale set** to include availability zones in its configuration (GA)
2. **Deallocate each VM** in the scale set
3. **Assign each VM to an availability zone** (Preview)
4. **Start each VM** in its new zone

> [!IMPORTANT]
>
> - Each VM must be deallocated before zone assignment. Plan for downtime accordingly.
> - Data migration between disks and zones happens transparently in the background after the zone is assigned.
> - VMs can be started immediately after zone assignment completes, even while background data migration is still in progress.
> - Migration is a one-way operation. You can't migrate a zonal VM back to a regional deployment.

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the migration preview feature registered
- **Contributor** role or higher on the resource group containing the scale set
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az module** installed
- An existing **regional (non-zonal) Virtual Machine Scale Set** with **Flexible** orchestration mode
- The target availability zones must support the VM sizes used in the scale set

### Register the preview feature

The stateful VM zone migration requires registration of a preview feature. Updating the scale set itself to include zones doesn't require any feature registration. Register the `RegionalToZonalVMMigrationForDeallocatedVM` preview feature for your subscription:

# [Azure CLI](#tab/cli)

```azurecli
# Set your subscription context
az account set --subscription "<subscription-name-or-id>"

# Register the migration feature
az feature register --namespace Microsoft.Compute --name RegionalToZonalVMMigrationForDeallocatedVM

# Check registration status (wait for "Registered" state)
az feature show --namespace Microsoft.Compute --name RegionalToZonalVMMigrationForDeallocatedVM --query properties.state -o tsv
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
# Set your subscription context
Set-AzContext -Subscription "<subscription-name-or-id>"

# Register the migration feature
Register-AzProviderFeature -FeatureName RegionalToZonalVMMigrationForDeallocatedVM -ProviderNamespace Microsoft.Compute

# Check registration status (wait for "Registered" state)
Get-AzProviderFeature -FeatureName RegionalToZonalVMMigrationForDeallocatedVM -ProviderNamespace Microsoft.Compute
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/RegionalToZonalVMMigrationForDeallocatedVM/register?api-version=2021-07-01
```

Check registration status:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/RegionalToZonalVMMigrationForDeallocatedVM?api-version=2021-07-01
```

---

> [!NOTE]
> Feature registration may take several minutes to complete. Wait until the state shows **Registered** before proceeding.

## Supported configurations

### Migration paths

| Source | Target | Description |
| ------ | ------ | ----------- |
| Regional VM in a Flexible scale set | Zonal VM in a Flexible scale set | VM placed in a specific availability zone (1, 2, or 3) while preserving VM name, disks, network config, and scale set membership |

### Limitations

The following configurations aren't supported for migration:

- VMs with **Basic public IP addresses** - Upgrade to Standard SKU before migration
- VMs behind a **Basic Load Balancer** - Upgrade to Standard Load Balancer before migration
- VMs with **unmanaged disks** - Convert to managed disks before migration

### Supported disk types

All managed disk types are supported and preserved during migration:

- Standard HDD (Standard_LRS)
- Standard SSD (StandardSSD_LRS)
- Premium SSD (Premium_LRS)
- Premium SSD v2 (PremiumV2_LRS)
- Ultra Disk (UltraSSD_LRS)

## Step 1: Update the scale set to include availability zones

Before migrating individual VMs, update the Flexible scale set to include the target availability zones in its configuration. This is a generally available operation and doesn't require preview feature registration. This step doesn't affect running VMs — it updates the scale set model so that VMs can be assigned to zones.

# [Azure CLI](#tab/cli)

```azurecli
az vmss update \
  --resource-group "<resource-group-name>" \
  --name "<scale-set-name>" \
  --set zones='["1","2","3"]'
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vmss = Get-AzVmss -ResourceGroupName "<resource-group-name>" -VMScaleSetName "<scale-set-name>"
$vmss.Zones = @("1", "2", "3")
Update-AzVmss -ResourceGroupName "<resource-group-name>" -Name "<scale-set-name>" -VirtualMachineScaleSet $vmss
```

# [REST API](#tab/rest)

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}?api-version=2024-11-01

{
  "properties": {},
  "zones": ["1", "2", "3"]
}
```

---

> [!NOTE]
> You can specify any combination of zones (for example, `["1"]` or `["1", "3"]`), depending on your requirements. Specifying all three zones (`["1", "2", "3"]`) provides the highest resiliency.

## Step 2: Deallocate the VM

Before updating the zone assignment, each VM must be fully deallocated. Repeat this step and the following steps for each VM in the scale set.

> [!TIP]
> To minimize downtime for applications with multiple VMs, consider migrating VMs in batches. Migrate a subset of VMs, validate application health, and then proceed with the next batch.

# [Azure CLI](#tab/cli)

```azurecli
az vm deallocate --resource-group "<resource-group-name>" --name "<vm-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Stop-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>" -Force
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/deallocate?api-version=2024-11-01
```

---

## Step 3: Update the zone assignment

Assign the VM to a specific availability zone:

# [Azure CLI](#tab/cli)

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --set zones='["<target-zone>"]'
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
$vm.Zones = @("<target-zone>")
Update-AzVM -ResourceGroupName "<resource-group-name>" -VM $vm
```

# [REST API](#tab/rest)

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-11-01

{
  "name": "{vmName}",
  "location": "{location}",
  "zones": ["<target-zone>"]
}
```

---

## Step 4: Start the VM

After zone assignment, the VM is in a **Stopped (deallocated)** state. Start the VM:

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
> You can start the VM immediately after zone assignment completes, even if background data migration is still in progress. The data migration continues transparently.

Repeat Steps 2 through 4 for each VM in the scale set that you want to migrate.

## Troubleshooting

### Scale set zone update fails

**Error:** `Cannot update zones on the Virtual Machine Scale Set`

**Solution:** Ensure the scale set uses **Flexible** orchestration mode and that the target zones are available in the scale set's region. Check that none of the VMs in the scale set are currently undergoing another operation.

### VM size not available in zone

**Error:** `The requested VM size is not available in the specified location/zone`

**Solution:** Check available VM sizes in the target zone:

```azurecli
az vm list-skus --location "<region>" --zone "<zone>" --resource-type virtualMachines --output table
```

Consider changing the VM size before migration if the current size isn't available in the target zone.

### Insufficient permissions

**Error:** `The client does not have authorization to perform action`

**Solution:** Ensure you have **Contributor** role or higher on the resource group.

### VM zone assignment fails while in scale set

**Error:** `Cannot assign zone to a VM in the specified Virtual Machine Scale Set`

**Solution:** Verify the following:

- The scale set has been updated to include the target zone (Step 1)
- The VM is fully deallocated
- The VM's size is available in the target zone
- No other operations are running on the scale set

```azurecli
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<scale-set-name>" \
  --query "{Zones:zones, OrchestrationMode:orchestrationMode}" \
  --output table
```

## Next steps

- [What are Azure availability zones?](/azure/reliability/availability-zones-overview)
- [Availability zone service and regional support](/azure/reliability/availability-zones-service-support)
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
- [Migrate a regional virtual machine to an availability zone](/azure/virtual-machines/migrate-to-availability-zone)
