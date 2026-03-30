---
title: Migrate availability sets to Virtual Machine Scale Sets
description: Learn how to migrate your Azure Virtual Machines from availability sets to Virtual Machine Scale Sets with Flexible orchestration using the Azure portal, Azure CLI, Azure PowerShell, or REST API.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 03/27/2026
---
# Migrate availability sets to Virtual Machine Scale Sets

This article describes how to migrate your Azure Virtual Machines (VMs) from availability sets to Virtual Machine Scale Sets with Flexible orchestration mode. 

> [!IMPORTANT]
> The availability set to Virtual Machine Scale Sets migration feature is currently in **preview**. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

## Why migrate to Virtual Machine Scale Sets?

Virtual Machine Scale Sets with Flexible orchestration provide several advantages over availability sets:

| Feature                              | Virtual Machine Scale Sets (Flexible)                 | Availability sets |
| ------------------------------------ | ----------------------------------------------------- | ----------------- |
| **Maximum instances**          | Up to 1,000 VMs                                       | Up to 200 VMs     |
| **Availability Zones support** | Yes                                                   | No                |
| **Autoscaling**                | Yes                                                   | No                |
| **Rolling upgrades**           | Yes                                                   | No                |
| **Instance protection**        | Yes                                                   | No                |
| **Availability SLA**           | 99.95% (fault domains) or 99.99% (availability zones) | 99.95%            |
| **Mixed VM sizes**             | Yes                                                   | Yes               |
| **Azure Backup support**       | Yes                                                   | Yes               |

For a complete comparison, see [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes#a-comparison-of-flexible-uniform-and-availability-sets).

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the migration preview feature registered
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az module** installed
- An existing **availability set** with VMs you want to migrate

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

## Supported Migration paths

| Source           | Target                             | Description                                            |
| ---------------- | ---------------------------------- | ------------------------------------------------------ |
| Availability set | Regional Virtual Machine Scale Set | VMs distributed across fault domains within the region |
| Availability set | Zonal Virtual Machine Scale Set    | VMs placed in specific availability zones (1, 2, or 3) |

### Limitations

The following configurations aren't supported for migration:

- VMs with **Basic public IP addresses** - Upgrade to Standard SKU before migration
- VMs behind a **Basic Load Balancer** - Upgrade to Standard Load Balancer before migration
- VMs with **unmanaged disks** - Convert to managed disks before migration

## Migration overview

> [!IMPORTANT]
>
> - VMs are stopped and deallocated during migration. Schedule the migration during a maintenance window where downtime is acceptable.
> - Once migration starts, the availability set is locked. No other changes can be made until migration completes or is canceled.
> - Migration is a one-way operation. You cannot migrate back to availability sets after completion.

The **Azure portal** provides a guided experience that handles validation, scale set selection (or creation), zone assignment, and migration in a streamlined flow. When migrating using the Azure portal, you can only migrate all virtual machines at the same time. If you need to migrate a single virtual machine at a time to ensure application availability, use alternative migration methods such as CLI, PowerShell, or REST.

For **Azure CLI, Azure PowerShell, or REST API**, the migration process consists of the following phases:

1. **Create target Virtual Machine Scale Set** - Create a new scale set with Flexible orchestration mode (if you don't have one)
2. **Validate migration** - Verify the availability set can be migrated
3. **Start migration** - Put the availability set into migration mode
4. **Migrate VMs** - Move each VM individually to the scale set
5. **Start VMs** - Power on the migrated VMs
6. **Clean up** - Delete the empty availability set

## Migrate using the Azure portal

[!NOTE]

> The portal migrates **all VMs in the availability set at once**. If you need to migrate VMs one at a time to maintain application uptime during the migration, use the [Azure CLI, Azure PowerShell, or REST API](#migrate-using-azure-cli-azure-powershell-or-rest-api) instead.

### Step 1: Start the migration process

1. In the [Azure portal](https://portal.azure.com), navigate to the availability set you want to migrate.
2. On the availability set **Overview** page, select the **Migrate option** from the toolbar.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-start-migration-wizard.png" alt-text="Screenshot showing the availability set overview page with the Migrate button in the toolbar and a banner promoting migration.":::

### Step 2: Select target scale set

The first page lists all virtual machines in the availability set along with their current status and size.

#### Use an existing scale set

1. Under **Select target scale set**, choose an existing scale set from the dropdown. Only compatible scale set configurations are displayed.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-select-target-scale-set.png" alt-text="Screenshot showing step 1 of the migration experience with the list of VMs, the Select a scale set dropdown set to myScaleSet, and the region displayed as UK South.":::

#### Quick create a new scale set

If you don't have an existing scale set, you can create one directly from the migration experience:

1. Select **Quick create a new scale set** below the dropdown.
2. In the **scale set quick create** pane, provide a name for the scale set and complete the administrator account settings. The subscription, resource group, location, and VM size are prepopulated from your availability set configuration.
3. Select **Create a scale set** to create the scale set and continue with the migration.

### Step 3: Assign availability zones (zonal migration only)

If you select a zonal scale set (one configured with availability zones), an additional **VM allocation** section appears. Use the dropdown for each VM to assign it to a specific availability zone.

> [!TIP]
> Distribute VMs across all available zones for maximum resilience. For example, assign VM1 to Zone 1, VM2 to Zone 2, and VM3 to Zone 3.

:::image type="content" source="media/availability-set-migration/availability-set-migration-assign-target-zones.png" alt-text="Screenshot showing the VM allocation section for a zonal scale set with availability zone dropdowns for each VM, showing Zone 1, Zone 2, and Zone 3 assignments.":::

### Step 4: Review and migrate

1. The review page displays the target scale set details and lists each VM with its migration status.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-review-zonal.png" alt-text="Screenshot showing the review page for a zonal migration with three VMs showing their zone assignments and Not started status.":::
2. Review the VM list and zone assignments (if applicable), then select **Migrate**.
3. The status for each VM changes to **Migration in progress**.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-in-progress-zonal.png" alt-text="Screenshot showing three VMs with Migration in progress status during a zonal migration.":::
4. When all VMs finish migrating, the status changes to **Migration completed**.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-completed-zonal.png" alt-text="Screenshot showing three VMs with Migration completed status and green checkmarks.":::

### Step 5: Start VMs and clean up

1. Select **Start VMs** to start all migrated VMs, or select **Go to scale set instances** to start them individually from the scale set.

   > [!IMPORTANT]
   > After migrating your VMs in the Azure portal and navigating to your scale set instances tab, you might notice some of the VMs missing from the list. This is just a delay from the portal viewer. The VMs have been migrated. If needed, you also can view the migrated VMs in the Virtual Machine overview blade and start them from there.
   >

   :::image type="content" source="media/availability-set-migration/availability-set-migration-start-delete.png" alt-text="Screenshot showing the next steps section with Restart your VMs and Delete your availability set cards.":::
2. After all VMs are migrated and verified, the availability set is empty and can be safely deleted

   :::image type="content" source="media/availability-set-migration/availability-set-empty.png" alt-text="Screenshot showing the empty availability set with 0 virtual machines and the Migrate to scale set Flex button grayed out.":::
3. Select **Delete** to remove the empty availability set.

## Migrate using Azure CLI, Azure PowerShell, or REST API

### Step 1: Create a target Virtual Machine Scale Set

[!NOTE]

> Migration via the available SDKs provides additional control over the migration steps, such as migrating individual VMs at a time or canceling the migration. If you are okay migrating all VMs at once, you can opt to use the Azure portal instead. Otherwise, if you do need more control over the individual migration steps, continue with the below steps. 

If you don't have an existing scale set, create one as the migration target. The scale set must use **Flexible orchestration mode**.

For detailed instructions on creating a scale set, see [Create virtual machines in a scale set using Azure portal](/azure/virtual-machine-scale-sets/flexible-virtual-machine-scale-sets-portal).

When creating the scale set for migration:

- Select **Flexible** orchestration mode
- For **zonal deployment** (recommended for highest availability), select one or more availability zones (1, 2, 3)
- For **regional deployment**, don't select any availability zones
- Set **Instance count** to **0** to create an empty scale set for migration

### Step 2: Validate migration

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
Test-AzAvailabilitySetMigration `
    -ResourceGroupName "<resource-group-name>" `
    -AvailabilitySetName "<availability-set-name>" `
    -VirtualMachineScaleSetFlexibleId $vmss.Id
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

### Step 3: Start migration

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
Start-AzAvailabilitySetMigration `
    -ResourceGroupName "<resource-group-name>" `
    -AvailabilitySetName "<availability-set-name>" `
    -VirtualMachineScaleSetFlexibleId $vmss.Id
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
> After starting the migration, the availability set is locked. You can only migrate VMs, complete migration, or cancel migration until the process finishes.

### Step 4: Migrate individual VMs

Migrate each VM from the availability set to the scale set.

#### Regional migration

For regional migration, VMs are distributed across fault domains automatically:

# [Azure CLI](#tab/cli)

```azurecli
az vm migrate-to-vmss \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Move-AzVirtualMachineToVmss`
    -ResourceGroupName "<resource-group-name>" `
    -VMName "<vm-name>"
```

# [REST API](#tab/rest)

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/migrateToVirtualMachineScaleSet?api-version=2024-11-01

```

---

#### Zonal migration

For zonal migration, specify the target availability zone for each virtual machine:

# [Azure CLI](#tab/cli)

```azurecli
az vm migrate-to-vmss \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>" \
  --target-zone "1"
```

# [Azure PowerShell](#tab/powershell)

```azurepowershell
Move-AzVirtualMachineToVmss`
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

#### Optional: Change VM size during migration

You can optionally change the VM size during zonal migration:

```json
{
  "targetZone": "1",
  "targetVMSize": "Standard_D4s_v5"
}

```

> [!TIP]
> To distribute VMs across availability zones for maximum resilience, assign different zones to different VMs. For example, migrate VM1 to Zone 1, VM2 to Zone 2, and VM3 to Zone 3.

### Step 5: Start migrated VMs

After migration, VMs are in a **Stopped (deallocated)** state. They do not start automatically. You can start VMs immediately after they reach the **Stopped (deallocated)** state.

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

### Step 6: Clean up the empty availability set

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

- [Convert availability sets to Virtual Machine Scale Sets](availability-set-convert-to-scale-sets.md)
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
- [Flexible orchestration mode for Virtual Machine Scale Sets](/azure/virtual-machines/flexible-virtual-machine-scale-sets)
- [Autoscale Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
