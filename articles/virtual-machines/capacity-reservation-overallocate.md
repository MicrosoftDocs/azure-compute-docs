---
title: Overallocate capacity reservation in Azure
description: Learn how overallocation works when it comes to capacity reservation.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/24/2023
ms.reviewer: cynthn, jushiman, mattmcinnes
ms.custom: template-how-to
# Customer intent: "As a cloud administrator, I want to understand how overallocation of capacity reservations works, so that I can optimize the use of virtual machines while managing capacity and service-level agreements effectively."
---

  # Overallocate capacity reservation

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

Azure permits the association of extra virtual machines (VMs) above the number of capacity reservations. These VMs are available to allow for burst and other scale-out scenarios without the limits of reserved capacity. The only difference is that the count of VMs beyond the quantity reserved doesn't receive the capacity availability service-level agreement (SLA) benefit. As long as Azure has available capacity that meets the VM requirements, the extra allocation succeeds.

The Instance View of a capacity reservation group provides a snapshot of usage for each member capacity reservation. You can use the Instance View to see how overallocation works.

This article assumes you created a capacity reservation group (`myCapacityReservationGroup`), a member capacity reservation (`myCapacityReservation`), and a VM (*myVM1*) that's associated to the group. For more information, see [Create a capacity reservation](capacity-reservation-create.md) and [Associate a VM to a capacity reservation](capacity-reservation-associate-vm.md).

## Instance View for a capacity reservation group

The Instance View for a capacity reservation group looks like this example:

```rest
GET 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/myCapacityReservationGroup?$expand=instanceview&api-version=2021-04-01
```

```json
{ 
    "name": "myCapacityReservationGroup", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup", 
    "type": "Microsoft.Compute/capacityReservationGroups", 
    "location": "eastus", 
    "properties": { 
        "capacityReservations": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/MYCAPACITYRESERVATIONGROUP/capacityReservations/MYCAPACITYRESERVATION" 
            } 
        ], 
        "virtualMachinesAssociated": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM1" 
            } 
        ], 
        "instanceView": { 
            "capacityReservations": [ 
                { 
                    "name": "myCapacityReservation", 
"utilizationInfo": { 
                        "virtualMachinesAllocated": [ 
                            { 
                                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM1" 
                            } 
                        ] 
                    }, 
                    "statuses": [ 
                        { 
                            "code": "ProvisioningState/succeeded", 
                            "level": "Info", 
                            "displayStatus": "Provisioning succeeded", 
                            "time": "<time>" 
                        } 
                    ] 
                } 
            ] 
        } 
    } 
} 
```

Let's say we create another VM named *myVM2* and associate it to the preceding capacity reservation group.

The Instance View for the capacity reservation group now looks like this example:

```json
{ 
    "name": "myCapacityReservationGroup", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup", 
    "type": "Microsoft.Compute/capacityReservationGroups", 
    "location": "eastus", 
    "properties": { 
        "capacityReservations": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/MYCAPACITYRESERVATIONGROUP/capacityReservations/MYCAPACITYRESERVATION" 
            } 
        ], 
        "virtualMachinesAssociated": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM1" 
            }, 
 { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM2" 
            } 
        ], 
        "instanceView": { 
            "capacityReservations": [ 
                { 
                    "name": "myCapacityReservation", 
"utilizationInfo": { 
                        "virtualMachinesAllocated": [ 
                            { 
                                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM1" 
                            }, 
{ 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/myVM2" 
            } 
                        ] 
                    }, 
                    "statuses": [ 
                        { 
                            "code": "ProvisioningState/succeeded", 
                            "level": "Info", 
                            "displayStatus": "Provisioning succeeded", 
                            "time": "<time>" 
                        } 
                    ] 
                } 
            ] 
        } 
    } 
} 
``` 

The length of `virtualMachinesAllocated` (2) is greater than `capacity` (1). This valid state is referred to as *overallocated*.

> [!IMPORTANT]
> Azure won't stop allocations because a capacity reservation is fully consumed. Autoscale rules, temporary scale-out, and related requirements work beyond the quantity of reserved capacity if Azure has available capacity and other constraints like available quota are met.

## States and considerations

There are three valid states for a specific capacity reservation:

| State  | Status  | Considerations  |
|---|---|---|
| Reserved capacity available  | Length of `virtualMachinesAllocated` < `capacity`  | Is all the reserved capacity needed? Optionally, reduce the capacity to reduce costs.  |
| Reservation consumed  | Length of `virtualMachinesAllocated` == `capacity`  | More VMs won't receive the capacity SLA unless some existing VMs are deallocated. Optionally, try to increase the capacity so that extra planned VMs receive an SLA.  |
| Reservation overallocated  | Length of `virtualMachinesAllocated` > `capacity`  | More VMs won't receive the capacity SLA. Also, the quantity of VMs (length of `virtualMachinesAllocated` – `capacity`) won't receive a capacity SLA if deallocated. Optionally, increase the capacity to add capacity SLA to more of the existing VMs.  |

## Next step

> [!div class="nextstepaction"]
> [Learn how to remove VMs from a capacity reservation](capacity-reservation-remove-vm.md)
