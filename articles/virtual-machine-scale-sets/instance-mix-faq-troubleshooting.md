---
title: Frequently Asked Questions and Troubleshooting
description: Frequently asked questions and troubleshooting for instance mix on virtual machine scale sets. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 03/03/2025
ms.reviewer: jushiman
---

# Virtual machine scale sets with instance mix frequently asked questions and troubleshooting

## Frequently Asked Questions
### Can I use Spot and Standard VMs with instance mix?
Yes, you can use both Spot and Standard Virtual Machines (VMs) in your scale set deployments using instance mix. To do so, use [Spot Priority Mix](./spot-priority-mix.md) to define a percentage split of Spot and Standard VMs. 

### Can I mix multiple CPU architectures with instance mix?
No, you can't mix multiple CPU architectures in a scale set using instance mix.

### Which regions support instance mix?
All public Azure regions support instance mix.

### Will instance mix request quota for me?
No, you must have quota for the VMs you specify in the `skuProfile`. If you don't have quota for a given VM size, we'll try using another VM size specified that does have quota.

### I updated my scale set to use instance mix, why aren't my VMs aligning to my allocation strategy?
After updating your scale set to use instance mix, all scale in or scale out actions use the inputs from instance mix to determine which VMs to scale in and out. 

### Can I use reservations or savings plan with instance mix?
Yes, you can apply your reservations and savings plan with instance mix. It's recommended that you use the `Prioritized` allocation strategy and set the reservation or savings plan VM sizes as the first rank.

## Troubleshooting
| Error Code                                 | Error Message                                                                                                        | Troubleshooting options                                                                                                                                                                                                                                                                                              |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SkuProfileAllocationStrategyInvalid`        | `Sku Profile’s Allocation Strategy is invalid.`                                                                        | Ensure that you're using `CapacityOptimized`, `Prioritized`, or `LowestPrice` as the `allocationStrategy`                                                                                                                                                                                                                     |
| `SkuProfileVMSizesCannotBeNullOrEmpty`       | `Sku Profile VM Sizes cannot be null or empty. Please provide a valid list of VM Sizes and retry.`                     | Provide at least one VM size in the `skuProfile`.                                                                                                                                                                                                                                                                    |
| `SkuProfileHasTooManyVMSizesInRequest`       | `Too many VM Sizes were specified in the request. Please provide no more than 5 VM Sizes.`                             | At this time, you can specify up to five VM sizes with instance mix.                                                                                                                                                                                                                                                 |
| `SkuProfileVMSizesCannotHaveDuplicates`      | `Sku Profile contains duplicate VM Size: {duplicateVmSize}. Please remove any duplicates and retry.`                   | Check the VM SKUs listed in the `skuProfile` and remove the duplicate VM size.                                                                                                                                                                                                                                       |
| `SkuProfileScenarioNotSupported`             | `{propertyName} is not supported on Virtual Machine Scale Sets with Sku Profile.`                                       | Instance mix doesn’t support certain scenarios today, like Azure Dedicated Host (`properties.hostGroup`), Capacity Reservations (`properties.virtualMachineProfile.capacityReservation`), and StandbyPools (`properties.standbyPoolProfile`). Adjust the template to ensure you’re not using unsupported properties. |
| `SkuNameMustBeMixIfSkuProfileIsSpecified`    | `Sku name is {skuNameValue}. Virtual Machine Scale Sets with Sku Profile must have the Sku name property set to "Mix".` | Ensure that the `sku.name property` is set to `"Mix"`.                                                                                                                                                                                                                                                               |
| `SkuTierMustNotBeSetIfSkuProfileIsSpecified` | `Sku tier is {skuTierValue}. Virtual Machine Scale Sets with Sku Profile must not have the Sku tier property set.`     | `sku.tier` is an optional property for scale sets. With instance mix, `sku.tier` must be set to `null` or not specified.                                                                                                                                                                                             |
