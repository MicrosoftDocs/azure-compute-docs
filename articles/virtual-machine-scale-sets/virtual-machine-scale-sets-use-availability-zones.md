---
title: Availability zone options for Azure Virtual Machine Scale Sets
description: Learn how to choose between customer-selected availability zones, automatic zone placement, and regional deployment for Azure Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 08/10/2026
ms.reviewer: fisteele
# Customer intent: As a cloud architect, I want to understand the availability zone placement options for Virtual Machine Scale Sets, so that I can choose the right resiliency configuration for my workload.
---

# Availability zone options for Azure Virtual Machine Scale Sets

Azure availability zones are fault-isolated locations within an Azure region that provide redundant power, cooling, and networking. They allow you to run applications with high availability and fault tolerance to data center failures. Azure regions that support availability zones have a minimum of three separate zones. Each availability zone consists of one or more data centers equipped with independent infrastructure power, network, and cooling. Availability zones are connected by a high-performance network with a round-trip latency of less than 2 milliseconds. For more information, see [Overview of availability zones](/azure/reliability/availability-zones-overview).

To protect your Virtual Machine Scale Sets from datacenter-level failures, you can create a scale set across availability zones. To use availability zones, your scale set must be created in a [supported Azure region](/azure/reliability/availability-zones-region-support).

For more information about how scale sets can be resilient to availability zone failures and other types of failures, see [Reliability in Azure Virtual Machine Scale Sets](/azure/reliability/reliability-virtual-machine-scale-sets?toc=/azure/virtual-machine-scale-sets/toc.json&bc=/azure/virtual-machine-scale-sets/breadcrumb/toc.json).

## Choose an availability zone deployment option

Virtual Machine Scale Sets supports three availability zone deployment options. You can select the zones yourself, use automatic zone placement to let Azure select them, or use a regional deployment without zone-pinned instances.

| Deployment option | Configuration | Zone topology | Who selects the zones? | Use when |
| --- | --- | --- | --- | --- |
| [Customer-selected zones](virtual-machine-scale-sets-configure-customer-selected-zones.md) | Set `zones`, such as `"zones": ["1", "2", "3"]`. | Zonal when you specify one zone, or zone spanning when you specify multiple zones. | You select the zones that the scale set can use. | Your workload must use specific availability zones. |
| [Automatic zone placement (Preview)](virtual-machine-scale-sets-automatic-zone-placement.md) | Set `placement.zonePlacementPolicy` to `"auto"`. | By default, Azure creates a zone-spanning scale set that targets three zones and requires a minimum of two zones. To create a single-zone scale set, set `maxZoneCount` to `1`. | Azure selects the zones within the constraints that you configure. | You want Azure to optimize zone selection based on capacity and SKU availability. |
| Regional (nonzonal) | Don't specify `zones` or a zone placement policy. | Regional instances aren't pinned to availability zones. | Not applicable. | Your workload doesn't require zone-level isolation or zone pinning. |

> [!IMPORTANT]
> You can't configure both the `zones` property and automatic zone placement on the same scale set.

Customer-selected zones and automatic zone placement can result in a zonal (single zone) or zone-spanning scale set. The following sections describe these topologies and regional deployments in more detail.

<a name="zone-redundant-or-zone-spanning"></a>

### Zone spanning

A zone-spanning scale set spreads instances across two or more availability zones. This approach is similar to zone-redundant deployments in other Azure services.

By default, the scale set performs a best effort approach to evenly spread instances across selected zones. However, you can specify that you want strict zone balance by setting `"zoneBalance": "true"` in your deployment. Each VM and its disks are zonal, so they are pinned to a specific zone. Instances between zones are connected by high-performance network with low latency. In the event of a zone outage or connectivity issue, connectivity to instances within the affected zone may be compromised, while instances in other availability zones should be unaffected. You may add capacity to the scale set during a zone outage, and the scale set adds more instances to the unaffected zones. When the zone is restored, you may need to scale down your scale set to the original capacity. A best practice would be to configure [autoscale](virtual-machine-scale-sets-autoscale-overview.md) rules based on CPU or memory usage. The autoscale rules would allow the scale set to respond to a loss of the VM instances in that one zone by scaling out new instances in the remaining operational zones.

Spreading instances across availability zones meets the 99.99% SLA for instances spread across availability zones, and is recommended for most workloads in Azure.

You can configure a zone-spanning scale set by selecting multiple zones yourself or by using automatic zone placement. By default, automatic zone placement targets three availability zones and requires a minimum of two zones.

### Zonal or zone aligned (single zone)

A zonal or zone aligned scale set places instances in a single availability zone. Each VM and its disks are zonal, so they are pinned to a specific zone. This configuration is primarily used when you need lower latency between instances.

You can configure a single-zone scale set by selecting one zone yourself or by setting `maxZoneCount` to `1` with automatic zone placement.

<a name="regional"></a>

### Regional (nonzonal)

A regional (nonzonal) Virtual Machine Scale Set is when the zone assignment isn't explicitly set (`"zones"=[]` or `"zones"=null`). In this configuration, the scale set creates regional (nonzonal, not zone-pinned) instances and implicitly places instances throughout the region. There is no guarantee for balance or spread across zones, or that instances land in the same availability zone. Disk colocation is guaranteed for Ultra and Premium v2 disks, best effort for Premium V1 disks, and not guaranteed for Standard SKU (SSD or HDD) disks.

In the rare case of a full zone outage, any or all instances within the scale set may be impacted.

### Fault domains and availability zones
A fault domain is a fault isolation group within an availability zone or datacenter of hardware nodes that share the same power, networking, cooling, and platform maintenance schedule. VM instances that are on different fault domains are not likely to be impacted by the same planned or unplanned outage. You can specify how instances are spread across fault domains within a region or zone.

- Max spreading (platformFaultDomainCount = 1)
- Fixed spreading (platformFaultDomainCount = 5)
- Fixed spreading aligned with storage disk fault domains (platformFaultDomainCount = 2 or 3, for regional (nonzonal) deployments only)

With max spreading, the scale set spreads your VMs across as many fault domains as possible within each zone. This spreading could be across greater or fewer than five fault domains per zone. With static fixed spreading, the scale set spreads your VMs across the specified number of fault domains. If the scale set can't allocate to at least the specified fault domain count to satisfy the allocation request, the request fails.

**We recommend deploying with max spreading for most workloads**, as this approach provides the best spreading in most cases. If you need replicas to be spread across distinct hardware isolation units, we recommend spreading across availability zones and utilize max spreading within each zone.

> [!NOTE]
> With max spreading, you only see one fault domain in the scale set VM instance view and in the instance metadata regardless of how many fault domains the VMs are spread across. The spreading within each zone is implicit.

### Placement groups

> [!IMPORTANT]
> Placement groups only apply to Virtual Machine Scale Sets running in Uniform orchestration mode.

When you deploy a scale set, you can deploy with a single [placement group](./virtual-machine-scale-sets-placement-groups.md) per availability zone, or with multiple per zone. For regional (nonzonal) scale sets, the choice is to have a single placement group in the region or to have multiple in the region. If the scale set property called `singlePlacementGroup` is set to false, the scale set can be composed of multiple placement groups and has a range of 0-1,000 VMs. When set to the default value of true, the scale set is composed of a single placement group, and has a range of 0-100 VMs. For most workloads, we recommend multiple placement groups, which allows for greater scale. In API version *2017-12-01*, scale sets default to multiple placement groups for single-zone and cross-zone scale sets, but they default to single placement group for regional (nonzonal) scale sets.

> [!NOTE]
> If you use max spreading, you must use multiple placement groups.

### Zone balancing

Zone placement determines which availability zones a scale set can use when placing new instances. Zone balancing controls how evenly instances are distributed across those zones after the eligible zones are selected.

For scale sets that span multiple zones, choose between best-effort and strict zone balancing. Zone balancing applies regardless of whether you select the zones or use automatic zone placement. For more information, see [Zone balancing in Virtual Machine Scale Sets](virtual-machine-scale-sets-zone-balancing.md).

Zone balancing is also different from [Automatic Zone Balance](auto-zone-balance-overview.md). Zone balancing controls instance distribution during scaling operations. Automatic Zone Balance proactively detects existing imbalances and corrects them by creating a replacement VM in an under-provisioned zone and deleting a VM from an over-provisioned zone.

## Next steps

- [Configure customer-selected availability zones](virtual-machine-scale-sets-configure-customer-selected-zones.md).
- Configure [automatic zone placement](virtual-machine-scale-sets-automatic-zone-placement.md).
- Learn about [zone balancing in Virtual Machine Scale Sets](virtual-machine-scale-sets-zone-balancing.md).
- Learn about [Automatic Zone Balance](auto-zone-balance-overview.md).
