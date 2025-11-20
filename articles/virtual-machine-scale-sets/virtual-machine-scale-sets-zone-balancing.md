---
title: Zone Balancing in Scale Sets
description: Find out about zone balancing in scale sets, including zone balancing modes and rebalancing behavior.
author: johndowns # TODO find an owner
ms.author: jodowns
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 11/11/2025
---

# Zone balancing in Virtual Machine Scale Sets

A *Zone-spanning* scale set spreads virtual machine (VM) instances across multiple availability zones, and uses *zone balancing* to evenly distribute instances across the zones that you've selected. This article discusses how a zone-spanning scale set uses zone balancing, including the difference between balanced and unbalanced, balancing modes and rebalancing behavior.


## Zone balancing modes

In order to set the zone balancing mode, your scale set must use multiple zones. A scale set that doesn't use zones or uses only one zone doesn't require balancing and therefore doesn't have a balancing mode.

For a scale set that uses multiple zones, you can choose between two zone balancing modes:

- **Best-effort zone balancing (Default mode):** The scale set aims to maintain balance across zones during scaling operations, but it's not guaranteed to remain balanced.

    If one zone becomes unavailable, the scale set allows temporary imbalance to ensure the scale set can scale out. However, this imbalance is only permitted when a single zone is unavailable. Once the zone is restored, the scale set adjusts by adding VMs to under-provisioned zones or removing VMs from over-provisioned zones to restore balance.

    If two or more zones go down, the scale set can't proceed with scaling operations, and any scaling operations are blocked.

- **Strict zone balancing:** Any scaling operation that would result in an unbalanced scale set is blocked, even if one or more zones are down.



## Balanced and unbalanced scale sets

A scale set is considered *balanced* if each zone has the same number of VMs ±1 VM. The deviation of 1 enables you to scale to any number of instances, and not just a multiple of the number of zones that the set uses.

Here are some examples of how Virtual Machine Scale Sets determines zone balancing for a zone-spanning scale set that's configured to use three zones:

- Example 1: A scale set with 2 VMs in zone 1, 2 VMs in zone 2, and 2 VMs in zone 3 is considered *balanced*. Each zone has the exact same number of instances.

    :::image type="content" source="media/virtual-machine-scale-sets-zone-balancing/zone-balancing-balanced-even.svg" alt-text="Diagram that shows a balanced scale set, with two instances in each zone." border="false":::

- Example 2: A scale set with 2 VMs in zone 1, 3 VMs in zone 2, and 3 VMs in zone 3 is considered *balanced*. There is only one zone with a different VM count and it is only 1 less than the other zones.

    :::image type="content" source="media/virtual-machine-scale-sets-zone-balancing/zone-balancing-balanced-odd.svg" alt-text="Diagram that shows a balanced scale set, with two instances in zone 1 and three instances in zones 2 and 3." border="false":::

- Example 3: A scale set with 1 VM in zone 1, 3 VMs in zone 2, and 3 VMs in zone 3 is considered *unbalanced*. Zone 1 has 2 fewer VMs than zones 2 and 3, which exceeds the allowed threshold of ±1 VM.

    :::image type="content" source="media/virtual-machine-scale-sets-zone-balancing/zone-balancing-unbalanced.svg" alt-text="Diagram that shows an unbalanced scale set, with one instance in zone 1 and three instances in zones 2 and 3." border="false":::


## Rebalancing and scale-out operations


When you add availability zones to an existing scale set, existing VMs remain unchanged and do not get moved or redistributed. In addition, adding a zone does not trigger a rebalancing operation. Rebalancing only happens during scale-out operations when new instances are added to the scale set. Rebalancing does not replace existing instances.

For example, suppose you move from a nonzonal scale set to a zone-spanning scale set that uses three zones. Immediately after adding zones to the scale set, the existing instances remain in a nonzonal state.

You can trigger *rebalancing* by running the following sequence of operations:
	
1. **Scale out.** Add more instances by [updating the scale set's capacity](virtual-machine-scale-sets-autoscale-overview.md). The new capacity should be set to the original capacity *plus* the number of new instances.

    For example, if your scale set currently has 5 nonzonal instances and you would like to scale out so that you have 3 instances in each of 3 zones, you should set the capacity to 14 (5 + (3 * 3)).

    The scale set attempts to create the new instances in the zones configured on the scale set.

1. **Scale in.** When the new instances are ready, scale in your scale set to remove the old instances. This process leaves your scale set in a balanced state.

    You can either manually delete specific instances, or scale in by reducing the scale set capacity. When scaling in via reducing scale set capacity, the platform always prefers removing the nonzonal instances, then follows the scale set's [scale-in policy](virtual-machine-scale-sets-scale-in-policy.md).

    > [!NOTE]
    > If you use the Flexible orchestration mode and attach, detach, or remove individual VMs, you should check the zones your VMs are in. If the VMs are all in a single zone, your scale set isn't resilient to an outage in that zone.

## VM extensions

It's possible that VMs in the scale set are successfully created, but extensions on those VMs fail to deploy. Any VMs with extension deployment failures are still counted when determining if a scale set is balanced. For example, a scale set with 3 VMs in zone 1, 3 VMs in zone 2, and 3 VMs in zone 3 is considered balanced even if all extensions failed in zone 1 and all extensions succeeded in zones 2 and 3.
