---
title: Frequently asked questions about standby pools for Virtual Machine Scale Sets
description: Get answers to frequently asked questions for standby pools on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/5/2024
ms.reviewer: ju-shim
---

# Standby pools FAQ 

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

To allow standby pools to create and manage virtual machines in your subscription, assign the appropriate permissions to the standby pool resource provider. 

1) In the Azure portal, navigate to your subscriptions.
2) Select the subscription you want to adjust permissions.
3) Select **Access Control (IAM)**.
4) Select **Add** and **Add role assignment**.
5) Under the **Role** tab, search for **Virtual Machine Contributor** and select it.
6) Move to the **Members** Tab.
7) Select **+ Select members**.
8) Search for **Standby Pool Resource Provider** and select it.
9) Move to the **Review + assign** tab.
10) Apply the changes. 
11) Repeat the above steps and assign the **Network Contributor** role and the **Managed Identity Operator** role to the Standby Pool Resource Provider. If you're using images stored in Compute Gallery assign the **Compute Gallery Sharing Admin** and **Compute Gallery Artifacts Publisher** roles as well.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).


### I do not see the option to create and manage standby pools in the Azure portal. 

To create and manage standby pools in the Azure portal, register the following feature flag:
`Register-AzProviderFeature -FeatureName StandbyVMPoolPreview -ProviderNamespace Microsoft.StandbyPool`

## Next steps

Learn more about [standby pools on Virtual Machine Scale Sets](standby-pools-overview.md).
