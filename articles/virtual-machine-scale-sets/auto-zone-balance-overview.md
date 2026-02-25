---
title: Automatic zone balance for Virtual Machine Scale Sets (Preview)
description: Learn about the Automatic Zone Balance feature for virtual machine scale sets.
author: hilaryw29
ms.author: hilarywang
ms.topic: article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 04/18/2025
---

# Automatic zone balance for Virtual Machine Scale Sets (Preview)

Automatic zone balance helps you maintain zone-resilient scale sets that are evenly distributed across availability zones. This feature monitors your scale set and moves VMs to maximize resiliency, reducing the risk of zone imbalances due to capacity constraints or scaling operations.

> [!IMPORTANT]
> Automatic Zone Balance for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature can change before general availability (GA).

> [!NOTE]
> Automatic zone balance doesn't monitor Virtual Machine (VM) health for zone outages and shouldn't be used as a zone-down recovery mechanism.

## Background

When you deploy a Virtual Machine Scale Set across multiple availability zones, the scale set attempts to maximize resiliency by spreading your VMs as evenly as possible. However, factors like capacity constraints or scaling operations can cause your scale set to become imbalanced across availability zones over time, with some zones having more VM instances than others. This imbalance can go unnoticed, but it increases the risk that a single zone failure could affect a disproportionate number of your VMs, reducing your application's availability. Automatic zone balance continuously monitors these imbalances and rebalances VMs in the background to maximize resiliency across zones.

**Key Terms:**
- A scale set is considered **balanced** if each zone has the same number of VMs +/- 1 VM as all other zones for the scale set. A scale set that doesn't meet this condition is considered imbalanced. More detail on zone balance is available [here](virtual-machine-scale-sets-zone-balancing.md).
- An **under-provisioned zone** is an availability zone with the fewest scale set instances.
  - In a scale set with 1 VM in zone 1, 3 VMs in zone 2, and 3 VMs in zone 3; zone 1 is the under-provisioned zone. 
- An **over-provisioned zone** is an availability zone with the most scale set instances.
  - In a scale set with 1 VM in zone 1, 3 VMs in zone 2, and 3 VMs in zone 3; zones 2 and 3 are the over-provisioned zones.

## How does automatic zone balance work? 

Automatic zone balance is designed for Virtual Machine Scale Sets deployed across two or more availability zones. The rebalancing process works by moving VMs across availability zones using a create-before-delete approach, ensuring minimal disruption to your applications.

When a zone imbalance is detected, automatic zone balance creates a new VM in the most under-provisioned zone. Once the new VM is healthy, automatic zone balance deletes a VM from the most over-provisioned zone. Only one VM is rebalanced at a time, and new VMs are always created with the latest SKU specified in your scale set model.

> [!IMPORTANT]
> During rebalancing, automatic zone balance temporarily increases your scale set capacity by one VM. Before enabling this feature, verify that:
> - If using autoscale, your maximum instance count and scale-in rules should have sufficient buffer to accommodate one extra VM during the create-before-delete process
> - Your subscription has adequate quota for the temporary capacity increase

Automatic zone balance waits up to 90 minutes for a newly created VM to report a healthy application signal. If the new VM becomes healthy, the source VM in the over-provisioned zone is deleted. If the new VM doesn't become healthy within 90 minutes, automatic zone balance checks the health of the source VM. If the source VM is healthy, the new (unhealthy) VM is deleted. However, if the source VM is unhealthy, it is then deleted and the new VM is kept. This workflow helps maintain zone balance while prioritizing workload health and availability.

:::image type="content" source="./media/virtual-machine-scale-sets-auto-zone-balance/auto-zone-balance-workflow.svg" alt-text="Diagram of the workflow for automatic zone balance.":::

### Safety features

#### Safety checks before rebalancing
Automatic zone balance is designed to be minimally intrusive, prioritizing the stability and availability of your workloads. A rebalance operation (creating a VM in a new zone and deleting a VM from an over-provisioned zone) only begins if the following safety conditions are met:

- The scale set isn't marked for deletion.
- The scale set doesn't have any ongoing or recently completed `PUT`, `PATCH`, `POST` operations within the past 60 minutes; such as VMs being added or deleted, or upgrades in progress.

Automatic zone balance won't move VMs under the [instance protection policy](./virtual-machine-scale-sets-instance-protection.md), or in deallocated / to-be-deleted state. Only one VM is moved per rebalance operation to minimize churn and ensure that changes to your scale set are gradual and controlled.

### Automatic instance repairs integration

Automatic zone balance works together with [automatic instance repairs](./virtual-machine-scale-sets-automatic-instance-repairs.md) to help you maintain highly available scale sets:

- **Automatic Instance Repairs** monitors and repairs unhealthy VM instances within your scale set
- **Automatic Zone Balance** ensures VMs are evenly distributed across availability zones

When you enable automatic zone balance, automatic instance repairs is also enabled by default. This bundling ensures that your scale set benefits from both instance-level health monitoring and zone-level resiliency. If your scale set already has automatic instance repairs enabled, your existing configuration is preserved.

To learn more about automatic instance repairs, see [Automatic instance repairs for Azure Virtual Machine Scale Sets](./virtual-machine-scale-sets-automatic-instance-repairs.md). If you'd like to use automatic zone balance without automatic instance repairs, see [Automatic Zone Balance without Instance Repairs](./auto-zone-balance-enable.md#enable-automatic-zone-balance-without-instance-repairs).

## Limitations

- **Recommended for stateless workloads**: Automatic zone balance uses delete and recreate operations to move VMs across availability zones. Instance IDs, networking, and disks aren't preserved today as part of rebalancing.
- **Best-effort operation**: Automatic zone balance could be delayed if an availability zone has limited capacity.
- **Subject to subscription quota limits**: Automatic zone balance requires enough quota to temporarily exceed current VM count when creating a new VM. 
- **SKU preservation**: New VMs created by Automatic Zone Balance always use the latest SKU in your scale set model; any VMs with a different SKU won't retain their SKU after rebalancing.
  - If you would like to prevent specific VMs from being rebalanced, you can [apply an instance protection policy](./virtual-machine-scale-sets-instance-protection.md).

## Next steps
- Learn how to [enable automatic zone balance on your scale set](./auto-zone-balance-enable.md).
- Learn more about [zone balancing for Virtual Machine Scale Sets](./virtual-machine-scale-sets-zone-balancing.md). 
