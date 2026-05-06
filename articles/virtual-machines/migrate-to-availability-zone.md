---
title: Migrate a regional virtual machine to an availability zone
description: Learn how to migrate your Azure Virtual Machines from a regional deployment to an availability zone.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 05/06/2026
---
# Migrate a regional virtual machine to an availability zone (Preview)

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

This article describes how to migrate an Azure Virtual Machine (VM) from a regional (non-zonal) deployment to a specific availability zone.

> [!IMPORTANT]
> The regional to zonal VM migration feature is currently in **Public Preview**. Preview features should be tested in non-production environments before migrating production workloads.

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the migration preview feature registered
- **Contributor** role or higher on the resource group containing the VM
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az module** installed
- An existing **regional (non-zonal) VM** you want to migrate
- The target availability zone must support the VM's current size in the target region

### Register the preview feature

The migration feature requires registration. Register the `RegionalToZonalVMMigrationForDeallocatedVM` preview feature for your subscription:

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

## Migration overview

The migration process consists of the following steps:

1. **Deallocate the VM** - Stop and deallocate the regional VM
2. **Update the zone assignment** - Assign the VM to a specific availability zone
3. **Start the VM** - Power on the VM in its new zone
4. **(Optional) Attach to a Virtual Machine Scale Set** - Add the zonal VM to a scale set with Flexible orchestration

> [!IMPORTANT]
>
> - The VM must be deallocated before zone assignment. Plan for downtime accordingly.
> - Data migration between disks and zones happens transparently in the background after the zone is assigned.
> - VMs can be started immediately after zone assignment completes, even while background data migration is still in progress.
> - Migration is a one-way operation. You can't migrate a zonal VM back to a regional deployment.

## Supported configurations

### Migration paths

| Source                                     | Target                                  | Description                                               |
| ------------------------------------------ | --------------------------------------- | --------------------------------------------------------- |
| Regional VM                                | Zonal VM                                | VM placed in a specific availability zone (1, 2, or 3)    |
| Regional VM                                | Zonal VM in a Virtual Machine Scale Set | VM placed in a zone and attached to a Flexible scale set  |
| Regional VM in a Proximity Placement Group | Zonal VM                                | VM moved to a zone with Proximity Placement Group removed |

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

## Step 1: Deallocate the VM

Before updating the zone assignment, the VM must be fully deallocated:

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

Verify that the VM is in the **Stopped (deallocated)** state before proceeding.

## Step 2: Update the zone assignment

Assign the VM to a specific availability zone. Choose the appropriate option based on your VM's configuration.

### Standard zone assignment

For VMs that aren't in a Proximity Placement Group:

# [Azure CLI](#tab/cli)

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --set zones='["<target-zone>"]'
```

# [Azure PowerShell](#tab/powershell)

Cast the value to `[string[]]` so PowerShell assigns it to the `Zones` property correctly:

```azurepowershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
$vm.Zones = [string[]]@("<target-zone>")
Update-AzVM -ResourceGroupName "<resource-group-name>" -VM $vm
```

# [REST API](#tab/rest)

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-11-01

{
  "name": "{vmName}",
  "location": "{location}",
  "zones": ["1"]
}
```

---

### Zone assignment with Proximity Placement Group removal

If your VM is currently in a Proximity Placement Group, you must remove the Proximity Placement Group association during zone assignment:

# [Azure CLI](#tab/cli)

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --set zones='["<target-zone>"]' \
  --ppg ""
```

# [Azure PowerShell](#tab/powershell)

To clear the proximity placement group, set `Id` on the existing object to an empty string. Setting `$vm.ProximityPlacementGroup` itself to `$null` doesn't clear the association.

```azurepowershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
$vm.Zones = [string[]]@("<target-zone>")
$vm.ProximityPlacementGroup.Id = ""
Update-AzVM -ResourceGroupName "<resource-group-name>" -VM $vm
```

# [REST API](#tab/rest)

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-11-01

{
  "name": "{vmName}",
  "location": "{location}",
  "zones": ["1"],
  "properties": {
    "proximityPlacementGroup": {
      "id": ""
    }
  }
}
```

---

A successful zone assignment returns HTTP status code **200**.

> [!TIP]
> Zone availability varies by region. To check which zones are available for your VM size in a specific region, run:
>
> ```azurecli
> az vm list-skus --location "<region>" --zone --resource-type virtualMachines --output table
> ```

## Step 3: Start the VM

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

## Step 4 (Optional): Attach the VM to a Virtual Machine Scale Set

After migrating the VM to an availability zone, you can optionally attach it to a Virtual Machine Scale Set with Flexible orchestration for enhanced management capabilities like autoscaling and rolling upgrades.

### Prerequisites for scale set attachment

- The scale set must use **Flexible** orchestration mode
- The scale set must be in the **same region** as the VM
- The scale set zone configuration must include the VM's assigned zone
- For scale sets with fault domain count of 1, `singlePlacementGroup` must be set to `false`

# [Azure CLI](#tab/cli)

A regional VM doesn't have a `virtualMachineScaleSet` property to dot-into, so use `az resource update` to PATCH the property as a JSON object on the underlying VM resource:

```azurecli
az resource update \
  --ids "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines/<vm-name>" \
  --set 'properties.virtualMachineScaleSet={"id":"/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scale-set-name>"}'
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
$vmss = Get-AzVmss -ResourceGroupName "<resource-group-name>" -VMScaleSetName "<scale-set-name>"

$vm.VirtualMachineScaleSet = New-Object Microsoft.Azure.Management.Compute.Models.SubResource $vmss.Id
Update-AzVM -ResourceGroupName "<resource-group-name>" -VM $vm
```

# [REST API](#tab/rest)

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-11-01

{
  "properties": {
    "virtualMachineScaleSet": {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}"
    }
  }
}
```

---

## Verify migration

After migration, confirm the VM is running in the correct availability zone:

# [Azure CLI](#tab/cli)

```azurecli
az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "{Name:name, Zone:zones[0], Location:location}" --output table
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
$vm = Get-AzVM -ResourceGroupName "<resource-group-name>" -Name "<vm-name>"
Write-Output "VM Name: $($vm.Name)"
Write-Output "Zone: $($vm.Zones)"
Write-Output "Location: $($vm.Location)"
```

# [REST API](#tab/rest)

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-11-01
```

Verify that the `zones` property in the response contains the target zone value.

---

## Troubleshooting

### VM size not available in zone

**Error:** `The requested VM size is not available in the specified location/zone`

**Solution:** Check available VM sizes in the target zone:

```azurecli
az vm list-skus --location "<region>" --zone "<zone>" --resource-type virtualMachines --output table
```

Consider changing the VM size before migration if the current size isn't available in the target zone.

### Proximity Placement Group conflict

**Error:** `Cannot assign zone to a VM in a Proximity Placement Group`

**Solution:** Use the zone assignment with Proximity Placement Group removal option described in [Step 2](#zone-assignment-with-proximity-placement-group-removal). This removes the Proximity Placement Group association and assigns the zone in a single operation.

### Insufficient permissions

**Error:** `The client does not have authorization to perform action`

**Solution:** Ensure you have **Contributor** role or higher on the resource group.

### Scale Set attachment fails after zone assignment

**Error:** `Cannot attach VM to the specified Virtual Machine Scale Set`

**Solution:** Verify the following:

- The scale set uses **Flexible** orchestration mode
- The scale set zone configuration includes the VM's zone
- For scale sets with fault domain count of 1, `singlePlacementGroup` is set to `false`

```azurecli
az vmss show --resource-group "<resource-group-name>" --name "<scale-set-name>" --query "{Zones:zones, OrchestrationMode:orchestrationMode, SinglePlacementGroup:singlePlacementGroup}"
```

## Next steps

- [What are Azure availability zones?](/azure/reliability/availability-zones-overview)
- [Availability zone service and regional support](/azure/reliability/availability-zones-service-support)
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
