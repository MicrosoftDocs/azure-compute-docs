---
title: Use multiple Virtual Machine sizes with instance mix
description: Use multiple Virtual Machine sizes in a scale set using instance mix. Optimize deployments using allocation strategies. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 09/12/2025
ms.reviewer: jushiman
# Customer intent: As a cloud administrator, I want to utilize multiple VM sizes in a scale set with instance mix, so that I can optimize cost, capacity, and flexibility for my diverse workload requirements.
---

# Use multiple Virtual Machine sizes with instance mix

Instance mix lets you specify multiple Virtual Machine (VM) sizes in a Virtual Machine Scale Set that uses Flexible Orchestration Mode. Use instance mix to increase provisioning success, optimize costs, or create predictable allocation ordering for workloads that can run on different VM sizes.

> [!IMPORTANT]
> Instance mix is available only for scale sets that use Flexible Orchestration Mode.

## Prerequisites

- A scale set that uses Flexible Orchestration Mode.
- Sufficient quota for each VM size in the target subscription and region.
- Consistent VM characteristics across selected sizes: architecture (x64/Arm64), storage interface, local disk configuration, and security profile.

## When to use instance mix

- You want to run a heterogeneous set of VM sizes in a single scale set to increase the likelihood of successful provisioning.
- You want to reduce cost by allowing Azure to choose lower-cost sizes when suitable.
- You need predictable allocation order to align with reservations, licensing, or savings plans using the Prioritized strategy.

## How instance mix works

- Define up to five VM sizes in the `skuProfile.vmSizes` list.
- Select an `allocationStrategy` to control how Azure chooses VM sizes at provisioning time.
- During scale-out, Azure evaluates availability and, depending on strategy, price to and allocate instances that satisfy the chosen strategy, subject to quotas and regional capacity.

## Allocation strategies

Instance mix supports three allocation strategies. Choose the strategy that matches your priorities (cost, capacity, or predictable ordering).

| Strategy | Best for | Behavior | Notes |
|---|---|---|---|
| `lowestPrice` (default) | Cost-sensitive, fault-tolerant workloads | Prefers the lowest-cost VM sizes from the `vmSizes` list while considering available capacity. Deploys as many of the lowest-priced VMs as capacity allows before moving to higher-priced sizes. | Best suited for Spot VMs. Higher-cost sizes may be selected to secure capacity. |
| `capacityOptimized` | Critical workloads that must provision reliably | Prioritizes VM sizes with the highest likelihood of availability in the target region; cost isn't considered. | Availability varies by region. May select higher-cost sizes to secure capacity. |
| `Prioritized` (Preview) | Predictable allocation order, reservation alignment | Respects user-defined `rank` values on VM sizes; lower rank means higher priority. Azure allocates instances according to rank while respecting capacity. | Ranks are optional, can be duplicated, and don't need to be sequential. Allocation remains subject to regional capacity constraints. |

> [!NOTE]
> Use `rank`  only with the `Prioritized` strategy. Omit ranks for `lowestPrice` and `capacityOptimized`.

## Scale set properties

### Changes to existing properties

| Property | Change | Notes |
|---|---|---|
| `sku.name` | Must be set to `"Mix"` for instance mix deployments. | VM sizes are moved into the `skuProfile` configuration. |
| `sku.tier` | Should be `null` for instance mix scenarios. | Optional property; set to `null` to avoid tier mismatch across sizes. |
| `sku.capacity` | Represents the desired total number of VMs in the scale set. | Keeps representing the scale set capacity (desired instances). |
| `scaleInPolicy` | Not required for instance mix. | Instance mix uses `allocationStrategy` to guide allocation; scale-in behavior follows the scale set's policy and allocation strategy. |

### New properties

| Property | Type | Description | Example Value |
|---|---|---|---|
| `skuProfile` | Object | Container for instance mix configuration (vmSizes, allocationStrategy, etc.). | `{ "vmSizes": [...], "allocationStrategy": "Prioritized" }` |
| `vmSizes` | Array of strings or objects | List (max 5) of VM sizes to include in the instance mix. Each item can be a string (size name) or an object with an optional `rank` for the `Prioritized` strategy. | `[{ "name": "Standard_D8s_v5", "rank": 0 }, { "name":"Standard_D8as_v5", "rank": 1]` |
| `allocationStrategy` | String | Determines how Azure chooses VM sizes at provisioning time. One of: `lowestPrice`, `capacityOptimized`, `Prioritized`. | `"Prioritized"` |

## Example: Prioritized allocation (JSON fragment)

```json
{
  "skuProfile": {
    "vmSizes": [
      { "name": "Standard_D8s_v5", "rank": 0 },
      { "name": "Standard_D8s_v4", "rank": 1 },
      { "name": "Standard_D4s_v5", "rank": 2 }
    ],
    "allocationStrategy": "Prioritized"
  }
}
```

> [!NOTE]
> Ranks: lower numbers indicate higher priority. Ranks can be non-sequential and duplicated. Omit ranks when using `lowestPrice` or `capacityOptimized`.

## Deployment checklist

Before you deploy an instance mix scale set:

- Verify the scale set is using Flexible Orchestration Mode.
- Confirm VM quotas for each selected size in the target subscription and region.
- Ensure all selected VM sizes have consistent architecture, storage interface, local disk configuration, and security profile.
- Choose an allocation strategy that matches your goals (cost, availability, predictability).
- For REST API deployments, ensure a virtual network exists in the target resource group.

## Recommendations

- To ensure balanced load distribution, use VM sizes with similar vCPU and memory.
- For consistent performance, use VM sizes of similar type (for example, both D-series).
- For reservation or savings-plan benefits, use `Prioritized` and place reservation-backed sizes at higher priority.

## Limitations and unsupported scenarios

- Orchestration mode: instance mix is available only with Flexible Orchestration Mode.
- VM families supported in `skuProfile`: A, B, D, E, and F families only.
- Up to five VM sizes can be specified.
- You can't mix VM architectures (for example, Arm64 and x64) in the same instance mix.
- VMs with different storage interfaces (SCSI vs NVMe) can't be mixed.
- You can't mix VM SKUs that use premium storage and non-premium storage in the same instance mix.
- All VMs must share the same Security Profile and local disk configuration.
- **DiffDisk settings**: Instance mix currently doesn't support `diffDiskSettings` on the OS disk.
- Instance mix doesn't support: Standby Pools, Azure Dedicated Host, Proximity Placement Groups, or on-demand capacity reservations.

## Next steps
Learn how to [create a scale set using instance mix](instance-mix-create.md).