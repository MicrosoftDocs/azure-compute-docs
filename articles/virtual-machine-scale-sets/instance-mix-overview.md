---
title: Use multiple Virtual Machine sizes with instance mix
description: Use multiple Virtual Machine sizes in a scale set using instance mix. Optimize deployments using allocation strategies. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 06/10/2025
ms.reviewer: jushiman
---

# Use multiple Virtual Machine sizes with instance mix

Instance mix enables you to specify multiple different Virtual Machine (VM) sizes in your Virtual Machine Scale Set with Flexible Orchestration Mode, and an allocation strategy to further optimize your deployments. 

Instance mix is best suited for workloads that are flexible in compute requirements and can be run on various different sized VMs. Using instance mix you can:
- Deploy a heterogeneous mix of VM sizes in a single scale set. You can view max scale set instance counts in the [documentation](./virtual-machine-scale-sets-orchestration-modes.md#what-has-changed-with-flexible-orchestration-mode).
- Optimize your deployments for cost, capacity, or rank through allocation strategies.
- Continue to make use of scale set features, like [Autoscale](./virtual-machine-scale-sets-autoscale-overview.md), [Spot Priority Mix](./spot-priority-mix.md), or [Upgrade Policies](./virtual-machine-scale-sets-set-upgrade-policy.md).
  
## Use cases
Instance mix is ideal for scenarios where flexibility and capacity attainment are key. Common use cases include:

- Running cost-sensitive workloads that can use multiple Spot VM sizes to minimize expenses.
- Gradually adopting newer VM generations, such as D32sv5 and D32sv6, while continuing to utilize existing older VMs in the same series.
- Supporting workloads with a primary or preferred VM size, while having fallback or secondary VM sizes available for added flexibility.
- Ensuring high availability and reliability by distributing a diverse mix of VMs across Availability Zones and Fault Domains, especially during periods of high demand.

## Changes to existing scale set properties
### sku.name
The `sku.name` property should be set to `"Mix"`. VM sizes are defined in the `skuProfile`.

### sku.tier
The `sku.tier` property is currently an optional scale set property and should be set to `null` for instance mix scenarios.

### sku.capacity
The `sku.capacity` property continues to represent the overall size of the scale set in terms of the total number of VMs.

### scaleInPolicy
The optional scale-in property isn't needed for scale set deployments using instance mix. During scaling in events, the scale set utilizes the allocation strategy to inform the decision on which VMs should be scaled in. For example, when you use `LowestPrice`, the scale set scales in by removing the more expensive VMs first.

## New scale set properties
### skuProfile
The `skuProfile` property represents the umbrella property for all properties related to instance mix, including VM sizes and allocation strategy.

### vmSizes
The `vmSizes` property is where you specify the specific VM sizes that you're using as part of your scale set deployment with instance mix.

### allocationStrategy
Instance mix introduces the ability to set allocation strategies for your scale set. The `allocationStrategy` property is where you specify which allocation strategy you'd like to use for your instance mix scale set deployments. There are three options for allocation strategies, `lowestPrice`, `capacityOptimized`, and `Prioritized`. Allocation strategies apply to both Spot and Standard VMs.

#### lowestPrice (default)
This allocation strategy is focused on workloads where cost and cost-optimization are most important. When evaluating what VM split to use, Azure looks at the lowest priced VMs of the VM sizes specified. Azure also considers capacity as part of this allocation strategy. The scale set deploys as many of the lowest priced VMs as it can, depending on available capacity, before moving on to the next lowest priced VM size specified. `lowestPrice` is the default allocation strategy.

#### capacityOptimized
The `capacityOptimized` allocation strategy is designed for workloads where securing VM capacity is the highest priority. This approach ensures that VMs are allocated based on availability rather than cost considerations.

##### How `capacityOptimized` allocation works
- Azure prioritizes available capacity, without factoring in price, when determining which VM sizes to deploy.
- VM sizes are dynamically selected based on underlying capacity availability, ensuring that instances can be allocated even in highly utilized regions.
- This strategy is useful for workloads that must secure compute resources without delays due to capacity shortages.

##### Considerations
- Cost isn't considered. The selected VM sizes may include more expensive options if those are the most readily available.
- No user-defined ranking is required. Unlike the `Prioritized` allocation strategy, the selection process is fully automated based on Azure’s capacity insights.
- VM allocation is region-dependent. Availability may vary across Azure regions, and the selection process adapts accordingly.
- Best suited for critical workloads. This strategy is ideal when securing VMs is more important than optimizing for cost.

Users can ensure that their workloads receive the necessary compute resources, even in situations where capacity constraints might otherwise prevent VM allocation.

#### Prioritized (Preview)
The `Prioritized` allocation strategy enables control over how VM sizes are allocated by defining a priority ranking. `Prioritized` allows for a more predictable allocation order based on preferred VM sizes.

##### How `Prioritized` allocation works
- Each VM size in the `vmSizes` list can be assigned a priority rank to influence the order in which instances are allocated.
- Lower rank numbers indicate higher priority. For example, a VM with a rank of 0 is prioritized over a VM with a rank of 2.
- If multiple VM sizes have the same rank, they share the same allocation priority, Azure distributes VMs across those sizes based on availability.

##### Considerations
- Ranking is optional. If no ranks are provided, all VM sizes are treated with equal priority.
- Ranks must be within the range of the vmSizes list size. For example, if there are five VM sizes, ranks must be within the range of 0 to 4 (or fewer if not all are assigned ranks).
- Ranks don't need to be sequential. It's valid to have ranks such as 0, 2, 5 without needing to define 1, 3, or 4.
- Duplicate ranks are allowed. Multiple VM sizes can share the same rank, allowing for a tiered allocation approach where several sizes are treated equally.
- Resource availability still applies. Even if a VM size has the highest priority, allocation is subject to regional capacity constraints.

## Cost
Following the scale set cost model, usage of instance mix is free. You continue to only pay for the underlying resources, like the VM, disk, and networking.

## Recommendations
* Use VMs of similar size for your workload to ensure an even spread of traffic from the load balancer. For example, using the `Standard_D8s_v4` and the `Standard_D8s_v5` VM sizes in your deployment would ensure your workload always runs on an eight core VM.
* Use VMs of [similar type](../virtual-machines/sizes/overview.md#list-of-vm-size-families-by-type) for consistent performance. 
* To benefit from reservation pricing, use the `Prioritized` allocation strategy and set your reservation VM sizes as the first rank.
* To benefit from savings plan pricing, use the `Prioritized` allocation strategy and set your savings plan VM sizes as the first rank.
* To ensure a smooth autoscaling experience, use VMs of similar vCPU and memory configurations.
  
## Limitations

When using instance mix, keep the following limitations in mind:
- **Orchestration Mode**: Instance mix is only available for scale sets using Flexible Orchestration Mode.
- **Quota Requirements**: Ensure you have sufficient quota for the VM sizes you're requesting with instance mix.
- **Virtual Machine Type**: Only VMs that are in the [A](../virtual-machines/sizes/general-purpose/a-family.md), [B](../virtual-machines/sizes/general-purpose/b-family.md), [D](../virtual-machines/sizes/general-purpose/d-family.md), [E](../virtual-machines/sizes/memory-optimized/e-family.md), and [F](../virtual-machines/sizes/compute-optimized/f-family.md) families can be specified in the `skuProfile`.
- **VM Size Limit**: You can specify up to **five VM sizes** in an instance mix deployment.
- **Virtual Network Requirement**: For REST API deployments, an existing virtual network must be present in the resource group where the scale set is being deployed.
- **Architecture Consistency**: Mixing VM architectures (for example, Arm64 and x64) in the same instance mix deployment isn't supported.
- **Storage Interface Consistency**: VMs with different storage interfaces (for example, SCSI and NVMe) can't be mixed in the same instance mix.
- **Security Profile Consistency**: All VMs specified in the `skuProfile` must share the same Security Profile.
- **Local Disk Configuration**: All selected VM sizes must have the same local disk configuration.
- **Unsupported Features**: Instance mix doesn't support the following features:
    - Standby Pools
    - Azure Dedicated Host
    - Proximity Placement Groups
- **Scaling Behavior with MaxSurge**: During scaling actions with MaxSurge, instance mix replaces the VM with the same size it had before the scaling action.

## Next steps
Learn how to [create a scale set using instance mix](instance-mix-create.md).