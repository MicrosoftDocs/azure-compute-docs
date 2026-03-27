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
- **Contributor** role or higher on the resource group containing the availability set
- **Azure CLI 2.72.0** or later, or **Azure PowerShell Az module** installed
- An existing **availability set** with VMs you want to migrate

### Register the preview feature

The migration feature requires registration. Register the `MigrateToVmssFlex` preview feature for your subscription:

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

> [!NOTE]
> Feature registration may take several minutes to complete. Wait until the state shows **Registered** before proceeding.

## Migration overview

The migration process consists of two steps:

1. **Select target scale set** - Choose an existing scale set or quick create a new one. For zonal scale sets, assign each VM to an availability zone.
2. **Review and migrate** - Review the migration plan, then migrate all VMs at once.

After migration completes, the portal provides next steps to restart your VMs and clean up the empty availability set.

> [!IMPORTANT]
>
> - VMs are stopped and deallocated during migration. Schedule the migration during a maintenance window where downtime is acceptable.
> - Once migration starts, the availability set is locked until all VMs are migrated.

## Step 1:Start the migration process

1. In the [Azure portal](https://portal.azure.com), navigate to the availability set you want to migrate.
2. On the availability set **Overview** page, select **Migrate option** from the toolbar.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-start-migration-wizard.png" alt-text="Screenshot showing the availability set overview page with the Migrate button in the toolbar and a banner promoting migration.":::

## Step 2: Select target scale set

The first page of the wizard lists all virtual machines in the availability set along with their current status and size.

> [!NOTE]
> An information banner reminds you that VMs will be stopped and deallocated during migration. You'll be able to restart them once migration completes.

### Use an existing scale set

1. Under **Select target scale set**, choose an existing scale set from the dropdown. Only compatible scale set configurations are displayed.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-select-target-scale-set.png" alt-text="Screenshot showing step 1 of the migration wizard with the list of VMs, the Select a scale set dropdown set to myScaleSet, and the region displayed as UK South.":::

### Quick create a new scale set

If you don't have an existing scale set, you can create one directly from the wizard:

1. Select **Quick create a new scale set** below the dropdown.
2. In the **scale set quick create** pane, provide a name for the scale set and complete the administrator account settings. The subscription, resource group, location, and VM size are pre-populated from your availability set configuration.
3. Select **Create a scale set** to create the scale set and return to the wizard.

## Step 3: Assign availability zones (zonal migration)

If you select a zonal scale set (one configured with availability zones), an additional **VM allocation** section appears. Use the dropdown for each VM to assign it to a specific availability zone.

[!TIP]
Distribute VMs across all available zones for maximum resilience. For example, assign VM1 to Zone 1, VM2 to Zone 2, and VM3 to Zone 3.

:::image type="content" source="media/availability-set-migration/availability-set-migration-assign-target-zones.png" alt-text="Screenshot showing the VM allocation section for a zonal scale set with availability zone dropdowns for each VM, showing Zone 1, Zone 2, and Zone 3 assignments.":::

## Step 4: Review

The review page displays the target scale set details and lists each VM with its migration status.

:::image type="content" source="media/availability-set-migration/availability-set-migration-review-zonal.png" alt-text="Screenshot showing the review page for a zonal migration with three VMs showing their zone assignments and Not started status.":::

## Step 5: Run the migration

1. Review the VM list and zone assignments (if applicable), then select **Migrate**.
2. The status for each VM changes to **Migration in progress**.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-in-progress-zonal.png" alt-text="Screenshot showing three VMs with Migration in progress status during a zonal migration.":::
3. When all VMs finish migrating, the status changes to **Migration completed**.

   :::image type="content" source="media/availability-set-migration/availability-set-migration-completed-zonal.png" alt-text="Screenshot showing three VMs with Migration completed status and green checkmarks.":::

## Step 6: Start VMs

 Select **Start VMs** to start all migrated VMs, or select **Go to scale set instances** to start them individually from the scale set.

> [!IMPORTANT]
> After migrating your VMs in the Azure portal and navigating to your scale set instances tab, you might notice some of the VMs missing from the list. This is just a delay from the portal viewer, the VMs have been migrated. If needed, navigate to the Virtual Machine overview blade to view/ start/ etc. 

:::image type="content" source="media/availability-set-migration/availability-set-migration-start-delete.png" alt-text="Screenshot showing the next steps section with Restart your VMs and Delete your availability set cards.":::


## Step: 6 Delete the empty availability set

After all VMs are migrated and verified, the availability set is empty and can be safely deleted:

1. Select **Go to the availability set** to navigate back to the availability set.

   :::image type="content" source="media/availability-set-migration/availability-set-empty.png" alt-text="Screenshot showing the empty availability set with 0 virtual machines and the Migrate to scale set Flex button grayed out.":::
2. Select **Delete** to remove the empty availability set.

## What's next

- [Migrate availability sets to Virtual Machine Scale Sets (CLI, PowerShell, REST)](availability-set-migrate-to-scale-sets.md)
- [Convert availability sets to Virtual Machine Scale Sets](availability-set-convert-to-scale-sets.md)
- [What are Virtual Machine Scale Sets?](/azure/virtual-machine-scale-sets/overview)
- [Orchestration modes for Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes)
