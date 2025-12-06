---
title: Remove a virtual machine association from a capacity reservation group
description: Learn how to remove a virtual machine from a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/24/2023
ms.reviewer: cynthn, jushiman, mattmcinnes
ms.custom: template-how-to, devx-track-azurepowershell
# Customer intent: As a cloud administrator, I want to remove a virtual machine's association from a capacity reservation group, so that I can manage my resources and optimize capacity allocation without encountering errors.
---

# Remove a VM association from a capacity reservation group

This article walks you through the steps of removing a virtual machine (VM) association to a capacity reservation group. To learn more about capacity reservations, see the [Capacity reservation overview](capacity-reservation-overview.md).

Because both the VM and the underlying capacity reservation logically occupy capacity, Azure imposes some constraints on this process to avoid ambiguous allocation states and unexpected errors.

There are three ways to change an association:

- Deallocate the virtual machine, change the capacity reservation group property, and, optionally, restart the VM.
- Update the reserved quantity to zero and then change the capacity reservation group property.
- Delete a VM.

## Deallocate the virtual machine

The first option is to deallocate the virtual machine, change the capacity reservation group property, and, optionally, restart the VM.

### [API](#tab/api1)

1. Deallocate the virtual machine:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/deallocate?api-version=2021-04-01
    ```

1. Update the VM to remove the association with the capacity reservation group:
    
    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/update?api-version=2021-04-01
    ```
    In the request body, set the `capacityReservationGroup` property to `null` to remove the VM association to the group:

    ```json
     {
    "location": "eastus",
    "properties": {
        "capacityReservation": {
            "capacityReservationGroup": {
                "id":null
            }
        }
    }
    }
    ```

### [Portal](#tab/portal1)

<!-- no images necessary if steps are straightforward --> 

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your VM and select **Overview**.
1. Select **Stop**.
    - You know your VM is deallocated when the status changes to *Stopped (deallocated)*.
    - At this point in the process, the VM is still associated to the capacity reservation group. This association is reflected in the `virtualMachinesAssociated` property of the capacity reservation.
1. Select **Configuration**.
1. Set the **Capacity Reservation group** value to **None**. The VM is no longer associated to the capacity reservation group.

### [CLI](#tab/cli1)

1. Deallocate the virtual machine:

    ```azurecli-interactive
    az vm deallocate 
    -g myResourceGroup 
    -n myVM
    ```

    After the status changes to *Stopped (deallocated)*, the VM is deallocated.

1. Update the VM to remove association with the capacity reservation group by setting the `capacity-reservation-group` property to `None`:

    ```azurecli-interactive
    az vm update 
    -g myresourcegroup 
    -n myVM 
    --capacity-reservation-group None
    ```

### [PowerShell](#tab/powershell1)

1. Deallocate the virtual machine:

    ```powershell-interactive
    Stop-AzVM
    -ResourceGroupName "myResourceGroup"
    -Name "myVM"
    ```

    After the status changes to *Stopped (deallocated)*, the VM is deallocated.

1. Update the VM to remove association with the capacity reservation group by setting the `CapacityReservationGroupId` property to `null`:

    ```powershell-interactive
    $VirtualMachine =
    Get-AzVM
    -ResourceGroupName "myResourceGroup"
    -Name "myVM"
    
    Update-AzVM
    -ResourceGroupName "myResourceGroup"
    -VM $VirtualMachine
    -CapacityReservationGroupId $null
    ```

To learn more, see the the Azure PowerShell commands [Stop-AzVM](/powershell/module/az.compute/stop-azvm), [Get-AzVM](/powershell/module/az.compute/get-azvm), and [Update-AzVM](/powershell/module/az.compute/update-azvm).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Update the reserved quantity to zero 

The second option involves updating the reserved quantity to zero and then changing the capacity reservation group property.

This option works well when the VM can't be deallocated and when a reservation is no longer needed. For example, you might create a capacity reservation to temporarily assure capacity during a large-scale deployment. After it's finished, the reservation is no longer needed.

### [API](#tab/api2)

1. Update the reserved quantity to zero:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/CapacityReservations/{CapacityReservationName}?api-version=2021-04-01
    ```

    In the request body, include the following parameters:
    
    ```json
    {
    "sku":
        {
        "capacity": 0
        }
    }
    ```
    
    Note that the `capacity` property is set to `0`.

1. Update the VM to remove the association with the capacity reservation group.

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName}/update?api-version=2021-04-01
    ```

    In the request body, set the `capacityReservationGroup` property to `null` to remove the association:
    
    ```json
    {
    "location": "eastus",
    "properties": {
        "capacityReservation": {
            "capacityReservationGroup": {
                "id":null
            }
        }
    }
    } 
    ```

### [Portal](#tab/portal2)

<!-- no images necessary if steps are straightforward --> 

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group and select **Overview**.
1. Select **Reservations**.
1. Select **Manage Reservation** at the top of the page.
1. On the **Manage Reservations** pane, enter **0** in the **Instances** field and select **Save**.
1. Go to your VM and select **Configuration**.
1. Set the **Capacity Reservation group** value to **None**. The VM is no longer associated to the capacity reservation group.

### [CLI](#tab/cli2)

1. Update the reserved quantity to zero:

   ```azurecli-interactive
   az capacity reservation update 
   -g myResourceGroup
   -c myCapacityReservationGroup 
   -n myCapacityReservation 
   --capacity 0
   ```

1. Update the VM to remove association with the capacity reservation group by setting the `capacity-reservation-group` property to `None`:

    ```azurecli-interactive
    az vm update 
    -g myresourcegroup 
    -n myVM 
    --capacity-reservation-group None
    ```

### [PowerShell](#tab/powershell2)

1. Update the reserved quantity to zero:

    ```powershell-interactive
    Update-AzCapacityReservation
    -ResourceGroupName "myResourceGroup"
    -ReservationGroupName "myCapacityReservationGroup"
    -Name "myCapacityReservation"
    -CapacityToReserve 0
    ```

1. Update the VM to remove association with the capacity reservation group by setting the `CapacityReservationGroupId` property to `null`:

    ```powershell-interactive
    $VirtualMachine =
    Get-AzVM
    -ResourceGroupName "myResourceGroup"
    -Name "myVM"
    
    Update-AzVM
    -ResourceGroupName "myResourceGroup"
    -VM $VirtualMachine
    -CapacityReservationGroupId $null
    ```

To learn more, see the Azure PowerShell commands [New-AzCapacityReservation](/powershell/module/az.compute/new-azcapacityreservation), [Get-AzVM](/powershell/module/az.compute/get-azvm), and [Update-AzVM](/powershell/module/az.compute/update-azvm).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Delete a VM

The virtual machine deletion process will remove a VM association from a capacity reservation. A VM delete must complete before Azure removes it from the capacity reservation. Some latency can occur between the delete request and the corresponding change in capacity reservation allocation state. See [Delete a VM](/azure/virtual-machines/delete?tabs=portal2%2Ccli3%2Cportal4%2Cportal5) for more information. Use [Capacity Reservation Instance View](/azure/virtual-machines/capacity-reservation-associate-vm?tabs=api1%2Capi2%2Capi3#view-vm-allocation-with-the-instance-view) to check the allocation status as needed.

## Next step

> [!div class="nextstepaction"]
> [Learn how to associate a scale set to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md)
