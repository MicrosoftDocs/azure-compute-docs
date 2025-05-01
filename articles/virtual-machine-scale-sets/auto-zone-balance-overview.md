---
title: Automatic zone balance for Azure Virtual Machine Scale Sets (Preview)
description: Learn about the Automatic Zone Balance feature for VMSS.
author: hil       ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 04/18/2025
ms.reviewer: hilarywang
---

# Automatic zone balance for Azure Virtual Machine Scale Sets (Preview)

Automatic zone balance helps you maintain zone-resilient scale sets that are evenly distributed across availability zones. This feature monitors your scale set and moves VMs to maximize resiliency, reducing the risk of zonal imbalances due to capacity constraints or scaling operations.

> [!NOTE]
> Automatic zone balance does not monitor VM health for zonal outages and should not be used as a zone-down recovery mechanism.

## Background

When you deploy a Virtual Machine Scale Set (VMSS) across multiple availability zones, the scale set attempts to maximize resiliency by spreading your VMs as evenly as possible. However, factors like capacity constraints or scaling operations can cause your scale set to become "zonally imbalanced" over time, with some zones having more VM instances than others. This imbalance can go unnoticed, but it increases the risk that a single zone failure could impact a disproportionate number of your VMs, reducing your application's availability. 

Automatic zone balance is designed to help improve zonal resiliency by monitoring your VMSS and automatically redistributing VMs as needed to maintain an even spread across zones. 

Key Terms:
- A scale set is considered "zonally balanced" if each zone has the same number of VMs +/- 1 VM as all other zones for the scale set. A scale set that doesn’t meet this condition is considered "zonally imbalanced". More details on zone balance available [here](./virtual-machine-scale-sets-use-availability-zones.md#zone-balancing).
- An "under-provisioned zone" is an availability zone with the fewest VM scale set instances.
  - In a scale set with one VM in zone 1, three VMs in zone 2, and three VMs in zone 3 -- zone 1 is the under-provisioned zone. 
- An "over-provisioned zone" is an availability zone with the most VM scale set instances.
  - In a scale set with one VM in zone 1, three VMs in zone 2, and three VMs in zone 3 -- zones 2 and 3 are the over-provisioned zones.

## How does automatic zone balance work? 

Automatic zone balance is designed for Virtual Machine Scale Sets (VMSS) deployed across two or more availability zones. The rebalancing process works by moving VMs across availability zones using a create before delete approach, ensuring minimal disruption to your applications.

When a zonal imbalance is detected, automatic zone balance creates a new VM in the most under-provisioned zone (the zone with the fewest VM instances) and, once the new VM is healthy, deletes a VM from the most over-provisioned zone (the zone with the most VM instances). Only one VM is rebalanced at a time, and new VMs are always created with the latest SKU specified in your VMSS model.

After creating a new VM, automatic zone balance waits up to 90 minutes for it to report a healthy application signal. If the new VM becomes healthy, the original VM in the over-provisioned zone is deleted. If the new VM doesn't become healthy within 90 minutes, automatic zone balance checks the health of the original VM: if the original VM is healthy, the new (unhealthy) VM is deleted; if the original VM is unhealthy, it's deleted and the new VM is kept. This workflow helps maintain zone balance while prioritizing workload health and availability.

![Automatic zone balance Workflow](./media/virtual-machine-scale-sets-auto-zone-balance/AutoZoneBalanceWorkflow.png)

### Safety Features

Automatic zone balance is designed to be minimally intrusive, prioritizing the stability and availability of your workloads. A rebalance operation (creating a VM in a new zone and deleting a VM from an over-provisioned zone) only begins if the following safety conditions are met:

- The VMSS is not marked for deletion.
- The scale set doesn’t have any ongoing or recently completed `PUT`, `PATCH`, `POST` operations within the past 60 minutes -- such as VMs being added or deleted, or upgrades in progress.

Automatic zone balance performs a maximum of one rebalance operation every 12 hours. Only one VM is moved in each rebalance operation. This limit is in place to minimize churn and ensure that changes to your scale set are gradual and controlled. Automatic zone balance will not move VMs under the instance protection policy, or in deallocated / to-be-delete state. 

## Limitations

- **Recommended for stateless workloads**: Automatic zone balance uses delete and recreate operations to move VMs across availability zones; Instance IDs, networking, and disks aren't preserved today as part of rebalancing.
- **Best-effort operation**: Automatic zone balance may be delayed if an availability zone has limited capacity.
- **Subject to subscription quota limits**: Auto AZ Balance requires enough quota to temporarily exceed current VM count when creating a new VM. 
- **SKU preservation**: New VMs created by automatic zone balance always use the latest SKU in your VMSS model; any VMs with a different SKU won'ts retain their SKU during rebalancing.
  - If you would like to prevent specific VMs from being rebalanced, you can [apply an instance protection policy](./virtual-machine-scale-sets-instance-protection.md).

## Next Steps
Learn how to [enable automatic zone balance on your scale set](./auto-zone-balance-enable.md).