---
title: Create a scale set that uses Azure Spot Virtual Machines 
description: Learn how to create Azure Virtual Machine Scale Sets that use Azure Spot Virtual Machines to save on costs.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.subservice: azure-spot-vm
ms.date: 04/01/2025
ms.reviewer: mimckitt
ms.custom: devx-track-azurecli, devx-track-azurepowershell

# Customer intent: As a cloud architect, I want to deploy Azure Spot Virtual Machines within scale sets, so that I can optimize costs for interruptible workloads while managing capacity fluctuations effectively.
---

# Azure Spot Virtual Machines for Virtual Machine Scale Sets 

Using [Azure Spot Virtual Machines](../virtual-machines/spot-vms.md) (VMs) on scale sets allows you to take advantage of our unused capacity at a significant cost savings. At any point in time when Azure needs the capacity back, the Azure infrastructure evicts Azure Spot Virtual Machine instances. Therefore, Azure Spot Virtual Machine instances are great for workloads that can handle interruptions like batch processing jobs, dev/test environments, large compute workloads, and more.

The amount of available capacity can vary based on size, region, time of day, and more. When deploying Azure Spot Virtual Machine instances on scale sets, Azure allocates the instance only if there's capacity available, but there's no Service Level Agreement (SLA) for these instances. An Azure Spot Virtual Machine Scale Set is deployed in a single fault domain and offers no high availability guarantees.

## Limitations

The following sizes aren't supported for Azure Spot Virtual Machines:
 - B-series
 - Promo versions of any size (like Dv2, NV, NC, H promo sizes)

Azure Spot Virtual Machine can be deployed to any region, except Microsoft Azure operated by 21Vianet.

The following [offer types](https://azure.microsoft.com/support/legal/offer-details/) are currently supported:

-	Enterprise Agreement
-	Pay-as-you-go offer code (003P)
-	Sponsored (0036P and 0136P)
- For Cloud Service Provider (CSP), see the [Partner Center](/partner-center/azure-plan-get-started) or contact your partner directly.

## Pricing

Pricing for Azure Spot Virtual Machine instances is variable, based on region and SKU. For more information, see pricing for [Linux](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/) and [Windows](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/windows/). 


With variable pricing, you have option to set a max price, in US dollars (USD), using up to five decimal places. For example, the value `0.98765`would be a max price of $0.98765 USD per hour. If you set the max price to be `-1`, the instance won't be evicted based on price. The price for the instance will be the current price for Azure Spot Virtual Machine or the price for a standard instance, which ever is less, as long as there's capacity and quota available.


## Eviction policy

When creating a scale set using Azure Spot Virtual Machines, you can set the eviction policy to `Deallocate` (default) or `Delete`. 

The `Deallocate` policy moves your evicted instances to the stopped-deallocated state allowing you to redeploy evicted instances. However, there's no guarantee that the allocation will succeed. The deallocated VMs counts against your scale set instance quota and you're charged for your underlying disks. 

If you would like your instances to be deleted when they're evicted, you can set the eviction policy to `Delete`. With the eviction policy set to `delete`, you can create new VMs by increasing the scale set instance count property. The evicted VMs are deleted together with their underlying disks, and therefore you aren't charged for the storage. You can also use the autoscaling feature of scale sets to automatically try to compensate for evicted VMs, however, there's no guarantee that the allocation succeeds. It's recommended you only use the autoscale feature on Azure Spot Virtual Machine Scale Sets when you set the eviction policy to delete to avoid the cost of your disks and hitting quota limits. 

Users can opt in to receive in-VM notifications through [Azure Scheduled Events](../virtual-machines/linux/scheduled-events.md). This notifies you if your VMs are being evicted and you have 30 seconds to finish any jobs and perform shutdown tasks prior to the eviction. 

## Eviction history
You can see historical pricing and eviction rates per size in a region in the portal. Select **View pricing history and compare prices in nearby regions** to see a table or graph of pricing for a specific size.  The pricing and eviction rates in the following images are only examples. 

**Chart**:

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/virtual-machines/spot-chart.png" alt-text="Screenshot of the region options with the difference in pricing and eviction rates as a chart.":::

**Table**:

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/virtual-machines/spot-table.png" alt-text="Screenshot of the region options with the difference in pricing and eviction rates as a table.":::

## Try & restore 

This platform-level feature uses AI to automatically try to restore evicted Azure Spot Virtual Machine instances inside a scale set to maintain the target instance count. 

Try & restore benefits:
- Attempts to restore Azure Spot Virtual Machines evicted due to capacity.
- Restored Spot VMs are expected to run for a longer duration with a lower probability of a capacity triggered eviction.
- Improves the lifespan of an Azure Spot Virtual Machine, so workloads run for a longer duration.
- Helps Virtual Machine Scale Sets to maintain the target count for Azure Spot Virtual Machines, similar to maintain target count feature that already exists for pay-as-you-go VMs.

Try & restore is disabled in scale sets that use [Autoscale](virtual-machine-scale-sets-autoscale-overview.md). The number of VMs in the scale set is driven by the autoscale rules.


## Placement Groups

Placement group is a construct similar to an Azure availability set, with its own fault domains and upgrade domains. By default, a scale set consists of a single placement group with a maximum size of 100 VMs. If the scale set property called `singlePlacementGroup` is set to `false`, the scale set can be composed of multiple placement groups and has a range of 0-1,000 VMs. 

> [!IMPORTANT]
> Unless you're using Infiniband for high-performance computing, it's strongly recommended to set the scale set property `singlePlacementGroup` to `false` to enable multiple placement groups for better scaling across the region or zone. 

## Deploying Azure Spot Virtual Machines in scale sets

To deploy Azure Spot Virtual Machines on scale sets, you can set the new `Priority` flag to `Spot`. All VMs in your scale set will be set to Spot. To create a scale set with Azure Spot Virtual Machines, use one of the following methods:

## [Azure portal](#tab/portal-1)

The process to create a scale set that uses Azure Spot Virtual Machines is the same as detailed in the [getting started article](quick-create-portal.md). When you're deploying a scale set, you can choose to set the Spot flag, eviction type, eviction policy and if you want to try to restore instances:
![Create a scale set with Azure Spot Virtual Machines](media/virtual-machine-scale-sets-use-spot/vmss-spot-portal-1.png)


## [Azure CLI](#tab/cli-1)

> [!IMPORTANT]
>Starting November 2023, scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

The process to create a scale set with Azure Spot Virtual Machines is the same as detailed in the [getting started article](quick-create-cli.md). Just add the '--Priority Spot', and add `--max-price`. In this example, we use `-1` for `--max-price` so the instance won't be evicted based on price.

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image Ubuntu2204 \
    --orchestration-mode Flexible \
    --single-placement-group false \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --eviction-policy Deallocate \
    --max-price -1 \
    --enable-spot-restore True \
    --spot-restore-timeout PT1H
```

## [PowerShell](#tab/ps-1)

> [!IMPORTANT]
>Starting November 2023, scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

The process to create a scale set with Azure Spot Virtual Machines is the same as detailed in the [getting started article](quick-create-powershell.md).
Just add `-Priority "Spot"`, and supply a `-max-price` to the [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).

```powershell
$vmssConfig = New-AzVmssConfig `
    -Location "East US 2" `
    -SkuCapacity 2 `
    -OrchestrationMode "Flexible" `
    -SkuName "Standard_DS2" `
    -Priority "Spot" `
    -max-price -1 `
    -EnableSpotRestore `
    -SpotRestoreTimeout 60 `
    -EvictionPolicy delete
```

## [Azure Resource Manager templates](#tab/ARM-1)

The process to create a scale set that uses Azure Spot Virtual Machines is the same as detailed in the getting started article for [Linux](quick-create-template-linux.md) or [Windows](quick-create-template-windows.md). 

For Azure Spot Virtual Machine template deployments, use`"apiVersion": "2019-03-01"` or later. 

Add the `priority`, `evictionPolicy`, `billingProfile` and `spotRestoryPolicy` properties to the `"virtualMachineProfile":`section and the `"singlePlacementGroup": false,` property to the `"Microsoft.Compute/virtualMachineScaleSets"` section in your template:

```json

{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  },
  "properties": {
    "singlePlacementGroup": false,
    }

        "virtualMachineProfile": {
              "priority": "Spot",
                "evictionPolicy": "Deallocate",
                "billingProfile": {
                    "maxPrice": -1
                },
                "spotRestorePolicy": {
                  "enabled": "bool",
                  "restoreTimeout": "string"
    },
            },
```

To delete the instance after it has been evicted, change the `evictionPolicy` parameter to `Delete`.

---

## Simulate an eviction

You can [simulate an eviction](/rest/api/compute/virtualmachines/simulateeviction) of an Azure Spot Virtual Machine to test how well your application responds to a sudden eviction. 

Replace the following with your information: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

`Response Code: 204` means the simulated eviction was successful. 

For more information, see [Testing a simulated eviction notification](../virtual-machines/windows/spot-powershell.md#simulate-an-eviction).

## Next steps

Check out the [Virtual Machine Scale Set pricing page](https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/) for pricing details.
