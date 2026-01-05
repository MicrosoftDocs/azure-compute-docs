---
title: Modify a capacity reservation in Azure
description: Learn how to modify a capacity reservation.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.update-cycle: 180-days
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell, portal
# Customer intent: "As a cloud administrator, I want to modify the capacity reserved for virtual machines, so that I can efficiently manage resource allocation and ensure optimal performance for my applications."
---

# Modify a capacity reservation

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

After you create a capacity reservation group and a capacity reservation, you might want to modify your reservations. This article explains how to do the following actions by using the API, the Azure portal, and PowerShell.

> [!div class="checklist"]
> * Update the number of instances reserved in a capacity reservation.
> * Resize VMs associated to a capacity reservation group.
> * Delete the capacity reservation group and capacity reservation.

## Update the number of instances reserved

Update the number of virtual machine (VM) instances reserved in a capacity reservation.

> [!IMPORTANT]
> In rare cases when Azure can't fulfill the request to increase the quantity reserved for existing capacity reservations, a reservation might go into a *Failed* state and become unavailable until the [quantity is restored to the original amount](#restore-instance-quantity).

### [API](#tab/api1)

```rest
    PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName}?api-version=2021-04-01
``` 
    
In the request body, update the `capacity` property to the new count that you want to reserve:
    
```json
{
    "sku":
    {
        "capacity": 5
    }
} 
```

In this example, the `capacity` property is now set to `5`.

### [Portal](#tab/portal1) 

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group.
1. Select **Overview**.
1. Select **Reservations**.
1. Select **Manage Reservation** at the top.
1. On the **Manage Reservations** page, enter the new quantity to be reserved in the **Instances** field.
1. Select **Save**.

### [CLI](#tab/cli1)

To update the quantity reserved, use `az capacity reservation update` with the updated `capacity` property.

 ```azurecli-interactive
 az capacity reservation update 
 -c myCapacityReservationGroup 
 -n myCapacityReservation 
 -g myResourceGroup2 
 --capacity 5
 ```

### [PowerShell](#tab/powershell1)

To update the quantity reserved, use `New-AzCapacityReservation` with the updated `capacityToReserve` property.

```powershell-interactive
Update-AzCapacityReservation
-ResourceGroupName "myResourceGroup"
-ReservationGroupName "myCapacityReservationGroup"
-Name "myCapacityReservation"
-CapacityToReserve 5
```

To learn more, see the Azure PowerShell command [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservation).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Resize VMs associated to a capacity reservation group

You must do one of the following options if the VM being resized is currently attached to a capacity reservation group and that group doesn't have a reservation for the target size:

- Create a new reservation for that size.
- Remove the VM from the reservation group before you resize.

Check if the target size is part of the reservation group.

### [API](#tab/api2)

1. Get the names of all capacity reservations within the group.
 
    ```rest
        GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}?api-version=2021-04-01
    ``` 
    
    ```json
    { 
        "name": "<CapacityReservationGroupName>", 
        "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}", 
        "type": "Microsoft.Compute/capacityReservationGroups", 
        "location": "eastUS", 
        "zones": [ 
            "1" 
        ], 
        "properties": { 
            "capacityReservations": [ 
                { 
                    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName1}" 
                }, 
    { 
                    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName2}" 
                } 
            ] 
        } 
    } 
    ```

1. Find out the VM size reserved for each reservation. The following example is for `capacityReservationName1`, but you can repeat this step for other reservations.

    ```rest
        GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName1}?api-version=2021-04-01
    ``` 
    
    ```json
    { 
        "name": "capacityReservationName1", 
        "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName1}", 
        "type": "Microsoft.Compute/capacityReservationGroups/capacityReservations", 
        "location": "eastUS", 
        "sku": { 
            "name": "Standard_D2s_v3", 
            "capacity": 3 
        }, 
        "zones": [ 
            "1" 
        ], 
        "properties": { 
            "reservationId": "<reservationId>", 
            "provisioningTime": "<provisioningTime>", 
            "provisioningState": "Succeeded" 
        } 
    }  
    ```

1. Consider the following scenarios:

   - If the target VM size isn't part of the group, [create a new capacity reservation](capacity-reservation-create.md) for the target VM.
   - If the target VM size already exists in the group, [resize the VM](resize-vm.md).

### [Portal](#tab/portal2)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group.
1. Select **Overview**.
1. Select **Reservations**.
1. Look at the **VM size** reserved for each reservation.
    1. If the target VM size isn't part of the group, [create a new capacity reservation](capacity-reservation-create.md) for the target VM.
    1. If the target VM size already exists in the group, [resize the VM](resize-vm.md).

### [CLI](#tab/cli2)

1. Get the names of all capacity reservations within the capacity reservation group with `az capacity reservation group show`.

    ```azurecli-interactive
    az capacity reservation group show 
    -g myResourceGroup
    -n myCapacityReservationGroup 
    ```

1. From the response, find the names of all the capacity reservations.

1. To find out the VM sizes reserved for each reservation, run the following commands:

    ```azurecli-interactive
    az capacity reservation show
    -g myResourceGroup
    -c myCapacityReservationGroup 
    -n myCapacityReservation 
    ```

1. Consider the following scenarios:

   - If the target VM size isn't part of the group, [create a new capacity reservation](capacity-reservation-create.md) for the target VM.
   - If the target VM size already exists in the group, [resize the VM](resize-vm.md).

### [PowerShell](#tab/powershell2)

1. Get the names of all capacity reservations within the group with `Get-AzCapacityReservationGroup`.

    ```powershell-interactive
    Get-AzCapacityReservationGroup
    -ResourceGroupName "myResourceGroup"
    -Name "myCapacityReservationGroup"
    ```

1. From the response, find the names of all the capacity reservations.

1. To find out the VM sizes reserved for each reservation, run the following commands:

    ```powershell-interactive
    $CapRes =
    Get-AzCapacityReservation
    -ResourceGroupName "myResourceGroup"
    -ReservationGroupName "myCapacityReservationGroup"
    -Name "mycapacityReservation"
    
    $CapRes.Sku
    ```

1. Consider the following scenarios:

   - If the target VM size isn't part of the group, [create a new capacity reservation](capacity-reservation-create.md) for the target VM.
   - If the target VM size already exists in the group, [resize the VM](resize-vm.md).

To learn more, see the Azure PowerShell commands [Get-AzCapacityReservationGroup](/powershell/module/az.compute/get-azcapacityreservationgroup) and [Get-AzCapacityReservation](/powershell/module/az.compute/get-azcapacityreservation).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Delete a capacity reservation group and capacity reservation

Azure allows a group to be deleted when all the member capacity reservations are deleted and no VMs are associated to the group.  

To delete a capacity reservation, first find out all of the VMs that are associated to it. The list of VMs is available under the `virtualMachinesAssociated` property.

### [API](#tab/api3)

First, find all the VMs associated to the capacity reservation group and dissociate them:
 
```rest
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}?$expand=instanceView&api-version=2021-04-01
``` 

```json
{ 
    "name": "<capacityReservationGroupName>", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}", 
    "type": "Microsoft.Compute/capacityReservationGroups", 
    "location": "eastus", 
    "properties": { 
        "capacityReservations": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}/capacityReservations/{capacityReservationName}" 
            } 
        ], 
        "virtualMachinesAssociated": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName1}" 
            }, 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName2}" 
            } 
        ], 
        "instanceView": { 
            "capacityReservations": [ 
                { 
                    "name": "{capacityReservationName}", 
                    "utilizationInfo": { 
                        "virtualMachinesAllocated": [ 
                            { 
                                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName1}" 
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

From the preceding response, find the names of all the VMs under the `virtualMachinesAssociated` property. Remove them from the capacity reservation group by using the steps in [Remove a VM association to a capacity reservation](capacity-reservation-remove-vm.md).

After all the VMs are removed from the capacity reservation group, delete the member capacity reservations:

```rest
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName}?api-version=2021-04-01
```

Lastly, delete the parent capacity reservation group:

```rest
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}?api-version=2021-04-01
```

### [Portal](#tab/portal3)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group.
1. Select **Resources**.
1. Find out all the VMs that are associated to the group.
1. [Disassociate every VM](capacity-reservation-remove-vm.md).
1. Delete every capacity reservation in the group:
    1. Go to **Reservations**.
    1. Select each reservation.
    1. Select **Delete**.
1. Delete the capacity reservation group:
    1. Go to the capacity reservation group.
    1. Select **Delete** at the top of the page.

### [CLI](#tab/cli3)

Find out all the VMs associated to the capacity reservation group and dissociate them.

1. Run the following command:
    
    ```azurecli-interactive
    az capacity reservation group show
    -g myResourceGroup
    -n myCapacityReservationGroup
    ```

1. From the preceding response, find out the names of all the VMs under the `VirtualMachinesAssociated` property and remove them from the capacity reservation group by using the steps in [Remove a VM association from a capacity reservation group](capacity-reservation-remove-vm.md).

1. Delete the capacity reservation:

    ```azurecli-interactive
    az capacity reservation delete 
    -g myResourceGroup 
    -c myCapacityReservationGroup 
    -n myCapacityReservation 
    ```

1. Delete the capacity reservation group:

    ```azurecli-interactive
    az capacity reservation group delete 
    -g myResourceGroup 
    -n myCapacityReservationGroup
    ```

### [PowerShell](#tab/powershell3)

Find out all the VMs associated to the capacity reservation group and dissociate them.

1. Run the following command:
    
    ```powershell-interactive
    Get-AzCapacityReservationGroup
    -ResourceGroupName "myResourceGroup"
    -Name "myCapacityReservationGroup"
    ```

1. From the preceding response, find out the names of all the VMs under the `VirtualMachinesAssociated` property. Remove them from the capacity reservation group by using the steps in [Remove a VM association from a capacity reservation group](capacity-reservation-remove-vm.md).

1. Delete the capacity reservation:

    ```powershell-interactive
    Remove-AzCapacityReservation
    -ResourceGroupName "myResourceGroup"
    -ReservationGroupName "myCapacityReservationGroup"
    -Name "myCapacityReservation"
    ```

1. Delete the capacity reservation group:

    ```powershell-interactive
    Remove-AzCapacityReservationGroup
    -ResourceGroupName "myResourceGroup"
    -Name "myCapacityReservationGroup"
    ```

To learn more, see the Azure PowerShell commands [Get-AzCapacityReservationGroup](/powershell/module/az.compute/get-azcapacityreservationgroup), [Remove-AzCapacityReservation](/powershell/module/az.compute/remove-azcapacityreservation), and [Remove-AzCapacityReservationGroup](/powershell/module/az.compute/remove-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Restore instance quantity

A well-formed request for reducing the reserved quantity should always succeed no matter the number of VMs associated to the reservation. However, increasing the reserved quantity might require more quota and for Azure to fulfill the request for more capacity. In a rare scenario in which Azure can't fulfill the request to increase the reserved quantity for existing reservations, the reservation might go into a *Failed* state and become unavailable until the reserved quantity is restored to the original amount.

> [!NOTE]
> If a reservation is in a *Failed* state, all the VMs that are associated to the reservation continue to work as normal.

For example, let's say `myCapacityReservation` has 5 instances reserved. You request 5 extra instances, which makes the total reserved quantity equal to 10. However, because of a constrained capacity situation in the region, Azure can't fulfill the extra 5 instances that you requested. In this case, `myCapacityReservation` fails to meet its intended state of 10 reserved instances and goes into a *Failed* state.

To resolve this failure, follow these steps to locate the old quantity reserved value:

1. In the Azure portal, go to [Application Change Analysis](https://portal.azure.com/#blade/Microsoft_Azure_ChangeAnalysis/ChangeAnalysisBaseBlade).
1. Select the applicable **Subscription**, **Resource group**, and **Time range** settings in the filters.
   You can only go back up to 14 days in the past in the **Time range** filter.
1. Search for the name of the capacity reservation.
1. Look for the change in the `sku.capacity` property for that reservation.
   The old quantity reserved is the value under the **Old Value** column.

Update `myCapacityReservation` to the old quantity reserved. After it's updated, the reservation is available immediately for use with your VMs.

## Next step

> [!div class="nextstepaction"]
> [Learn how to remove VMs from a capacity reservation](capacity-reservation-remove-vm.md)
