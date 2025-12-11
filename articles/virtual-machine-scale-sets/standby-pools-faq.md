---
title: Frequently asked questions about standby pools for Virtual Machine Scale Sets
description: Get answers to frequently asked questions for standby pools on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: cynthn
# Customer intent: As a cloud infrastructure administrator, I want to understand the configurations and limitations of standby pools for Virtual Machine Scale Sets, so that I can effectively manage resource allocation and ensure optimal capacity in my environment.
---

# Standby pools FAQ 

> [!IMPORTANT]
> For standby pools to successfully create and manage resources, it requires access to the associated resources in your subscription. Ensure the correct permissions are assigned to the standby pool resource provider in order for your standby pool to function properly. For detailed instructions, see **[configure role permissions for standby pools](standby-pools-configure-permissions.md)**.

Get answers to frequently asked questions about standby pools for Virtual Machine Scale Sets in Azure.

### Can I use standby pools on Virtual Machine Scale Sets with Uniform Orchestration?
Standby pools are only supported on Virtual Machine Scale Sets with Flexible Orchestration.

### Does using a standby pool guarantee capacity? 
Using a standby pool with deallocated instances doesn't guarantee capacity. When starting the deallocated virtual machine, there needs to be enough capacity in the region your instances are deployed in to start the machines. If using running virtual machines in your pool, those virtual machines are already allocated and consuming compute capacity. When the virtual machine moves from the standby pool to the Virtual Machine Scale Set, it doesn't release the compute resources and doesn't require any additional allocation of resources. 

### How long can my standby pool name be? 
A standby pool can be anywhere between 3 and 24 characters. For more information, see [Resource naming restrictions for Azure resources](/azure/azure-resource-manager/management/resource-name-rules)

### How many virtual machines can my standby pool for Virtual Machine Scale Sets have? 
The maximum number of virtual machines between a scale set and a standby pool is 1,000. 

### Can my standby pool span multiple Virtual Machine Scale Sets? 
A standby pool resource can't span multiple scale sets. Each scale set has its own standby pool attached to it. A standby pool inherits the unique properties of the scale set such as networking, virtual machine profile, extensions, and more. 


### Can a standby pool resource be moved?
No. Standby Pools doesn't currently support that capability. If you need to move a standby pool, you can consider deleting it recreating it in another location.

### I created a standby pool and I noticed that some virtual machines are coming up in a failed state. 
Ensure you have enough quota to complete the standby pool creation. Insufficient quota results in the platform attempting to create the virtual machines in the standby pool but unable to successfully complete the create operation. Check for multiple types of quotas such as Cores, Network Interfaces, IP Addresses, etc.

### I increased my scale set instance count but the virtual machines in my standby pool weren't used. 
Ensure that the virtual machines in your standby pool are in the desired state prior to attempting a scale-out event. For example, if using a standby pool with the virtual machine states set to deallocated, the standby pool will only give out instances that are in the deallocated state. If instances are in any other states such as creating, running, updating, etc., the scale set will default to creating a new instance directly in the scale set.

### I tried to create a standby pool but I got an error regarding not having the correct permissions. 

Review [configure standby pool permissions](standby-pools-configure-permissions.md) for detailed steps on configuring the right permissions for the standby pool resource provider. 

## Next steps

Learn more about [standby pools on Virtual Machine Scale Sets](standby-pools-overview.md).
