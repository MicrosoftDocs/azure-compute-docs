---
title: Use Spot Instances in standby pools (Preview)
description: Learn how to configure and use Spot Instances with standby pools in Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 5/6/2025
ms.reviewer: cynthn
# Customer intent: As a cloud architect, I want to configure Virtual Machine Scale Sets with Spot Instances in standby pools, so that I can optimize costs while ensuring scalability and resource availability for my applications.
---

# Use Spot Instances in standby pools (Preview)

> [!IMPORTANT]
> Using spot instances in standby pools for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

Azure Spot Instances allow you to take advantage of unused Azure capacity at a significant cost savings. By combining Spot Instances with standby pools in Virtual Machine Scale Sets, you can optimize costs while maintaining scalability. However, there are specific considerations and limitations when using Spot Instances with standby pools:

- Standby pools only support spot instances when used with scale sets configured to use 100% Spot Instances (not a mix of Spot and regular instances).
- Standby pools only support spot instances when used with a scale set that has the spot eviction policy set as delete. Deallocate eviction policy isn't currently supported.
- Standby pools using spot instances can't use a hibernated VM state. Only running and deallocated pool state are supported.

This article explains how to configure and use Spot Instances with standby pools, including details about supported VM states and their behavior.

## Supported VM states for Spot Instances in standby pools

When using Spot Instances with standby pools, you can configure the pool to use either a **running** or **deallocated** VM state. Hibernate isn't supported when using spot instances. Each state has different behaviors:

### Running state
- The virtual machines in the standby pool remain in a running state.
- If a VM in the pool is evicted due to Spot capacity constraints, it is deleted and replaced with a new instance.
- This configuration ensures that the pool is always ready to provide running instances to the scale set as long as you have a minimum ready capacity configured or have not reached the max ready capacity configured on your standby pool. 

### Deallocated state
- The Spot Instances in the standby pool complete provisioning and are then shut down (deallocated).
- When the scale set requires new instances, they're automatically pulled from the pool and started in the scale set.
- This configuration reduces costs by releasing the compute resources associated with the virtual machines after they have finished all post provisioning steps. 

> [!IMPORTANT]
> There are no guarantees or SLAs when using Spot Instances in standby pools. Spot Instances are subject to eviction based on Azure capacity availability.

## Configure a Virtual Machine Scale Set with Spot Instances

> [!NOTE]
> Adding a standby pool to a Virtual Machine Scale Set using spot instances in the Azure portal isn't yet supported. Instead, create your scale set using spot instances, then use an alternative SDK to add the standby pool after scale set creation. 

To use Spot Instances with standby pools, you must configure your scale set to use 100% Spot Instances and set the eviction policy to delete. 

  - **Instance type**: Select a Spot Instance type.
  - **Eviction policy**: Set the eviction policy to **Delete**.
  - **Spot allocation**: Ensure the scale set is configured to use 100% Spot Instances.

Once your scale set is configured with Spot Instances, there is no additional configurations required to enable spot instnaces in the standby pool. Create and attach the standby pool to your scale set with spot instances and the instances within the pool takes on the properties configured in your scale set.

## Next steps

Configure event monitoring and alertings using [Log Analytics](standby-pools-monitor-pool-events.md).
