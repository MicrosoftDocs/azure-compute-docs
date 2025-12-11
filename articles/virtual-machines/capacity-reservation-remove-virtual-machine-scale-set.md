---
title: Remove a virtual machine scale set association from a capacity reservation group
description: Learn how to remove a virtual machine scale set from a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to, devx-track-azurepowershell
# Customer intent: "As a cloud administrator, I want to remove a virtual machine scale set from a capacity reservation group, so that I can manage resource allocation and avoid conflicts in capacity usage."
---

# Remove a virtual machine scale set association from a capacity reservation group

**Applies to:** :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

This article walks you through removing a virtual machine scale set association from a capacity reservation group. To learn more about capacity reservations, see the [overview of capacity reservations](capacity-reservation-overview.md).

Because both the virtual machine (VM) and the underlying capacity reservation logically occupy capacity, Azure imposes some constraints on this process to avoid ambiguous allocation states and unexpected errors.

There are three ways to change an association:

- Deallocate the virtual machine scale set, change the capacity reservation group property at the scale set level, and then update the underlying VMs.
- Update the reserved quantity to zero and then change the capacity reservation group property.
- Delete the virtual machine scale set

## Deallocate the virtual machine scale set

The first option is to deallocate the virtual machine scale set, change the capacity reservation group property at the scale set level, and then update the underlying VMs.

For more information about automatic, rolling, and manual upgrades, see [Upgrade policies](#upgrade-policies).

### [API](#tab/api1)

1. Deallocate the virtual machine scale set:

    ```rest
    POST  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/deallocate?api-version=2021-04-01
    ```

1. Update the virtual machine scale set to remove the association with the capacity reservation group:
    
    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/update?api-version=2021-04-01
    ```
    
    In the request body, set the `capacityReservationGroup` property to `null` to remove the virtual machine scale set association to the group:

    ```json
    {
    "location": "eastus",
    "properties": {
        "virtualMachineProfile": {
            "capacityReservation": {
                "capacityReservationGroup":{
                    "id":null    
                }
            }
        }
    }
    }
    ```

### [CLI](#tab/cli1)

1. Deallocate the virtual machine scale set. The following command deallocates all VMs within the scale set:

    ```azurecli-interactive
    az vmss deallocate
    --location eastus
    --resource-group myResourceGroup 
    --name myVMSS 
    ```

1. Update the scale set to remove the association with the capacity reservation group. Setting the `capacity-reservation-group` property to `None` removes the association of the scale set to the capacity reservation group:

    ```azurecli-interactive
    az vmss update 
    --resource-group myresourcegroup 
    --name myVMSS 
    --capacity-reservation-group None
    ```

### [PowerShell](#tab/powershell1)

1. Deallocate the virtual machine scale set. The following command deallocates all VMs within the scale set:

    ```powershell-interactive
    Stop-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myVmss"
    ```

1. Update the scale set to remove the association with the capacity reservation group. Setting the `CapacityReservationGroupId` property to `null` removes the association of the scale set to the capacity reservation group:

    ```powershell-interactive
    $vmss =
    Get-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myVmss"
    
    Update-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myvmss"
    -VirtualMachineScaleSet $vmss
    -CapacityReservationGroupId $null
    ```

To learn more, see the Azure PowerShell commands [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss), [Get-AzVmss](/powershell/module/az.compute/get-azvmss), and [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

### [Portal](#tab/portal1)
	
<!-- no images necessary if steps are straightforward --> 
	
1. Open the [Azure portal](https://portal.azure.com).
1. Go to your Virtual Machine Scale Set and select **Overview**.
1. Select **Stop**.
	- You know your scale set is deallocated when the status changes to *Stopped (deallocated)*.
	- At this point in the process, the scale set is still associated to the capacity reservation group. This association is reflected in the `virtualMachinesAssociated` property of the capacity reservation.
1. Select **Configuration**.
1. Set the **Capacity Reservation group** value to **None**. The scale set is no longer associated to the capacity reservation group.

 
--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Update the reserved quantity to zero

The second option involves updating the reserved quantity to zero and then changing the capacity reservation group property.

This option works well when the scale set can't be deallocated and when a reservation is no longer needed. For example, you might create a capacity reservation to temporarily assure capacity during a large-scale deployment. After deployment is finished, the reservation is no longer needed.

For more information about automatic, rolling, and manual upgrades, see [Upgrade policies](#upgrade-policies).

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

1. Update the virtual machine scale set to remove the association with the capacity reservation group.

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/update?api-version=2021-04-01
    ```

    In the request body, set the `capacityReservationGroup` property to `null` to remove the association:
    
    ```json
    {
    "location": "eastus",
    "properties": {
        "virtualMachineProfile": {
            "capacityReservation": {
                "capacityReservationGroup":{
                    "id":null
                }
            }
        }
    }
    }
    ```

### [CLI](#tab/cli2)

1. Update the reserved quantity to zero:

    ```azurecli-interactive
    az capacity reservation update 
    -g myResourceGroup 
    -c myCapacityReservationGroup 
    -n myCapacityReservation 
    --capacity 0
    ```

1. Update the scale set to remove the association with the capacity reservation group by setting the `capacity-reservation-group` property to `None`:

    ```azurecli-interactive
    az vmss update 
    --resource-group myResourceGroup 
    --name myVMSS 
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

1. Update the scale set to remove the association with the capacity reservation group by setting the `CapacityReservationGroupId` property to `null`:

    ```powershell-interactive
    $vmss =
    Get-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myVmss"
    
    Update-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myvmss"
    -VirtualMachineScaleSet $vmss
    -CapacityReservationGroupId $null
    ```

To learn more, see the Azure PowerShell commands [New-AzCapacityReservation](/powershell/module/az.compute/new-azcapacityreservation), [Get-AzVmss](/powershell/module/az.compute/get-azvmss), and [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

### [Portal](#tab/portal2)
	
<!-- no images necessary if steps are straightforward --> 
	
1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group and select **Overview**.
1. Select **Reservations**.
1. Select **Manage Reservation** at the top of the page.
1. On the **Manage Reservations** pane, enter **0** in the **Instances** field and select **Save**.
1. Go to your Virtual Machine Scale Set and select **Configuration**.
1. Set the **Capacity Reservation group** value to **None**. The scale set is no longer associated to the capacity reservation group.

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->


## Delete the virtual machine scale set

Deletion process of a scale set will remove its association from a capacity reservation. The delete request must complete before Azure removes it from the capacity reservation. Some latency can occur between the delete request and the corresponding change in capacity reservation allocation state. See [Delete a VM](/azure/virtual-machines/delete?tabs=portal2%2Ccli3%2Cportal4%2Cportal5) for more information. Use [Capacity Reservation Instance View](/azure/virtual-machines/capacity-reservation-associate-vm?tabs=api1%2Capi2%2Capi3#view-vm-allocation-with-the-instance-view) to check the allocation status as needed.

## Upgrade policies

- **Automatic upgrade**: In this mode, the scale set VM instances are automatically dissociated from the capacity reservation group without any further action from you.
- **Rolling upgrade**: In this mode, the scale set VM instances are dissociated from the capacity reservation group without any further action from you. However, they're updated in batches with an optional pause time between them.
- **Manual upgrade**: In this mode, nothing happens to the scale set VM instances when the virtual machine scale set is updated. You need to individually remove each scale set VM by [upgrading it with the latest scale set model](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md).

## Next step

> [!div class="nextstepaction"]
> [Learn about overallocating a capacity reservation](capacity-reservation-overallocate.md)
