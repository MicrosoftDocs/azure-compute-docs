---
title: Associate a virtual machine scale set with Uniform Orchestration to a capacity reservation group.
description: Learn how to associate a new or existing virtual machine scale with Uniform Orchestration set to a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell
---

# Associate a virtual machine scale set to Uniform Orchestration to a capacity reservation group

**Applies to:** :heavy_check_mark: Uniform scale set

Azure Virtual Machine Scale Sets has two modes:

- **Uniform Orchestration**: In this mode, virtual machine scale sets use a virtual machine (VM) profile or a template to scale up to the capacity you want. Although there's some ability to manage or customize individual VM instances, Uniform Orchestration uses identical VM instances. These instances are exposed through the virtual machine scale set's VM APIs and aren't compatible with the API commands that are standard for Azure infrastructure as a service (IaaS) VMs. Because the scale set performs all the actual VM operations, reservations are associated to the virtual machine scale set directly. After the scale set is associated to the reservation, all the subsequent VM allocations are done against the reservation.
- **Flexible Orchestration**: In this mode, you get more flexibility to manage the individual virtual machine scale set VM instances. They can use the standard Azure IaaS VM APIs instead of by using the scale set interface. To use reservations with Flexible Orchestration mode, define both the virtual machine scale set property and the capacity reservation property on each VM.

To learn more about these modes, see [Virtual Machine Scale Sets orchestration modes](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).

This content applies to the Uniform Orchestration mode. For Flexible Orchestration mode, see [Associate a virtual machine scale set with Flexible Orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set-flex.md).

## Limitations of scale sets in Uniform Orchestration

- For virtual machine scale sets in Uniform Orchestration to be compatible with capacity reservation, the `singlePlacementGroup` property must be set to `False`.
- The **Static Fixed Spreading** availability option for multizone uniform scale sets isn't supported with capacity reservation. This option requires the use of five fault domains. However, the reservations only support up to three fault domains for general purpose sizes. The approach we recommend is to use the **Max Spreading** option that spreads VMs across as many fault domains as possible within each zone. If needed, configure a custom fault domain configuration of three or less.

There are some other restrictions when you use capacity reservations. For the complete list, see the [overview of capacity reservations](capacity-reservation-overview.md).

## Associate a new virtual machine scale set to a capacity reservation group

> [!IMPORTANT]
> Starting November 2023, virtual machine scale sets created by using PowerShell and the Azure CLI default to Flexible Orchestration mode if no orchestration mode is specified. For more information about this change and what actions you should take, see [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](
https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295).

### [API](#tab/api1)  

To associate a new uniform virtual machine scale set to a capacity reservation group, construct the following `PUT` request to the `Microsoft.Compute` provider:

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}?api-version=2021-04-01
```

Add the `capacityReservationGroup` property in the `virtualMachineProfile` property:

```json
{ 
    "name": "<VMScaleSetName>", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}", 
    "type": "Microsoft.Compute/virtualMachineScaleSets", 
    "location": "eastus", 
    "sku": { 
        "name": "Standard_D2s_v3", 
        "tier": "Standard", 
        "capacity": 3 
}, 
"properties": { 
    "virtualMachineProfile": { 
        "capacityReservation": { 
            "capacityReservationGroup":{ 
                "id":"subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroup/{CapacityReservationGroupName}" 
            } 
         }, 
        "osProfile": { 
            … 
        }, 
        "storageProfile": { 
            … 
        }, 
        "networkProfile": { 
            …,
            "extensionProfile": { 
                … 
            } 
        } 
    } 
```

### [CLI](#tab/cli1)

Use `az vmss create` to create a new virtual machine scale set and add the `capacity-reservation-group` property to associate the scale set to an existing capacity reservation group. The following example creates a uniform scale set for a Standard_Ds1_v2 VM in the East US location and associates the scale set to a capacity reservation group.

```azurecli-interactive
az vmss create 
--resource-group myResourceGroup 
--name myVMSS 
--location eastus 
--orchestration-mode Uniform
--vm-sku Standard_Ds1_v2 
--image Ubuntu2204 
--capacity-reservation-group /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName} 
```

### [PowerShell](#tab/powershell1) 

Use `New-AzVmss` to create a new virtual machine scale set and add the `CapacityReservationGroupId` property to associate the scale set to an existing capacity reservation group. The following example creates a uniform scale set for a Standard_Ds1_v2 VM in the East US location and associates the scale set to a capacity reservation group.

```powershell-interactive
$vmssName = <"VMSSNAME">
$vmPassword = ConvertTo-SecureString <"PASSWORD"> -AsPlainText -Force
$vmCred = New-Object System.Management.Automation.PSCredential(<"USERNAME">, $vmPassword)
New-AzVmss
–Credential $vmCred
-VMScaleSetName $vmssName
-ResourceGroupName "myResourceGroup"
-CapacityReservationGroupId "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}"
-OrchestrationMode "Uniform"
-PlatformFaultDomainCount 2
```

To learn more, see the Azure PowerShell command [New-AzVmss](/powershell/module/az.compute/new-azvmss).

### [ARM template](#tab/arm1)

An [Azure Resource Manager template (ARM template)](/azure/azure-resource-manager/templates/overview) is a JSON file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment.

ARM templates let you deploy groups of related resources. In a single template, you can create capacity reservation group and capacity reservations. You can deploy templates through the Azure portal, the Azure CLI, or Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines.

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Associate an existing virtual machine scale set to a capacity reservation group 

To add an existing capacity reservation group to an existing uniform scale set:

- Stop the scale set to deallocate the VM instances.
- Update the scale set to use a matching capacity reservation group.
- Start the scale set.

This process ensures that the placement for the capacity reservations and scale set in the region are compatible.

### Important notes on upgrade policies

- **Automatic upgrade**: In this mode, the scale set VM instances are automatically associated to the capacity reservation group without any further action from you. When the scale set VMs are reallocated, they start consuming the reserved capacity.
- **Rolling upgrade**: In this mode, scale set VM instances are associated to the capacity reservation group without any further action from you. However, they're updated in batches with an optional pause time between them. When the scale set VMs are reallocated, they start consuming the reserved capacity.
- **Manual upgrade**: In this mode, nothing happens to the scale set VM instances when the virtual machine scale set is attached to a capacity reservation group. You need to update to each scale set VM by [upgrading it with the latest scale set model](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md).

### [API](#tab/api2)

1. Deallocate the virtual machine scale set:

    ```rest
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourcegroupname}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/deallocate?api-version=2021-04-01
    ```

1. Add the `capacityReservationGroup` property to the scale set model. Construct the following `PUT` request to `Microsoft.Compute` provider:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourcegroupname}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}?api-version=2021-04-01
    ```

    In the request body, include the `capacityReservationGroup` property:

    ```json
    "location": "eastus",
    "properties": {
        "virtualMachineProfile": {
             "capacityReservation": {
                      "capacityReservationGroup": {
                            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}"
                      }
                }
        }
    }
    ```

### [CLI](#tab/cli2)

1. Deallocate the virtual machine scale set:

    ```azurecli-interactive
    az vmss deallocate 
    --location eastus
    --resource-group myResourceGroup 
    --name myVMSS 
    ```

1. Associate the scale set to the capacity reservation group:

    ```azurecli-interactive
    az vmss update 
    --resource-group myResourceGroup 
    --name myVMSS 
    --capacity-reservation-group /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}
    ```

### [PowerShell](#tab/powershell2) 

1. Deallocate the virtual machine scale set:

    ```powershell-interactive
    Stop-AzVmss
    -ResourceGroupName "myResourceGroup”
    -VMScaleSetName "myVmss”
    ```

1. Associate the scale set to the capacity reservation group:

    ```powershell-interactive
    $vmss =
    Get-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myVmss"
    
    Update-AzVmss
    -ResourceGroupName "myResourceGroup"
    -VMScaleSetName "myVmss"
    -VirtualMachineScaleSet $vmss
    -CapacityReservationGroupId "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}"
    ```

To learn more, see the Azure PowerShell commands [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss), [Get-AzVmss](/powershell/module/az.compute/get-azvmss), and [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## View virtual machine scale set association with the Instance View

After the uniform virtual machine scale set is associated to the capacity reservation group, all the subsequent VM allocations will happen against the capacity reservation. Azure automatically finds the matching capacity reservation in the group and consumes a reserved slot.

### [API](#tab/api3) 

The capacity reservation group Instance View reflects the new scale set VMs under the `virtualMachinesAssociated` and `virtualMachinesAllocated` properties:  

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}?$expand=instanceview&api-version=2021-04-01 
```

```json
{ 
    "name": "<CapacityReservationGroupName>", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}", 
    "type": "Microsoft.Compute/capacityReservationGroups", 
    "location": "eastus" 
}, 
    "properties": { 
        "capacityReservations": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{CapacityReservationName}" 
            } 
        ], 
        "virtualMachinesAssociated": [ 
            { 
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/virtualMachines/{VirtualMachineId}" 
            } 
        ], 
        "instanceView": { 
            "capacityReservations": [ 
                { 
                    "name": "<CapacityReservationName>", 
                    "utilizationInfo": { 
                        "virtualMachinesAllocated": [ 
                            { 
                                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMScaleSetName}/virtualMachines/{VirtualMachineId}" 
                            } 
                        ] 
                    },
                    "statuses": [ 
                        { 
                            "code": "ProvisioningState/succeeded", 
                            "level": "Info", 
                            "displayStatus": "Provisioning succeeded", 
                            "time": "2021-05-25T15:12:10.4165243+00:00" 
                        } 
                    ] 
                } 
            ] 
        } 
    } 
} 
```  

### [CLI](#tab/cli3)

```azurecli-interactive
az capacity reservation group show 
-g myResourceGroup
-n myCapacityReservationGroup 
``` 

### [PowerShell](#tab/powershell3) 

View the association of your virtual machine scale set and capacity reservation group by using Instance View in PowerShell.

```powershell-interactive
$CapRes=
Get-AzCapacityReservationGroup
-ResourceGroupName <"ResourceGroupName">
-Name <"CapacityReservationGroupName">
-InstanceView

$CapRes.InstanceView.Utilizationinfo.VirtualMachinesAllocated
``` 

To learn more, see the Azure PowerShell command [Get-AzCapacityReservationGroup](/powershell/module/az.compute/get-azcapacityreservationGroup).

### [Portal](#tab/portal3)

1. Open [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group.
1. Under **Setting**, select **Resources**.
1. In the table, you can see all the scale set VMs that are associated to the capacity reservation group.

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Region and availability zone considerations

You can create virtual machine scale sets regionally or in one or more availability zones to help protect them from datacenter-level failure. To learn more about multizonal virtual machine scale sets, see [Virtual machine scale sets that use availability zones](../virtual-machine-scale-sets/virtual-machine-scale-sets-use-availability-zones.md).  


>[!IMPORTANT]
> The location (region and availability zones) of the virtual machine scale set and the capacity reservation group must match for the association to succeed. For a regional scale set, the region must match between the scale set and the capacity reservation group. For a zonal scale set, both the regions and the zones must match between the scale set and the capacity reservation group.

When a scale set is spread across multiple zones, it always attempts to deploy evenly across the included availability zones. Because of that even deployment, a capacity reservation group should always have the same quantity of reserved VMs in each zone. As an illustration of why this even deployment is important, consider the following example.

In this example, each zone has a different quantity reserved. Let's say that the virtual machine scale set scales out to 75 instances. Because a scale set always attempts to deploy evenly across zones, the VM distribution should look like this example:

| Zone  | Quantity reserved  | Number of scale set VMs in each zone  | Unused quantity reserved  | Overallocated   |
|---|---|---|---|---|
| 1  | 40  | 25  | 15  | 0  |
| 2  | 20  | 25  | 0  | 5  |
| 3  | 15  | 25  | 0  | 10  |

In this case, the scale set incurs extra cost for 15 unused instances in Zone 1. The scale-out is also relying on 5 VMs in Zone 2 and 10 VMs in Zone 3 that aren't protected by capacity reservation. If each zone had 25 capacity instances reserved, then all 75 VMs would be protected by capacity reservation and the deployment wouldn't incur any extra cost for unused instances.

Because the reservations can be overallocated, the scale set can continue to scale normally beyond the limits of the reservation. The only difference is that the VMs allocated above the quantity reserved aren't covered by capacity reservation service-level agreement. To learn more, see [Overallocate capacity reservation](capacity-reservation-overallocate.md).

## Next step

> [!div class="nextstepaction"]
> [Learn how to remove a scale set association from a capacity reservation](capacity-reservation-remove-virtual-machine-scale-set.md)
