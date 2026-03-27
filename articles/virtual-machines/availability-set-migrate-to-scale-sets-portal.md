---
title: Migrate availability sets to Virtual Machine Scale Sets using the Azure portal
description: Learn how to migrate your Azure Virtual Machines from availability sets to Virtual Machine Scale Sets with Flexible orchestration using the Azure portal.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 03/27/2026
---
# Migrate availability sets to Virtual Machine Scale Sets using the Azure portal

This article walks through migrating your Azure Virtual Machines (VMs) from an availability set to a Virtual Machine Scale Set with Flexible orchestration using the Azure portal. The portal provides a guided wizard that handles validation, migration, and VM zone assignment in a streamlined experience.

> [!IMPORTANT]
> The availability set to Virtual Machine Scale Sets migration feature is currently in **preview**. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

## Prerequisites

Before you begin, ensure you have the following:

- **Azure subscription** with the `MigrateToVmssFlex` preview feature registered. For registration steps, see [Register the preview feature](availability-set-migrate-to-scale-sets.md#register-the-preview-feature).
- **Contributor** role or higher on the resource group containing the availability set.
- An existing **availability set** with VMs you want to migrate.
- VMs in the availability set must use **managed disks**, **Standard SKU public IPs**, and **Standard Load Balancers** (if applicable).

For a full list of supported configurations and limitations, see [Supported configurations](availability-set-migrate-to-scale-sets.md#supported-configurations).

## Migration overview

The portal wizard consolidates the migration into two steps:

1. **Select target VMSS** - Choose an existing scale set or quick create a new one. For zonal scale sets, assign each VM to an availability zone.
2. **Review and migrate** - Review the migration plan, then migrate all VMs at once.

After migration completes, the portal provides next steps to restart your VMs and clean up the empty availability set.

> [!IMPORTANT]
>
> - VMs are stopped and deallocated during migration. Schedule the migration during a maintenance window where downtime is acceptable.
> - Once migration starts, the availability set is locked until all VMs are migrated.

## Start the migration wizard

1. In the [Azure portal](https://portal.azure.com), navigate to the availability set you want to migrate.

1. On the availability set **Overview** page, select **Migrate to VMSS Flex** from the toolbar.

   :::image type="content" source="media/avset-migration/migrate-avsets-1.png" alt-text="Screenshot showing the availability set overview page with the Migrate to VMSS Flex button in the toolbar and a banner promoting migration.":::

   The **Migrate Availability Set VMs to VMSS** wizard opens.

## Step 1: Select target VMSS

The first page of the wizard lists all virtual machines in the availability set along with their current status and size.

> [!NOTE]
> An information banner reminds you that VMs will be stopped and deallocated during migration. You'll be able to restart them once migration completes.

### Use an existing scale set

1. Under **Select target VMSS**, choose an existing scale set from the **Select a VMSS** dropdown. Only compatible VMSS configurations are displayed.

   :::image type="content" source="media/avset-migration/migrate-avsets-2.png" alt-text="Screenshot showing step 1 of the migration wizard with the list of VMs, the Select a VMSS dropdown set to myScaleSet, and the region displayed as UK South.":::

### Quick create a new VMSS

If you don't have an existing scale set, you can create one directly from the wizard:

1. Select **Quick create a new VMSS** below the dropdown.

1. In the **VMSS quick create** pane, provide a name for the scale set and complete the administrator account settings. The subscription, resource group, location, and VM size are pre-populated from your availability set configuration.

   :::image type="content" source="media/avset-migration/migrate-avsets-3.png" alt-text="Screenshot showing the VMSS quick create pane with pre-populated project details, scale set name field, and administrator account fields.":::

   > [!NOTE]
   > Data disk contents are not copied over to the new instances. Before scaling out, ensure your subnet or load balancer backend pool has sufficient available IP addresses.

1. Select **Create a VMSS** to create the scale set and return to the wizard.

### Assign availability zones (zonal migration)

If you select a zonal VMSS (one configured with availability zones), an additional **VM allocation** section appears. Use the dropdown for each VM to assign it to a specific availability zone.

:::image type="content" source="media/avset-migration/migrate-avsets-4.png" alt-text="Screenshot showing the VM allocation section for a zonal VMSS with availability zone dropdowns for each VM, showing Zone 1, Zone 2, and Zone 3 assignments.":::

> [!TIP]
> Distribute VMs across all available zones for maximum resilience. For example, assign VM1 to Zone 1, VM2 to Zone 2, and VM3 to Zone 3.

1. Select **Next** to proceed to the review step.

## Step 2: Review and migrate

The review page displays the target VMSS details and lists each VM with its migration status.

### Regional migration

For a regional VMSS, the review page shows each VM with its current size and status:

:::image type="content" source="media/avset-migration/migrate-avsets-5.png" alt-text="Screenshot showing the review page for a regional migration with three VMs listed as Not started.":::

### Zonal migration

For a zonal VMSS, the review page also displays the assigned **Zone** for each VM:

:::image type="content" source="media/avset-migration/migrate-avsets-6.png" alt-text="Screenshot showing the review page for a zonal migration with three VMs showing their zone assignments and Not started status.":::

### Run the migration

1. Review the VM list and zone assignments (if applicable), then select **Migrate**.

1. The status for each VM changes to **Migration in progress**.

   :::image type="content" source="media/avset-migration/migrate-avsets-7.png" alt-text="Screenshot showing three VMs with Migration in progress status during a zonal migration.":::

1. When all VMs finish migrating, the status changes to **Migration completed**.

   :::image type="content" source="media/avset-migration/migrate-avsets-8.png" alt-text="Screenshot showing three VMs with Migration completed status and green checkmarks.":::

## After migration: Next steps

After migration completes, the wizard displays two actions:

:::image type="content" source="media/avset-migration/migrate-avsets-9.png" alt-text="Screenshot showing the next steps section with Restart your VMs and Delete your availability set cards.":::

### Restart your VMs

VMs are in a **Stopped (deallocated)** state after migration. To start them:

1. Select **Start VMs** to start all migrated VMs, or select **Go to VMSS instances** to start them individually from the scale set.

### Delete the empty availability set

After all VMs are migrated and verified, the availability set is empty and can be safely deleted:

1. Select **Go to the availability set** to navigate back to the availability set.

   :::image type="content" source="media/avset-migration/migrate-avsets-10.png" alt-text="Screenshot showing the empty availability set with 0 virtual machines and the Migrate to VMSS Flex button grayed out.":::

1. Select **Delete** to remove the empty availability set.

## Related content

- [Migrate availability sets to Virtual Machine Scale Sets (CLI, PowerShell, REST)](availability-set-migrate-to-scale-sets.md)
- [Convert availability sets to Virtual Machine Scale Sets](availability-set-convert-to-scale-sets.md)
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
