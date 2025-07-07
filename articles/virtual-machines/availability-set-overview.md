---
title: Availability sets overview 
description: Learn about availability sets for virtual machines in Azure.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 10/10/2024
ms.custom: engagement-fy23
# Customer intent: As a cloud infrastructure architect, I want to understand how availability sets work, so that I can effectively manage virtual machine deployments to ensure high availability and reduce the risk of correlated failures in my applications.
---

# Availability sets overview

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs

This article provides an overview of the availability features of Azure virtual machines (VMs).

> [!NOTE]
> We recommend that customers choose [Virtual Machine Scale Sets with flexible orchestration mode](../virtual-machine-scale-sets/overview.md) for high availability with the widest range of features. Virtual Machine Scale Sets:
>
> - Allow VM instances to be centrally managed, configured, and updated.
> - Automatically increase or decrease the number of VM instances in response to demand or a defined schedule.
>
> Availability sets offer only high availability.

## What is an availability set?

Availability sets are logical groupings of VMs that reduce the chance of correlated failures bringing down related VMs at the same time. Availability sets place VMs in different fault domains for better reliability. This action is especially beneficial if a region doesn't support availability zones.

When you use availability sets, create two or more VMs within an availability set. Using two or more VMs in an availability set helps keep applications highly available and meets the 99.95% Azure service-level agreement (SLA). There's no extra cost for using availability sets. You only pay for each VM instance that you create.

Availability sets offer improved VM-to-VM latencies compared to availability zones, because VMs in an availability set are allocated in closer proximity. Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability. Availability sets are still susceptible to certain shared infrastructure failures, like datacenter network failures, which can affect multiple fault domains.

For more reliability than availability sets offer, use [availability zones](availability.md#availability-zones). Availability zones have the highest reliability. Each VM is deployed in multiple datacenters to help protect you from loss of power, networking, or cooling in an individual datacenter. If your highest priority is the best reliability for your workload, replicate your VMs across multiple availability zones.

## How do availability sets work?

The underlying Azure platform assigns an *update domain* and a *fault domain* to each virtual machine in your availability set. Each availability set can have up to 3 fault domains and 20 update domains. You can't change these configurations after you create the availability set.

### Update domains

Update domains indicate groups of virtual machines and underlying physical hardware that can be restarted at the same time.

When more than five virtual machines are configured within a single availability set with five update domains, the sixth virtual machine is placed into the same update domain as the first virtual machine. The seventh virtual machine is placed in the same update domain as the second virtual machine. And the sequence continues.

The order of update domains being restarted might not proceed sequentially during planned maintenance, but only one update domain is restarted at a time. A restarted update domain has 30 minutes to recover before maintenance starts on a different update domain.

### Fault domains

Fault domains define the group of virtual machines that share a common power source and network switch. By default, the virtual machines configured within your availability set are separated across up to three fault domains.

Placing your virtual machines into an availability set doesn't protect your application from operating system or application-specific failures. But it does limit the impact of potential physical hardware failures, network outages, or power interruptions.

:::image type="content" source="./media/virtual-machines-common-manage-availability/ud-fd-configuration.png" alt-text="Diagram that shows compute clusters split into fault domains that contain update domains.":::

### Disk fault domains

VMs are also aligned with disk fault domains. This alignment ensures that all the managed disks attached to a VM are within the same fault domains.

Only VMs with managed disks can be created in a managed availability set. The number of managed-disk fault domains varies by region: either two or three managed-disk fault domains per region.

The following command retrieves a list of fault domains per region:

```azurecli-interactive
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location, MaximumFaultDomainCount:capabilities[0].value}' -o Table
```

### Shared fault domains

Under certain circumstances, two VMs in the same availability set might share a fault domain. You can confirm a shared fault domain by going to your availability set and checking the **Fault Domain** column.

If your VMs have a shared fault domain, it could be because you took these actions when deploying your VMs:

1. Deploy the first VM.
2. Stop or deallocate the first VM.
3. Deploy the second VM.

Under these circumstances, the OS disk of the second VM might be created on the same fault domain as the first VM, so the two VMs will be on same fault domain. To avoid this problem, don't stop or deallocate VMs between deployments.

:::image type="content" source="media/disks-high-availability/disks-availability-set.png" alt-text="Diagram of fault domain alignment with regional virtual machine scale sets and availability sets." lightbox="media/disks-high-availability/disks-availability-set.png":::

## Related content

- For best practices related to Azure availability, see [Resiliency checklist for specific Azure services](/azure/architecture/checklist/resiliency-per-service).
