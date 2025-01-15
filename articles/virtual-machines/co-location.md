---
title: Proximity Placement Groups 
description: Learn about using Proximity Placement Groups in Azure.
author: mattmcinnes
ms.author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: proximity-placement-groups
ms.topic: conceptual
ms.date: 09/12/2022
---

# Proximity Placement Groups

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Placing VMs in a single region reduces the physical distance between the instances. Placing them within a single availability zone will also bring them physically closer together. However, as the Azure footprint grows, a single availability zone may span multiple physical data centers, which may result in a network latency impacting your application. 

To get VMs as close as possible, achieving the lowest possible latency, you should deploy them within a Proximity Placement Group.

A Proximity Placement Group is a logical grouping used to make sure that Azure compute resources are physically located close to each other. Proximity Placement Groups are useful for workloads where low latency is a requirement.

- Low latency between stand-alone VMs.
- Low Latency between VMs in a single availability set or a virtual machine scale set. 
- Low latency between stand-alone VMs, VMs in multiple Availability Sets, or multiple scale sets. You can have multiple compute resources in a single placement group to bring together a multi-tiered application. 
- Low latency between multiple application tiers using different hardware types. For example, running the backend using M-series in an availability set and the front end on a D-series instance, in a scale set, in a single Proximity Placement Group.

![Graphic for Proximity Placement Groups](./media/virtual-machines-common-ppg/ppg.png)

## Using Proximity Placement Groups 

A Proximity Placement Group is a resource in Azure. You need to create one before using it with other resources. Once created, it could be used with virtual machines, availability sets, or virtual machine scale sets. 
You specify a Proximity Placement Group when creating compute resources providing the Proximity Placement Group ID. 

You can also move an existing resource into a Proximity Placement Group. When moving a resource into a Proximity Placement Group, you should stop (deallocate) the asset first since it will be redeployed potentially into a different data center in the region to satisfy the colocation constraint. 

In the case of availability sets and virtual machine scale sets, you should set the Proximity Placement Group at the resource level rather than the individual virtual machines. 

A Proximity Placement Group is a colocation constraint rather than a pinning mechanism. It's pinned to a specific data center with the deployment of the first resource to use it. Once all resources using the Proximity Placement Group have been stopped (deallocated) or deleted, it's no longer pinned. Therefore, whenever you use a Proximity Placement Group with multiple VM series, it's important to specify all the required types upfront in a template if possible or follow a deployment sequence, which will improve your chances for a successful deployment. If your deployment fails, restart the deployment with the VM size, which has failed as the first size to be deployed.

## Use intent to specify VM sizes

You can use the optional `intent` parameter to provide the intended [VM Sizes](../virtual-machines/sizes.md) to be part of the Proximity Placement Group. This parameter can be specified at the time of creating a Proximity Placement Group or it can be added/modified while updating a Proximity Placement Group after deallocating all of the VMs. 

When specifying `intent`, you can also add the optional `zone` parameter to specify an availability zone, indicating that the Proximity Placement Group must be created within a specific availability zone. Note the following points when providing the `zone` parameter:

- The availability zone parameter can only be provided during the creation of the Proximity Placement Group and can't be modified later.
- The `zone` parameter can only be use with `intent`, it can't be used alone.
- Only one availability zone can be specified.

Proximity Placement Group creation or update will succeed only when at least one data center supports all the VM Sizes specified in the intent. Otherwise, the creation or update will fail with "OverconstrainedAllocationRequest", indicating that the combination of VM Sizes can't be supported within a Proximity Placement Group. The **intent does not provide any capacity reservation or guarantee**. The VM Sizes and zone  given in `intent` are used to select an appropriate data center, reducing the chances of failure if the desired VM size isn't available in a data center. Allocation failures can still occur if there is no more capacity for a VM size at the time of deployment. 

> [!NOTE]
> To use intent for your Proximity Placement Groups, ensure that the API version is 2021-11-01 or higher

### Best Practices while using intent

- Provide an availability zone for your Proximity Placement Group only when you provide an intent. Providing an availability zone without an intent will result in an error when creating the Proximity Placement Group.
- If you provide an availability zone in the intent, ensure that the availability zone of the VMs you deploy match with what that specified in the intent, to avoid errors while deploying VMs.
- Creating or adding VMs with sizes that are not included in the intent is allowed, but not recommended. The size may not exist in the selected datacenter and can result in failures at the time of VM deployment.
- For existing placement groups, we recommend you include the sizes of the existing VMs when updating the intent, in order to avoid failure when redeploying the VMs.

### Intent can be affected with decommissioning

- It is possible that after creating a Proximity Placement Group with intent and before deploying VMs, planned maintenance events such as hardware decommissioning at an Azure datacenter could occur, resulting in the combination of VM Sizes specified in the intent not being available in the data center. In such cases, an "OverconstrainedAllocationRequest" error will occur, even while deploying VMs of sizes specified in the intent. You can try deallocating all the resources in the Proximity Placement Group and recreate them to get a data center that can accommodate the intent. If there is no datacenter with the specified VM Sizes after the decommissioning, you may have to modify the intent to use a different combination of VM Sizes, since the combination of VM sizes is no longer supported.
- Azure may retire an entire VM family or a specific set of VM sizes. If you have such a VM size in the intent, you may have to either remove it or replace it with a different size before the retirement date for the original VM size. Otherwise, the intent will no longer be valid.

## What to expect when using Proximity Placement Groups 
Proximity Placement Groups offer colocation in the same data center. However, because Proximity Placement Groups represent an additional deployment constraint, allocation failures can occur. There are few use cases where you may see allocation failures when using Proximity Placement Groups:

- When you ask for the first virtual machine in the Proximity Placement Group, the data center is automatically selected. In some cases, a second request for a different VM size, may fail if it doesn't exist in that data center. In this case, an **OverconstrainedAllocationRequest** error is returned. To avoid this error, try changing the order in which you deploy your VM sizes or have both resources deployed using a single ARM template.
- If the Proximity Placement Group is created with intent, the VMs are not required to be deployed in any particular order and are not required to be batched using a single ARM template, since the intent is used to select a datacenter that supports all VM sizes indicated in the intent.
- In the case of elastic workloads, where you add and remove VM instances, having a Proximity Placement Group constraint on your deployment may result in a failure to satisfy the request resulting in **AllocationFailure** error.
- Stopping (deallocate) and starting your VMs as needed is another way to achieve elasticity. Since the capacity is not kept once you stop (deallocate) a VM, starting it again may result in an **AllocationFailure** error.
- VM start and redeploy operations will continue to respect the Proximity Placement Group once successfully configured.

## Planned maintenance and Proximity Placement Groups

Planned maintenance events, like hardware decommissioning at an Azure datacenter, could potentially affect the alignment of resources in Proximity Placement Groups. Resources may be moved to a different data center, disrupting the collocation and latency expectations associated with the Proximity Placement Group.

### Check the alignment status

You can do the following to check the alignment status of your Proximity Placement Groups.

- Proximity Placement Group colocation status can be viewed using the portal, CLI, and PowerShell.

    -   PowerShell - colocation status can be obtained through Get-AzProximityPlacementGroup cmdlet by including the optional parameter ‘-ColocationStatus`.

    -   CLI - colocation status can be obtained through `az ppg show` by including the optional parameter ‘--include-colocation-status`.

- For each Proximity Placement Group, a **colocation status** property
    provides the current alignment status summary of the grouped resources. 

    - **Aligned**: Resource is within the same latency envelop of the Proximity Placement Group.

    - **Unknown**: At least one of the VM resources are deallocated. After re-starting them successfully, the status should go back to **Aligned**.

    - **Not aligned**: At least one VM resource is not aligned with the Proximity Placement Group. The specific resources that are not aligned will also be called out separately in the membership section

- For Availability Sets, you can see information about alignment of individual VMs in the Availability Set Overview page.

- For scale sets, information about alignment of individual instances can be seen in the **Instances** tab of the **Overview** page for the scale set. 

### Realign resources 

If a Proximity Placement Group is `Not Aligned`, you can stop\deallocate, and then restart the affected resources. If the VM is in an availability set or a scale set, all VMs in the availability set or scale set must be stopped\deallocated first before restarting them.

If there is an allocation failure due to deployment constraints, you may have to stop\deallocate all resources in the affected Proximity Placement Group (including the aligned resources) first, and then restart them to restore alignment.

## Best practices 
- For the lowest latency, use Proximity Placement Groups together with accelerated networking. For more information, see [Create a Linux virtual machine with Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli) or [Create a Windows virtual machine with Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-powershell).
- In order to avoid landing on hardware that doesn't support all the VM SKUs and sizes you require, use intent for Proximity Placement Groups. If it is an already existing Proximity Placement Group without intent, you can use a single ARM template with all VM sizes specified to avoid this issue.
- When reusing an existing placement group from which VMs were deleted, wait for the deletion to fully complete before adding VMs to it.
- If latency is your first priority, put VMs in a Proximity Placement Group and the entire solution in an availability zone. But, if resiliency is your top priority, spread your instances across multiple availability zones (a single Proximity Placement Group cannot span zones).

## Next steps

- Deploy a VM to a Proximity Placement Group using the [Azure CLI](./linux/proximity-placement-groups.md) or [PowerShell](./windows/proximity-placement-groups.md).
- Learn how to [test network latency](/azure/virtual-network/virtual-network-test-latency).
- Learn how to [optimize network throughput](/azure/virtual-network/virtual-network-optimize-network-bandwidth).
- Learn how to [use Proximity Placement Groups with SAP applications](./workloads/sap/sap-proximity-placement-scenarios.md).
