---
title: Availability sets overview 
description: Learn about availability sets for virtual machines in Azure.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: conceptual
ms.date: 10/10/2024
ms.custom: engagement-fy23
---

# Availability sets overview

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs

> [!NOTE]
> We recommend that customers choose [virtual machine scale sets with flexible orchestration mode](../virtual-machine-scale-sets/overview.md) for high availability with the widest range of features. Virtual machine scale sets allow VM instances to be centrally managed, configured, and updated, and will automatically increase or decrease the number of VM instances in response to demand or a defined schedule. Availability sets only offer high availability.

This article provides you with an overview of the availability features of Azure virtual machines (VMs).

## What is an availability set? 

Availability sets are logical groupings of VMs that reduce the chance of correlated failures bringing down related VMs at the same time. Availability sets place VMs in different fault domains for more reliability. This is especially beneficial if a region does not support availability zones. When using Availability Sets, this requires the creation of two or more VMs within an availability set. Using two or more VMs in an availability set helps "highly available" applications meet the 99.95% Azure SLA. There is no extra cost for using availability sets, as you only pay for each VM instance you create.

Availability Sets offer improved VM-to-VM failover latency reduction compared to availability zones. This is due to the VMs in an availability set being allocated in closer proximity. Availability Sets have fault isolation for many possible failures, minimize single points of failure, and offer high availability. However, Availability Sets are still susceptible to certain shared infrastructure failures, like datacenter network failures, which can affect multiple fault domains.

For greater reliability than the capabilities of Availability Sets, use [availability zones](availability.md#availability-zones). Availability Zones offer the highest reliability since each VM is deployed in multiple datacenters, protecting you from loss of either power, networking, or "cooling" in an individual datacenter. If your highest priority is the best reliability for your workload, replicate your VMs across multiple availability zones.

## How do availability sets work?
Each virtual machine in your availability set is assigned an **update domain** and a **fault domain** by the underlying Azure platform. Each Availability Set can be configured with up to 3 fault domains and 20 update domains. These configurations can't be changed once the Availability Set has been created. Update domains indicate groups of virtual machines and underlying physical hardware that can be rebooted at the same time. When more than five virtual machines are configured within a single availability set with five update domains, the sixth virtual machine is placed into the same update domain as the first virtual machine, the seventh in the same update domain as the second virtual machine, and so on. The order of update domains being rebooted may not proceed sequentially during planned maintenance, but only one update domain is rebooted at a time. A rebooted update domain is given 30 minutes to recover before maintenance is initiated on a different update domain.

Fault domains define the group of virtual machines that share a common power source and network switch. By default, the virtual machines configured within an Availability Set are separated across three fault domains (maximum). While placing your virtual machines into an availability set does not protect an application from operating system or application-specific failures, it does limit the impact of potential physical hardware failures, network outages, or power interruptions.

:::image type="content" source="./media/virtual-machines-common-manage-availability/ud-fd-configuration.png" alt-text="Diagram showing various compute clusters split into fault domains and within those fault domains, we have multiple update domains":::

VMs are also aligned with disk fault domains. This alignment ensures that all the managed disks attached to a VM are within the same fault domains. 

Only VMs with managed disks can be created in a managed availability set. The number of managed disk fault domains varies by region - either two or three managed disk fault domains per region. The following command retrieves a list of fault domains per region: 

```azurecli-interactive
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location, MaximumFaultDomainCount:capabilities[0].value}' -o Table
```

Under certain circumstances, two VMs in the same Availability Set might share a fault domain. You can confirm a shared fault domain by going to your availability set and checking the Fault Domain column. A shared fault domain might be caused by the completing following sequence when you deployed the VMs:
1. Deploy the first VM.
2. Stop/deallocate the first VM.
3. Deploy the second VM.

Under these circumstances, the OS disk of the second VM might be created on the same fault domain as the first VM, so the two VMs will be on same fault domain. To avoid this issue, we recommend that you don't stop/deallocate VMs between deployments.

:::image type="content" source="media/disks-high-availability/disks-availability-set.png" alt-text="Diagram of fault domain alignment with regional virtual machine scale sets and availability sets." lightbox="media/disks-high-availability/disks-availability-set.png":::

## Next steps
For best practices information, see [Azure availability best practices](/azure/architecture/checklist/resiliency-per-service).

