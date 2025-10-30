---
title: On-demand capacity reservation in Azure
description: Learn how to reserve compute capacity in an Azure region or an availability zone with capacity reservation.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: overview
ms.date: 02/24/2023
ms.reviewer: cynthn, jushiman, mattmcinnes
ms.custom: template-how-to
# Customer intent: "As an IT administrator, I want to create and manage on-demand capacity reservations in Azure, so that I can ensure the availability of compute resources for critical workloads without long-term commitments."
---

# On-demand capacity reservation

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

On-demand capacity reservation enables you to reserve compute capacity in an Azure region or an availability zone for any duration of time. Unlike [reserved instances](https://azure.microsoft.com/pricing/reserved-vm-instances/), you don't have to sign up for a one-year or three-year term commitment. You can create and delete reservations at any time and have full control over how you want to manage your reservations.

After you create the capacity reservation, you can use the resources immediately. Capacity is reserved for you until you delete the reservation.

Capacity reservation has some basic properties that are always defined at the time of creation:

- **VM size**: Each reservation is for one virtual machine (VM) size. An example is `Standard_D2s_v3`.
- **Location**: Each reservation is for one location (region). If that location has availability zones, the reservation can also specify one of the zones.
- **Quantity**: Each reservation has a quantity of instances to be reserved.

To create a capacity reservation, the parameters are passed to Azure as a capacity request. If Azure doesn't have capacity available that meets the request, the reservation deployment fails. Your deployment fails if you don't have an adequate subscription quota. Request a higher quota or try a different VM size, location, or zone combination. 

After Azure accepts your reservation request, it's available for VMs with matching configurations. To consume capacity reservation, the VM has to specify the reservation in its properties. Otherwise, the capacity reservation isn't used. One benefit of this design is that you can target only critical workloads to reservations and other noncritical workloads can run without reserved capacity. 

## Benefits of capacity reservation 

- After deployment, capacity is reserved for your use and is always available within the scope of applicable service-level agreements (SLAs).
- Capacity can be deployed and deleted at any time with no term commitment.
- Capacity can be combined automatically with reserved instances to use term-commitment discounts.

## SLA for capacity reservation 

Read the SLA details in the [SLA for capacity reservation](https://aka.ms/CapacityReservationSLAForVM). 

Any claim against the SLA requires that you calculate the Minutes Not Available for the reserved capacity. Here's an example of how to calculate Minutes Not Available:

- An on-demand capacity reservation has a total capacity of five reserved units. The on-demand capacity reservation starts in the Unused Capacity state with zero VMs allocated. 
- A supported deployment with a quantity of 5 is allocated to the on-demand capacity reservation. Three VMs succeed and two fail with a VM capacity error. The result is that two reserved units begin to accumulate Minutes Not Available. 
- No action is taken for 20 minutes. The result is that two reserved units each accumulate 15 Minutes Not Available. 
- At 20 minutes, a supported deployment with a quantity of 2 is attempted. One VM succeeds and the other VM fails with a VM capacity error. The result is one reserved unit stays at 15 accumulated Minutes Not Available. Another reserved unit resumes accumulating Minutes Not Available.
- Four more supported deployments with a quantity of 1 are made at 10-minute intervals. On the fourth attempt (60 minutes after the first capacity error), the VM is deployed. The result is that the last reserved unit adds 40 minutes of Minutes Not Available (four attempts x 10 minutes between attempts) for a total of 55 Minutes Not Available.

From this example accumulation of Minutes Not Available, here's the calculation of service credit:

- One reserved unit accumulated 15 minutes of downtime. The percentage uptime is 99.97%. This reserved unit doesn't qualify for a service credit.
- Another reserved unit accumulated 55 minutes of downtime. The percentage uptime is 99.87. This reserved unit qualifies for a service credit of 10%.

## Limitations and restrictions

- Creating capacity reservations requires a quota in the same manner as when you create VMs.
- Creating capacity reservations is currently limited to certain VM series and sizes. The compute [Resource SKUs list](/rest/api/compute/resource-skus/list) advertises the set of supported VM sizes.
- The following VM series support the creation of capacity reservations:

  Reserved capacity for these VM series supports high availability using multi-zone or fault domains. The fault domain count for a Virtual Machine Scale Set is limited to 3, enforced at deployment.  

  | Type | VM Series |
  | ----- | ----------- |
  | General Purpose (Burstable) | B (Intel) <br> Bsv2 (Intel), Basv2 (AMD), Bpsv2 (ARM) |
  | General Purpose (General use A, D) | Av2 (Intel) <br> D and Ds series, v2 and newer (Intel) <br> Dd and Dds, v4 and newer (Intel, local disk) <br> Dls and Dlds, v5 and newer (Intel, low memory) <br> Da and Das, v4 series and newer (AMD) <br> Dads, v5 and newer (AMD, local disk) <br> Dalsv6 and Daldsv6 (AMD) <br> Dps and Dpds series, v5 and newer (ARM) <br> Dpls and Dplds series, v5 and newer (ARM, low memory) |
  | General Purpose (General use, high memory, E )| E and Es, all versions (Intel) <br> Ed and Eds series, v4 and newer (Intel, local disk) <br> Ebdsv5 and Ebsv5 series (Intel, block storage) <br> Ea and Eas, v4 and newer (AMD) <br> Eads, v5 series (AMD, local disk) <br> Eps and Epds series, v5 and newer (ARM) |
  | General Purpose (General use, compute optimized, F) | Fsv2 series (Intel) <br> Fasv6 and Falsv6 series (AMD)  <br> Famsv6 (AMD) <br> FX series (Intel) <br> FXmsv2 (Intel) <br> FXmdsv2 (Intel) |
  | Storage optimized| Lsv2 (Intel)-Preview <br> Lsv3 (Intel) <br> Lasv3 (AMD) |
  | Confidential compute | DCsv2 series (Intel) <br> DCasv5 and DCadsv5 series (AMD) <br> DCesv5 and DCedsv5 series (Intel) <br> ECasv5 and ECadsv5 series (AMD) <br> ECesv5 and ECedsv5 series (Intel) |

  Reserved capacity for these VM series supports only multi-zone high availability. Virtual Machine Scale Sets with a fault domain availability construct is not supported. 

  | Type | VM Series |
  | ----- | ----------- |
  | GPU-accelerated compute | NC-series, v3 (Intel) <br> NCasT4_v3 series (AMD) <br> NCADSA10_v4 series (AMD) <br> NC_A100_v4 series (AMD) <br> NV-series, v3 and newer (Intel) <br> NVadsA10_v5 series (AMD) <br> NVads V710 v5 series (AMD) <br> NGads V620_v1 series (AMD) |
  | Memory optimized | Msv2 Medium Memory series (Intel) <br> Mdsv2 Medium Memory series (Intel) <br> Mv2 (Intel) <br>Mdsv3 Medium Memory series (Intel) <br> Msv3 Medium Memory series (Intel) <br> Mbsv3 and Mbdsv3 series (Intel) |

- Support for other VM series isn't currently available:
  
  | Type | VM Series |
  | ----- | ----------- |
  | GPU-accelerated compute | ND-series <br> Hb-series <br> Hc-series |
  | Memory optimized | M series, v1 <br> Msv3 HM <br> Mdsv3 HM and Mdsv3 VHM |
  
- The following deployment types are supported: 
    - Single VM
    - Virtual Machine Scale Sets with Uniform Orchestration
    - Virtual Machine Scale Sets with Flexible Orchestration
- The following deployment types aren't supported: 
    - Spot VMs 
    - Azure Dedicated Host nodes or VMs deployed to dedicated hosts 
    - Availability sets 
- Other deployment constraints aren't supported. For example: 
    - Proximity placement group 
    - Update domains 
    - Virtual Machine Scale Sets with single placement group set to `true` 
    - Azure Ultra Disk Storage (formerly UltraSSD)
    - VMs resuming from hibernation 
- A pinned subscription can't use the feature.
- Deployments using On demand capacity reservations are compatible with vNet Encryption when deployed using a supported VM size. For more information on which VM sizes this feature supports, see [Virtual Network Encryption](/azure/virtual-network/virtual-network-encryption-overview).
- Reservations are only available to paid Azure customers. Sponsored accounts such as Free Trial and Azure for Students aren't eligible to use this feature.
- Clouds supported for capacity reservation:
   - Azure Cloud
   - Azure for Government
   - Azure in China

## Pricing and billing 

Capacity reservations are priced at the same rate as the underlying VM size. For example, if you create a reservation for 10 D2s_v3 VMs, you start getting billed for 10 D2s_v3 VMs, even if the reservation isn't being used.

If you then deploy a D2s_v3 VM and specify the reservation property, the capacity reservation gets used. After the VM is in use, you pay for only the VM and not the capacity reservation. Let's say you deploy six D2s_v3 VMs against the previously mentioned capacity reservation. You see a bill for six D2s_v3 VMs and four unused capacity reservations, both charged at the same rate as a D2s_v3 VM.

Both used and unused capacity reservations are eligible for Savings Plan and Reserved Instances term commitment discounts. In the previous example, if you have reserved instances for two D2s_v3 VMs in the same Azure region, the billing for two resources (either VM or unused capacity reservation) is zeroed out. The remaining eight D2s_v3 are billed normally. The term commitment discounts could be applied on either the VM or the unused capacity reservation.

## Difference between on-demand capacity reservation and reserved instances


| Differences | On-demand capacity reservation | Reserved instances|
|---|---|---|
| Term | No term commitment required. Can be created and deleted as per the customer requirement. | Fixed-term commitment of either one year or three years.|
| Billing discount | Charged at pay-as-you-go rates for the underlying VM size.* | Significant cost savings over pay-as-you-go rates. |
| Capacity SLA | Provides capacity guarantee in the specified location (region or availability zone). | Doesn't provide a capacity guarantee. Customers can choose **Capacity priority** to gain better access, but that option doesn't carry an SLA. |
| Region vs. availability zones | Can be deployed per region or per availability zone. | Only available at the regional level. |

*Eligible for the reserved instances discount if purchased separately.

## Work with capacity reservation

Capacity reservation is created for a specific VM size in an Azure region or an availability zone. All reservations are created and managed as part of a capacity reservation group. 

The group specifies the Azure location:

- The group sets the region in which all reservations are created. Examples are East US, North Europe, or Southeast Asia. 
- The group sets the eligible zones. Examples are AZ1, AZ2, and AZ3 in any combination. 
- If no zones are specified, then Azure will select the best zone available upon initial capacity reservation creation. All VMs using the capacity reservation group must be deployed without specifying a zone; each associated VM wil be deployed to the zone Azure selected for the group.

Each reservation in a group is for one VM size. If eligible zones were selected for the group, the reservation must be for one of the supported zones. 

A group can have only one reservation per VM size per zone, or just one reservation per VM size if no zones are selected. 

To consume capacity reservation, specify the capacity reservation group as one of the VM properties. If the group doesn't have a reservation matching the size and location, Azure returns an error message. 

You can adjust the quantity reserved for reservation after initial deployment by changing the capacity property. Other changes to capacity reservation, such as VM size or location, aren't permitted. We recommend that you create a new reservation, migrate any existing VMs, and then delete the old reservation if it's no longer needed.

Capacity reservation doesn't create limits on the number of VM deployments. Azure supports allocating as many VMs as desired against the reservation. Because the reservation itself requires a quota, the quota checks are omitted for VM deployment up to the reserved quantity. Allocating VMs beyond the reserved quantity is called overallocating the reservation. Overallocating VMs isn't covered by the SLA, and the VMs are subject to quota checks and Azure fulfilling the extra capacity. After they're deployed, these extra VM instances can cause the quantity of VMs allocated against the reservation to exceed the reserved quantity. To learn more, see [Overallocate capacity reservation](capacity-reservation-overallocate.md).

## Capacity reservation lifecycle

When a reservation is created, Azure sets aside the requested number of capacity instances in the specified location. 

![Diagram that shows the requested number of capacity instances in a location.](./media/capacity-reservation-overview/capacity-reservation-1.jpg) 

Track the state of the overall reservation through the following properties: 
 
- `capacity`: Total quantity of instances reserved by the customer. 
- `virtualMachinesAllocated`: List of VMs allocated against the capacity reservation and count toward consuming the capacity. These VMs are either **Running** or **Stopped** (**Allocated**), or they're in a transitional state such as **Starting** or **Stopping**. This list doesn't include the VMs that are in a deallocated state, which are referred to as **Stopped** (deallocated). 
- `virtualMachinesAssociated`: List of VMs associated to the capacity reservation. This list has all the VMs that were configured to use the reservation, including the ones that are in a deallocated state.

The previous example starts with `capacity` as 2 and the length of `virtualMachinesAllocated` and `virtualMachinesAssociated` as 0.

When a VM is then allocated against the capacity reservation, it consumes one of the reserved capacity instances.

![Diagram that shows one of the reserved capacity instances consumed.](./media/capacity-reservation-overview/capacity-reservation-2.jpg) 

The status of the capacity reservation shows `capacity` as 2 and the length of `virtualMachinesAllocated` and `virtualMachinesAssociated` as 1.

Allocations against the capacity reservation succeed if the VMs have matching properties and there's at least one empty capacity instance.

As shown in our example, when a third VM is allocated against the capacity reservation, the reservation enters the [overallocated](capacity-reservation-overallocate.md) state. This third VM requires unused quota and extra capacity fulfillment from Azure. After the third VM is allocated, the capacity reservation now looks like this diagram: 

![Diagram that shows capacity reservation with the third VM allocated.](./media/capacity-reservation-overview/capacity-reservation-3.jpg) 

The `capacity` is 2 and the length of `virtualMachinesAllocated` and `virtualMachinesAssociated` is 3. 

Now suppose the application scales down to the minimum of two VMs. Because the VM 0 needs an update, it's chosen for deallocation. The reservation automatically shifts to this state: 

![Diagram that shows capacity reservation scaled down to the minimum of two VMs.](./media/capacity-reservation-overview/capacity-reservation-4.jpg) 

The `capacity` and the length of `virtualMachinesAllocated` are both 2. However, the length for `virtualMachinesAssociated` is still 3 because VM 0, although deallocated, is still associated to the capacity reservation. To prevent a quota overrun, the deallocated VM 0 still counts against the quota allocated to the reservation. If you have enough unused quota, you can deploy new VMs to the capacity reservation and receive the SLA from any unused reserved capacity. Or you can delete VM 0 to remove its use of the quota. 

The capacity reservation exists until it's explicitly deleted. To delete a capacity reservation, the first step is to dissociate all the VMs in the `virtualMachinesAssociated` property. After disassociation is finished, the capacity reservation should look like this diagram: 

![Diagram that shows capacity reservation after disassociation is finished.](./media/capacity-reservation-overview/capacity-reservation-5.jpg) 

The status of the capacity reservation shows `capacity` as 2 and the length of `virtualMachinesAssociated` and `virtualMachinesAllocated` as 0. From this state, you can delete the capacity reservation. After it's deleted, you don't pay for the reservation anymore.

![Diagram that shows capacity reservation deleted.](./media/capacity-reservation-overview/capacity-reservation-6.jpg)

## Usage and billing 

When a capacity reservation is empty, VM usage is reported for the corresponding VM size and the location. [VM reserved instances](https://azure.microsoft.com/pricing/reserved-vm-instances/) can cover some or all of the capacity reservation usage even when VMs aren't deployed. 

### Example

For example, let's say that a capacity reservation with the reserved quantity of 2 was created. The subscription has access to one matching reserved VM instance of the same size. The result is two usage streams for the capacity reservation, one of which is covered by the reserved instance. 

![Diagram that shows capacity reservation with two usage streams.](./media/capacity-reservation-overview/capacity-reservation-7.jpg)

In the previous diagram, a reserved VM instance discount is applied to one of the unused instances and the cost for that instance is zeroed out. For the other instance, the pay-as-you-go rate is charged for the VM size reserved.

When a VM is allocated against the capacity reservation, the other VM components such as disks, network, extensions, and any other requested components must also be allocated. In this state, the VM usage reflects one allocated VM and one unused capacity instance. The reserved VM instance will zero out the cost of either the VM or the unused capacity instance. The other charges for disks, networking, and other components associated to the allocated VM also appear on the bill. 

![Diagram that shows one allocated VM and one unused capacity instance.](./media/capacity-reservation-overview/capacity-reservation-8.jpg)

In the previous image, the VM reserved instance discount is applied to VM 0, which is only charged for other components, such as disks and networking. The other unused instance is being charged at a pay-as-you-go rate for the VM size reserved.

## Frequently asked questions 

- **What's the price of on-demand capacity reservation?**

    The price of your on-demand capacity reservation is the same as the price of the underlying VM size associated to the reservation. When you use capacity reservation, you're charged for the VM size you selected at pay-as-you-go rates, whether the VM was provisioned or not. For more information, see the [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) and [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) VM pricing pages. 

- **Will I get charged twice for the cost of on-demand capacity reservation and for the actual VM when I finally provision it?**

    No, you only get charged once for on-demand capacity reservation. 

- **Can I apply Azure Reserved Virtual Machine Instances to on-demand capacity reservations to lower my costs?**

    Yes, you can apply existing or future reserved instances to on-demand capacity reservations and receive reserved instance discounts. Available reserved instances are applied automatically to capacity reservations the same way they're applied to VMs.

- **What's the difference between Reserved Virtual Machine Instances and on-demand capacity reservations?**

    Both reserved instances and on-demand capacity reservations are applicable to Azure VMs. However, reserved instances provide discounted reservation rates for your VMs compared to pay-as-you-go rates as a result of a one-year or three-year term commitment. Conversely, on-demand capacity reservations don't require a commitment. 

    You can create or cancel a capacity reservation at any time. However, no discounts are applied, and you incur charges at pay-as-you-go rates after your capacity reservation is successfully provisioned. Unlike reserved instances, which prioritize capacity but don't guarantee it, when you purchase an on-demand capacity reservation, Azure sets aside compute capacity for your VM and provides an SLA guarantee.

- **Which scenarios would benefit the most from on-demand capacity reservations?**

    Typical scenarios include business continuity, disaster recovery, and scale-out of mission-critical applications.

## Related content

Get started reserving compute capacity. Check out other capacity reservation articles:

- [Create a capacity reservation](capacity-reservation-create.md)
- [Overallocating capacity reservation](capacity-reservation-overallocate.md)
- [Modify a capacity reservation](capacity-reservation-modify.md)
- [Associate a VM](capacity-reservation-associate-vm.md)
- [Remove a VM](capacity-reservation-remove-vm.md)
- [Associate a virtual machine scale set - Flexible Orchestration](capacity-reservation-associate-virtual-machine-scale-set-flex.md)
- [Associate a virtual machine scale set - Uniform Orchestration](capacity-reservation-associate-virtual-machine-scale-set.md)
- [Remove a virtual machine scale set](capacity-reservation-remove-virtual-machine-scale-set.md)
