---
title: Associate a virtual machine scale set with Flexible Orchestration to a capacity reservation group
description: Learn how to associate a new virtual machine scale set with Flexible Orchestration mode to a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to
# Customer intent: As a cloud engineer, I want to associate a virtual machine scale set using Flexible Orchestration to a capacity reservation group, so that I can efficiently manage resource allocation for my deployed VMs and ensure optimal performance.
---

# Associate a virtual machine scale set with Flexible Orchestration to a capacity reservation group

**Applies to:** :heavy_check_mark: Flexible scale sets

Azure Virtual Machine Scale Sets has two modes:

- **Uniform Orchestration**: In this mode, virtual machine scale sets use a virtual machine (VM) profile or a template to scale up to the capacity you want. Although there's some ability to manage or customize individual VM instances, Uniform Orchestration uses identical VM instances. These instances are exposed through the virtual machine scale set's VM APIs. They aren't compatible with the API commands that are standard for Azure infrastructure as a service (IaaS) VMs. Because the scale set performs all the actual VM operations, reservations are associated to the virtual machine scale set directly. After the scale set is associated to the reservation, all the subsequent VM allocations are done against the reservation.
- **Flexible Orchestration**: In this mode, you get more flexibility to manage the individual instances of virtual machine scale set VMs. They can use the standard Azure IaaS VM APIs instead of using the scale set interface. To use reservations with Flexible Orchestration mode, define both the virtual machine scale set property and the capacity reservation property on each VM.

To learn more about these modes, see [Orchestration modes for Virtual Machine Scale Sets](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).

This content applies to the Flexible Orchestration mode. For more information about Uniform Orchestration mode, see [Associate a virtual machine scale set with Uniform Orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).

## Associate a new virtual machine scale set to a capacity reservation group

You have two options:

- **Add to the VM profile:** If the scale set with Flexible Orchestration includes a VM profile, add the capacity reservation group property to the profile during scale set creation. Follow the same process that's used for a scale set that uses Uniform Orchestration. For sample code, see [Associate a virtual machine scale set with Uniform Orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).
- **Add to the first deployed VM:** If the scale set omits a VM profile, you must add the capacity reservation group to the first VM that you deployed by using the virtual machine scale set. Follow the same process that's used to associate a VM. For sample code, see [Associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md).

## Associate an existing virtual machine scale set to a capacity reservation group

There are two steps:

- **Add to the virtual machine scale set:** For sample code, see [Associate a virtual machine scale set with Uniform Orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).
- **Add to the deployed VMs:** You must add the capacity reservation group to the VMs that you deployed by using the scale set, depending on the upgrade mode. Follow the same process used to associate a VM. For sample code, see [Associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md).

## Next step

> [!div class="nextstepaction"]
> [Learn how to remove a scale set association from a capacity reservation](capacity-reservation-remove-virtual-machine-scale-set.md)
