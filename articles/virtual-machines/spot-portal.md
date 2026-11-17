---
title: Use the portal to deploy Azure Spot Virtual Machines
description: How to use the Portal to deploy Spot Virtual Machines
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: azure-spot-vm
ms.topic: how-to
ms.date: 11/06/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud administrator, I want to deploy Azure Spot Virtual Machines through the portal so that I can utilize cost-effective solutions for my interruptible workloads while managing pricing and eviction strategies effectively.
---

# Deploy Azure Spot Virtual Machines using the Azure portal

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale sets

By using [Azure Spot Virtual Machines](spot-vms.md), you can take advantage of unused capacity at a significant cost savings. At any point when Azure needs the capacity back, the Azure infrastructure evicts Azure Spot Virtual Machines. Therefore, Azure Spot Virtual Machines are great for workloads that can handle interruptions like batch processing jobs, dev/test environments, large compute workloads, and more. [Spot Plus (Public Preview)](spot-plus-public-preview-azure-compute-technical.md) is an enhanced tier of Azure Spot Virtual Machines for interruption-tolerant workloads. It's designed to deliver better runtimes and improved eviction and rebalance signals, while remaining evictable when Azure needs the capacity.

Pricing for Azure Spot Virtual Machines is variable, based on region and SKU. For more information, see VM pricing for [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) and [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). For more information about setting the max price, see [Azure Spot Virtual Machines - Pricing](spot-vms.md#pricing).

You have option to set a max price you are willing to pay, per hour, for the VM. The max price for an Azure Spot Virtual Machine can be set in US dollars (USD), using up to 5 decimal places. For example, the value `0.05701`would be a max price of $0.05701 USD per hour. If you set the max price to be `-1`, the VM won't be evicted based on price. The price for the VM will be the current price for spot or the price for a standard VM, which ever is less, as long as there is capacity and quota available.

When the VM is evicted, you have the option to either delete the VM and the underlying disk or deallocate the VM so that it can be restarted later.

## Create the VM

When you are deploying a VM, you can choose to use an Azure spot instance.

On the **Basics** tab, in the **Instance details** section, select **Run with Azure Spot discount** to choose a Spot variant.

:::image type="content" source="media/spot-portal/spot-plus-pricing-tier-selection.png" alt-text="Screenshot of the Basics tab with the Run with Azure Spot discount option.":::

After you choose the Spot variant, the section expands and you can choose your [eviction type and eviction policy](spot-vms.md#eviction-policy). 

:::image type="content" source="media/spot-portal/spot-plus-eviction-settings.png" alt-text="Screenshot of the expanded Spot configuration with eviction type and eviction policy options.":::

Select **View pricing history and compare prices in nearby regions** to compare the pricing and eviction rates with other similar regions. Currently, this feature is available for **Spot** only.

In this example, the Canada Central region is less expensive and has a lower eviction rate than the East US region.

:::image type="content" source="./media/spot-portal/pricing.png" alt-text="Screenshot of the region options with the difference in pricing and eviction rates.":::

You can change the region by selecting the choice that works the best for you and then selecting **OK**.

## Simulate an eviction

You can [simulate an eviction](/rest/api/compute/virtualmachines/simulateeviction) of an Azure Spot Virtual Machine, to testing how well your application will respond to a sudden eviction. 

Replace the following with your information: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

`Response Code: 204` means the simulated eviction was successful. 

## Next steps

You can also create Azure Spot Virtual Machines using [PowerShell](./windows/spot-powershell.md), [CLI](./linux/spot-cli.md), or a [template](./linux/spot-template.md).
