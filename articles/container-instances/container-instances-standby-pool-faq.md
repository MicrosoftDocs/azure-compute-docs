---
title: Frequently asked questions about standby pools for Azure Container Instances (Preview)
description: Get answers to frequently asked questions for standby pools for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.custom:
  - ignite-2024
ms.topic: how-to
ms.date: 11/6/2024
ms.reviewer: tomvcassidy
---

# Frequently asked questions about standby pools for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

Get answers to frequently asked questions about standby pools for Azure Container Instances. 

### How long can my standby pool name be? 
A standby pool can be anywhere between 3 and 24 characters. For more information, see [Resource naming restrictions for Azure resources](/azure/azure-resource-manager/management/resource-name-rules)

### How many container groups can my standby pool have?  
The maximum number of container groups in a standby pool is 2,000.  

### Can I change the size of my standby pool without needing to recreate it? 
Yes. To change the size of your standby pool, update the max ready capacity setting.  

### Can a standby pool resource be moved?
No. Standby pools don't currently support move capabilities. If you need to move a standby pool, you can consider deleting it recreating it in another location.

### I created a standby pool and I noticed that some containers are coming up in a failed state. 
Ensure you have enough quota to complete the standby pool creation. Insufficient quota results in the platform attempting to create the containers in the standby pool fail after encountering a quota error. Check for multiple types of quotas such as Cores, Network Interfaces, IP Addresses, etc.

### I requested a container from my pool but it created a new container instead. 
Ensure that the container groups in your standby pool are in the running state before issuing a request. For example, if using a standby If containers are in any other states such as creating or deleting, the container request defaults to creating a new container group from scratch. 

### Can I create a standby pool with containers spread across availability zones? 
Yes. However, standby pools for Azure Container Instances only currently support creating and requesting containers across availability zones using [REST APIs](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update) using version 2024-08-01.  


## Next steps

Learn more about [standby pools on Azure Container Instances](container-instances-standby-pool-overview.md).
