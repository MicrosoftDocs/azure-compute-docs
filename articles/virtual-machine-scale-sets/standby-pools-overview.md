---
title: Standby pools for Virtual Machine Scale Sets
description: Learn how to utilize standby pools to reduce scale-out latency with Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: conceptual
ms.date: 5/6/2025
---

# Standby pools for Virtual Machine Scale Sets

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

Standby pools for Virtual Machine Scale Sets enables you to increase scaling performance by creating a pool of pre-provisioned virtual machines. The virtual machines in the standby pool complete all post provisioning processes such as installing applications, downloading data packages, etc. Once the virtual machines are fully provisioned, they are be maintained in a running, stopped (deallocated), or hibernated (preview) state. When your scale set requires more instances, the instances in the standby pool are automatically moved into the scale set. A standby pool significantly reduces the time it takes to scale-out a Virtual Machine Scale Set. 

If maintaining a standby pool of running virtual machines, the machines are immediately ready to receive traffic after being moved into the scale set. If maintaining a standby pool of stopped (deallocated) or hibernated virtual machines, the virtual machines are automatically started or resumed after moving into the scale set. 

## Provider Registration
Register the standby pool resource provider with your subscription using Azure Cloud Shell. Registration can take up to 30 minutes to successfully show as registered. You can rerun the below commands to determine when the feature is successfully registered. 

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.StandbyPool
```

## Scaling

Moving virtual machines between the standby pool into the scale set happens automatically whenever a scale-out event is triggered. There's no extra configuration required. As long as there's an available instance in the standby pool that has completed all provisioning steps, the scale set by default uses that instance when scaling up. 

When scaling back down, the instances are deleted from your scale set based on the [scale-in policy](virtual-machine-scale-sets-scale-in-policy.md) and the standby pool refills to meet the max ready capacity configured. If at any point in time your scale set needs to scale beyond the number of instances you have in your standby pool, the scale set defaults to standard scale-out methods and creates new instances.

Standby pools only give out virtual machines from the pool that match the desired power state configured. For example, if your desired power state is set as stopped (deallocated), the standby pool only gives the scale set instances matching that current power state. If virtual machines are in a creating, failed or any other state than the expected state the scale set defaults to new virtual machine creation instead.

## Standby pool size
There are three settings that determine how many instances are in your standby pool at any given point in time. This includes the scale set instance count, the minimum ready capacity and the maximum ready capacity. 

The **scale set instance count** is how many instances are currently deployed in your scale set. This is a scale set level property that can be changed at any point in time by either scaling up or scaling down. Regardless of how you're managing the scaling rules for your scale set, the standby pool keeps track of how many instances are deployed and adjust accordingly. 

The **minimum ready capacity** is a user defined parameter. By default, the minimum ready capacity for any new standby pool is zero. By setting the minimum ready capacity, it informs the standby pool that it should maintain that many instances at minimum. For example, if you have a minimum ready capacity of 5, anytime a virtual machine is moved from the pool into the scale set which reduces the minimum ready capacity to less than 5, the standby pool automatically creates another instance and begins preparing it for scale-out. 

The **maximum ready capacity** is a user defined parameter. This setting tells the standby pool how many instances at most should be maintained in the pool. Maximum ready capacity is directly tied to the scale set instance count. If you have a maximum ready capacity of 20 and you currently have 10 instances in your scale set, the pool size would equal 10. If your scale set scales down to 5, the pool size would increase to 15. This continues to dynamically adjust as the scale set increases and decreases instance count. 

| Setting | Description | 
|---|---|
| maxReadyCapacity | The maximum number of virtual machines to be created in the pool.|
| minReadyCapacity | The minimum number of virtual machines to be maintained in the pool.|
| instanceCount | The current number of virtual machines already deployed in the scale set.|
| Standby pool size | Standby pool size = `maxReadyCapacity`– `instanceCount` |

## Virtual machine states

The virtual machines in the standby pool can be kept in a running, hibernated, or stopped (deallocated) state. 

**Deallocated:** Deallocated virtual machines are shut down and keep any associated disks, network interfaces, and any static IPs. [Ephemeral OS disks](../virtual-machines/ephemeral-os-disks.md) don't support the deallocated state. 

:::image type="content" source="media/standby-pools/deallocated-vm-pool.png" alt-text="A screenshot showing the workflow when using deallocated virtual machine pools.":::

**Running:** Using virtual machines in a running state is recommended when latency and reliability requirements are strict. Virtual machines in a running state remain fully provisioned. 

:::image type="content" source="media/standby-pools/running-vm-pool.png" alt-text="A screenshot showing the workflow when using running virtual machine pools.":::

**Hibernated (Preview):** Using virtual machines in a hibernated state is recommended for systems with long boot times due to memory-intensive applications. These applications can be initialized on VMs and hibernated. These hibernated VMs can then be quickly started when needed, with the applications already up and running in the desired state.

> [!NOTE]
> To use the hibernated VM state in your standby pool, the instances must be fully eligible and configured to support hibernation. For more information, see the [Azure VM hibernation documentation](../virtual-machines/hibernate-resume.md).

:::image type="content" source="media/standby-pools/hibernated-vm-pool.png" alt-text="A screenshot showing the workflow when using hibernated virtual machine pools.":::


## Availability zones
When using standby pools with a Virtual Machine Scale Set spanning [availability zones](virtual-machine-scale-sets-use-availability-zones.md), the instances in the pool are spread across the same zones the Virtual Machine Scale Set is using. 

When a scale-out is triggered in one of the zones, a virtual machine in the pool in that same zone is used. If a virtual machine is needed in a zone where you no longer have any pooled virtual machines left, the scale set creates a new virtual machine directly in the scale set. 

## Pricing

Users are charged based on the resources deployed in the standby pool. For example, virtual machines in a running state incur compute, networking, and storage costs. Virtual machines in a deallocated state doesn't incur any compute costs, but any persistent disks or networking configurations continue incur cost. Thus, a pool of running virtual machines incur more cost than a pool of deallocated virtual machines. For more information on virtual machine billing, see [states and billing status of Azure Virtual Machines](../virtual-machines/states-billing.md).

## Limitations and considerations

When using standby pools with Virtual Machine Scale Sets, there are specific limitations and considerations to keep in mind:

- **Azure Autoscale**: Standby pools can't be created or attached to a Virtual Machine Scale Set with Azure autoscale enabled. Ensure autoscale is disabled before configuring a standby pool.
- **Fault Domains**: Standby pools aren't supported with Virtual Machine Scale Sets configured with a fault domain count greater than 1.
- **Region and Subscription**: The standby pool must be in the same region and subscription as the Virtual Machine Scale Set it's attached to. Cross-region or cross-subscription configurations aren't supported.
- **Single Standby Pool**: A Virtual Machine Scale Set can only have one standby pool attached at a time. Attaching multiple standby pools to the same scale set is not supported.
- **Orchestration Mode**: Standby pools are only supported with Virtual Machine Scale Sets using Flexible Orchestration. Uniform Orchestration is not supported.

## Next steps

Learn how to [create a standby pool](standby-pools-create.md).
