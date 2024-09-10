---
title: Associate a virtual machine to a capacity reservation group
description: Learn how to associate a new or existing virtual machine to a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell
---

# Associate a VM to a capacity reservation group

**Applies to:** :heavy_check_mark: Windows Virtual Machines :heavy_check_mark: Linux Virtual Machines 

You can use capacity reservation groups with new or existing virtual machines (VMs). To learn more about capacity reservations, see the [Capacity reservation overview](capacity-reservation-overview.md).

## Associate a new virtual machine

To associate a new virtual machine to the capacity reservation group, the group must be explicitly referenced as a property of the VM. This reference protects the matching reservation in the group for applications and workloads intended to use it.

### [API](#tab/api1)

To add the `capacityReservationGroup` property to a VM, construct the following `PUT` request to the `Microsoft.Compute` provider:

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName}?api-version=2021-04-01
```

In the request body, include the `capacityReservationGroup` property:

```json
{ 
  "location": "eastus", 
  "properties": { 
    "hardwareProfile": { 
      "vmSize": "Standard_D2s_v3" 
    }, 
    … 
   "capacityReservation":{ 
    "capacityReservationGroup":{ 
        "id":"subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}" 
    } 
    "storageProfile": { 
    … 
    }, 
    "osProfile": { 
    … 
    }, 
    "networkProfile": { 
     …     
    } 
  } 
} 
```

### [Portal](#tab/portal1)

<!-- no images necessary if steps are straightforward --> 

1. Open the [Azure portal](https://portal.azure.com).
1. In the search bar, enter **virtual machine**.
1. Under **Services**, select **Virtual machines**.
1. On the **Virtual machines** page, select **Create** and then select **Virtual machine**.
1. On the **Basics** tab, under **Project details**, select the correct subscription. Then choose to create a new resource group or use an existing one.
1. Under **Instance details**, enter the VM name and choose your region.
1. Select an **Image** and the VM size.
1. Under **Administrator account**, enter a username and a password. The password must be at least 12 characters long and meet the defined complexity requirements.
1. Go to the **Advanced section**.
1. In the **Capacity Reservations** dropdown list, select the capacity reservation group that you want to associate to the VM.
1. Select **Review + create**.
1. After validation runs, select **Create**.
1. After the deployment is finished, select **Go to resource**.

### [CLI](#tab/cli1)

Use `az vm create` to create a new VM and add the `capacity-reservation-group` property to associate it to an existing capacity reservation group. The following example creates a Standard_D2s_v3 VM in the East US location and associates the VM to a capacity reservation group.

```azurecli-interactive
az vm create 
--resource-group myResourceGroup 
--name myVM 
--location eastus 
--size Standard_D2s_v3 
--image Ubuntu2204 
--capacity-reservation-group /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}
```

### [PowerShell](#tab/powershell1)

Use `New-AzVM` to create a new VM and add the `CapacityReservationGroupId` property to associate it to an existing capacity reservation group. The following example creates a Standard_D2s_v3 VM in the East US location and associates the VM to a capacity reservation group.

```powershell-interactive
New-AzVm
-ResourceGroupName "myResourceGroup"
-Name "myVM"
-Location "eastus"
-VirtualNetworkName "myVnet"
-SubnetName "mySubnet"
-SecurityGroupName "myNetworkSecurityGroup"
-PublicIpAddressName "myPublicIpAddress"
-Size "Standard_D2s_v3"
-CapacityReservationGroupId "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}"
```

To learn more, see the Azure PowerShell command [New-AzVM](/powershell/module/az.compute/new-azvm).

### [ARM template](#tab/arm1)

An [Azure Resource Manager template (ARM template)](/azure/azure-resource-manager/templates/overview) is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment.

ARM templates let you deploy groups of related resources. In a single template, you can create a capacity reservation group and capacity reservations. You can deploy templates through the Azure portal, the Azure CLI, or Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines.

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Associate an existing virtual machine

For the initial release of a capacity reservation, a virtual machine must be allocated to a capacity reservation:

1. Follow guidance to create a capacity reservation group and capacity reservation, if necessary. Or increment the quantity of an existing capacity reservation so there's unused reserved capacity.
1. Deallocate the virtual machine.
1. Update the capacity reservation group property on the VM.
1. Restart the VM.

Follow the steps to associate the virtual machine to a capacity reservation group.

### [API](#tab/api2)

1. Deallocate the virtual machine:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourcegroupname}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName}/deallocate?api-version=2021-04-01
    ```

1. Add the `capacityReservationGroup` property to the VM. Construct the following `PUT` request to `Microsoft.Compute` provider:

    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VirtualMachineName}?api-version=2021-04-01
    ```

    In the request body, include the `capacityReservationGroup` property:
    
    ```json
    {
    "location": "eastus",
    "properties": {
        "capacityReservation": {
            "capacityReservationGroup": {
                "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}"
            }
        }
    }
    }
    ```


### [Portal](#tab/portal2)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your VM.
1. Select **Overview**.
1. Select **Stop** at the top of the page to deallocate the VM.
1. Go to **Configurations** on the left.
1. In the **Capacity Reservation group** dropdown list, select the group that you want to associate to the VM.

### [CLI](#tab/cli2)

1. Deallocate the virtual machine:

    ```azurecli-interactive
    az vm deallocate 
    -g myResourceGroup 
    -n myVM
    ```

1. Associate the VM to a capacity reservation group:

    ```azurecli-interactive
    az vm update 
    -g myresourcegroup 
    -n myVM 
    --capacity-reservation-group subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}
    ```

### [PowerShell](#tab/powershell2)

1. Deallocate the virtual machine:

    ```powershell-interactive
    Stop-AzVM
    -ResourceGroupName "myResourceGroup"
    -Name "myVM"
    ```

1. Associate the VM to a capacity reservation group:

    ```powershell-interactive
    $VirtualMachine =
    Get-AzVM
    -ResourceGroupName "myResourceGroup"
    -Name "myVM"
    
    Update-AzVM
    -ResourceGroupName "myResourceGroup"
    -VM $VirtualMachine
    -CapacityReservationGroupId "subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}"
    ```

To learn more, see the Azure PowerShell commands [Stop-AzVM](/powershell/module/az.compute/stop-azvm), [Get-AzVM](/powershell/module/az.compute/get-azvm), and [Update-AzVM](/powershell/module/az.compute/update-azvm).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## View VM association with the Instance View

After the `capacityReservationGroup` property is set, an association now exists between the VM and the group. Azure automatically finds the matching capacity reservation in the group and consumes a reserved slot. The capacity reservation's Instance View reflects the new VM in the `virtualMachinesAllocated` property:

### [API](#tab/api3)

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{capacityReservationGroupName}?$expand=instanceView&api-version=2021-04-01 
```

```json
{
   "name":"{CapacityReservationGroupName}",
   "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}",
   "type":"Microsoft.Compute/capacityReservationGroups",
   "location":"eastus",
   "properties":{
      "capacityReservations":[
         {
            "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/ {CapacityReservationGroupName}/capacityReservations/{CapacityReservationName}"
         }
      ],
      "virtualMachinesAssociated":[
         {
            "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{myVM}"
         }
      ],
      "instanceView":{
         "capacityReservations":[
            {
               "name":"{CapacityReservationName}",
               "utilizationInfo":{
                  "virtualMachinesAllocated":[
                     {
                        "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{myVM}"
                     }
                  ]
               },
               "statuses":[
                  {
                     "code":"ProvisioningState/succeeded",
                     "level":"Info",
                     "displayStatus":"Provisioning succeeded",
                     "time":"2021-05-25T15:12:10.4165243+00:00"
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
az capacity reservation show 
-g myResourceGroup
-c myCapacityReservationGroup 
-n myCapacityReservation 
```

### [PowerShell](#tab/powershell3)

```powershell-interactive
$CapRes=
Get-AzCapacityReservation
-ResourceGroupName <"ResourceGroupName">
-ReservationGroupName] <"CapacityReservationGroupName">
-Name <"CapacityReservationName">
-InstanceView

$CapRes.InstanceView.Utilizationinfo.VirtualMachinesAllocated
```

To learn more, see the Azure PowerShell command [Get-AzCapacityReservation](/powershell/module/az.compute/get-azcapacityreservation).

### [Portal](#tab/portal3)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your capacity reservation group.
1. Under **Settings** on the left, select **Resources**.
1. Look at the table to see all the VMs that are associated to the capacity reservation group.

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Next step

> [!div class="nextstepaction"]
> [Remove a VM's association to a capacity reservation group](capacity-reservation-remove-vm.md)
