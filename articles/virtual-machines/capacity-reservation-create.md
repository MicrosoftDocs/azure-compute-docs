---
title: Create a capacity reservation in Azure
description: Learn how to reserve compute capacity in an Azure region or an availability zone by creating a capacity reservation.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.update-cycle: 180-days
ms.date: 04/24/2023
ms.reviewer: cynthn, jushiman, mattmcinnes
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell, portal
# Customer intent: As an IT administrator, I want to create a capacity reservation and group in a cloud environment, so that I can ensure dedicated compute resources for my virtual machine deployment.
---

# Create a capacity reservation

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

Capacity reservation is always created as part of a capacity reservation group. The first step is to create a group if a suitable one doesn't exist already, and then create reservations. After reservations are successfully created, they're immediately available for use with virtual machines (VMs). The capacity is reserved for your use as long as the reservation isn't deleted.

A well-formed request for a capacity reservation group should always succeed because it doesn't reserve any capacity. It just acts as a container for reservations. However, a request for capacity reservation could fail if you don't have the required quota for the VM series or if Azure doesn't have enough capacity to fulfill the request. Either request more quota or try a different VM size, location, or zone combination.

Capacity reservation creation succeeds or fails in its entirety. For a request to reserve 10 instances, success is returned only if all 10 instances could be allocated. Otherwise, the capacity reservation creation fails.

## Considerations

The capacity reservation must meet the following rules:

- The location parameter must match the location property for the parent capacity reservation group. A mismatch results in an error. 
- The VM size must be available in the target region. Otherwise, the reservation creation fails. 
- The subscription must have available quota equal to or more than the quantity of VMs being reserved for the VM series and for the region overall. If needed, [request more quota](/azure/quotas/per-vm-quota-requests). 
    - As needed to satisfy existing quota limits, you can do single VMs in stages. Create a capacity reservation with a smaller quantity and reallocate that quantity of VMs. This approach frees up quota to increase the quantity reserved and add more VMs. Alternatively, if the subscription uses different VM sizes in the same series, reserve and redeploy VMs for the first size. Then add a reservation to the group for another size and redeploy the VMs for the new size to the reservation group. Repeat the process until it's complete. 
    - For scale sets, available quota is required unless the scale set or you delete its VM instances, capacity is reserved, and the scale set instances are added by using reserved capacity. If the scale set is updated by using blue green deployment, then reserve the capacity and deploy the new scale set to the reserved capacity at the next update. 
- Each capacity reservation group can have exactly one reservation for a specific VM size. For example, you can create only one capacity reservation for the VM size `Standard_D2s_v3`. Attempting to create a second reservation for `Standard_D2s_v3` in the same capacity reservation group results in an error. However, you can create another reservation in the same group for other VM sizes, such as `Standard_D4s_v3` and `Standard_D8s_v3`.  
- For a capacity reservation group that supports zones, each reservation type is defined by the combination of **VM size** and **zone**. For example, one capacity reservation for `Standard_D2s_v3` in `Zone 1`, another capacity reservation for `Standard_D2s_v3` in `Zone 2`, and a third capacity reservation for `Standard_D2s_v3` in `Zone 3` are supported.

## Check VM sizes that are available for capacity reservation in a region

Before you create a capacity reservation, you can check the VM sizes that are available for the reservation for a particular region.

### [Portal](#tab/portal1)

<!-- no images necessary if steps are straightforward --> 

1. Open the [Azure portal](https://portal.azure.com).
1. In the search bar, enter **capacity reservation groups**.
1. Select **Capacity reservation groups** from the options.
1. Select **Create**.
1. On the **Basics** tab, create a capacity reservation group:
    1. Select a **Subscription**.
    1. Select or create a resource group.
    1. Name your group.
    1. Select a region.
    1. Optionally, select **Availability zones** or allow Azure to choose for you.
1. Select **Next**.
1. On **VM size**, select **See all sizes** to check what VM sizes are available for capacity reservation.

  ### [CLI](#tab/cli1)

Before you create a capacity reservation, you can check the reservation's available VM sizes for the region that you selected. The following example lists the capacity reservation VM sizes that are available in the East US location by using the Azure CLI:

   ```azurecli-interactive
    az vm list-skus -l eastus --resource-type virtualMachines --query "[?contains(capabilities[?name == 'CapacityReservationSupported' && value == 'True'].name,'CapacityReservationSupported')].name"
   ```

### [PowerShell](#tab/powershell1)

Before you create a capacity reservation, you can check the capacity reservation VM sizes that are available for the region you selected by using `Get-AzComputeResourceSku` for the `capability` property for the resource type `virtualMachines`. The following example lists the capacity reservation VM sizes that are available in the East US location:

   ```powershell-interactive
    $vmsizes = Get-AzComputeResourceSku -Location eastus | where {$_.ResourceType -eq "virtualMachines"}
    foreach($vmsize in $vmsizes) { foreach($capability in $vmsize.capabilities) {  if($capability.Name -eq 'CapacityReservationSupported' -and $capability.Value -eq 'true'){ $vmsize.name  } } }
   ```
--- 
<!-- The three dashes above show that your section of tabbed content is complete. Do not remove them :) -->

## Create a capacity reservation 

### [API](#tab/api1)

1. Create a capacity reservation group.

    To create a capacity reservation group, construct the following `PUT` request on the `Microsoft.Compute` provider: 
    
    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}&api-version=2021-04-01
    ``` 
    
    In the request body, include the following parameter:
    
    ```json
    { 
      "location":"eastus"
    } 
    ```
    
    This group is created to contain reservations for the US East location.
    
    The group in the following example only supports regional reservations because zones weren't specified at the time of creation. To create a zonal group, pass an extra parameter `zone` in the request body: 
    
    ```json
    { 
      "location":"eastus",
      "zones": ["1", "2", "3"] 
    } 
    ```
 
1. Create a capacity reservation.

    To create a reservation, construct the following `PUT` request on the `Microsoft.Compute` provider:
    
    ```rest
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName}?api-version=2021-04-01 
    ```
    
    In the request body, include the following parameters: 
    
    ```json
    { 
      "location": "eastus", 
      "sku": { 
        "name": "Standard_D2s_v3", 
        "capacity": 5 
      }, 
     "tags": { 
            "environment": "testing" 
    } 
    ```
    
    The preceding request creates a reservation in the East US location for five quantities of the D2s_v3 VM size. 


### [Portal](#tab/portal2)

<!-- no images necessary if steps are straightforward --> 

1. Open the [Azure portal](https://portal.azure.com).
1. In the search bar, enter **capacity reservation groups**.
1. Select **Capacity reservation groups** from the options.
1. Select **Create**.
1. On the **Basics** tab, create a capacity reservation group:
    1. Select a subscription.
    1. Select or create a resource group.
    1. Name your group.
    1. Select a region.
    1. Optionally, select **Availability zones** or allow Azure to choose for you.
1. Select **Next**.
1. On the **Reservations** tab, create at least one capacity reservation:
    1. Give each reservation a reservation name, the quantity of VM instances, and select a unique VM size.
    1. Billing information based on your selections appears in the **Cost/month** column.
1. Select **Next**.
1. On the **Tags** tab, optionally create tags.
1. Select **Next**.
1. On the **Review + Create** tab, review your capacity reservation group information.
1. Select **Create**.

### [CLI](#tab/cli2)

1. Before you can create a capacity reservation, create a resource group with `az group create`. The following example creates a resource group `myResourceGroup` in the East US location.

    ```azurecli-interactive
    az group create 
    -l eastus 
    -g myResourceGroup
    ```

1. Now create a capacity reservation group with `az capacity reservation group create`. The following example creates a group `myCapacityReservationGroup` in the East US location for all three availability zones.

    ```azurecli-interactive
    az capacity reservation group create 
    -n myCapacityReservationGroup 
    -l eastus 
    -g myResourceGroup 
    --zones 1 2 3 
    ```

1. After the capacity reservation group is created, create a new capacity reservation with `az capacity reservation create`. The following example creates `myCapacityReservation` for five quantities of Standard_D2s_v3 VM size in Zone 1 of the East US location.

    ```azurecli-interactive
    az capacity reservation create 
    -c myCapacityReservationGroup 
    -n myCapacityReservation 
    -l eastus 
    -g myResourceGroup 
    --sku Standard_D2s_v3 
    --capacity 5 
    --zone 1
    ```

### [PowerShell](#tab/powershell2)

1. Before you can create a capacity reservation, create a resource group with `New-AzResourceGroup`. The following example creates a resource group `myResourceGroup` in the East US location.

    ```powershell-interactive
    New-AzResourceGroup
    -ResourceGroupName "myResourceGroup"
    -Location "eastus"
    ```

1. Now create a capacity reservation group with `New-AzCapacityReservationGroup`. The following example creates a group `myCapacityReservationGroup` in the East US location for all three availability zones.

    ```powershell-interactive
    New-AzCapacityReservationGroup
    -ResourceGroupName "myResourceGroup"
    -Location "eastus"
    -Zone "1","2","3"
    -Name "myCapacityReservationGroup"
    ```

1. After the capacity reservation group is created, create a new capacity reservation with `New-AzCapacityReservation`. The following example creates `myCapacityReservation` for five quantities of Standard_D2s_v3 VM size in Zone 1 of the East US location.

    ```powershell-interactive
    New-AzCapacityReservation
    -ResourceGroupName "myResourceGroup"
    -Location "eastus"
    -Zone "1"
    -ReservationGroupName "myCapacityReservationGroup"
    -Name "myCapacityReservation"
    -Sku "Standard_D2s_v3"
    -CapacityToReserve 5
    ```

To learn more, see the Azure PowerShell commands [New-AzResourceGroup](/powershell/module/az.Resources/new-azresourcegroup), [New-AzCapacityReservationGroup](/powershell/module/az.compute/new-azcapacityreservationgroup), and [New-AzCapacityReservation](/powershell/module/az.compute/new-azcapacityreservation).


### [ARM template](#tab/arm1)

An [Azure Resource Manager template (ARM template)](/azure/azure-resource-manager/templates/overview) is a JSON file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment.

ARM templates let you deploy groups of related resources. In a single template, you can create a capacity reservation group and capacity reservations. You can deploy templates through the Azure portal, the Azure CLI, or Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines.

To create a zonal Capacity Reservation Group, see the following ARM template. Replace the resource names with your own. The following example creates a capacity reservation group called `myZonalCapacityReservationGroup` with capacity reservations of SKU D2s_v3 in all three zones.

Alternatively, you can remove the *zone* information to create a regional Capacity Reservation Group.


 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type":"string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Azure region for the resource"
            }
        },
          "Zones": {
            "type":"array",
            "defaultValue": ["1","2","3"],
            "metadata": {
                "description": "Zone(s) to use for Capacity Reservation Group"
            }
        },
        "CRGName": {
            "type":"string",
            "defaultValue": "myZonalCapacityReservationGroup",
            "metadata": {
                "description": "Name of Capacity Reservation Group"
            }
        },
        "CR1Name": {
            "type":"string",
            "defaultValue": "myCapacityReservation1",
            "metadata": {
                "description": "Name of Capacity Reservation 1"
            }
        },
        "CR1Zone": {
            "type":"array",
            "defaultValue": ["1"],
            "metadata": {
                "description": "Zone of Capacity Reservation 1"
            }
        },
        "CR2Name": {
            "type":"string",
            "defaultValue": "myCapacityReservation2",
            "metadata": {
                "description": "Name of Capacity Reservation 2"
            }
        },
         "CR2Zone": {
            "type":"array",
            "defaultValue": ["2"],
            "metadata": {
                "description": "Zone of Capacity Reservation 2"
            }
        },
        "CR3Name": {
            "type":"string",
            "defaultValue": "myCapacityReservation3",
            "metadata": {
                "description": "Name of Capacity Reservation 3"
            }
        },
         "CR3Zone": {
           "type":"array",
            "defaultValue": ["3"],
            "metadata": {
                "description": "Zone of Capacity Reservation 3"
            }
        },
        "SKU": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
                "description": "VM size to be reserved. Customers are allowed to create reservations for different VM sizes for every capacity reservation within a capacity reservation group"
            }
        },
         "quantityReserved": {
            "type": "string",
            "defaultValue": "2",
            "metadata": {
                "description": "Quantity of VMs to be reserved across all three Capacity Reservation. Customers are allowed to reserve different quantities across all 3 reservations"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/capacityReservationGroups",
            "name": "[parameters('CRGName')]",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "zones": "[parameters('Zones')]"    
        },
        {
            "type": "Microsoft.Compute/capacityReservationGroups/capacityReservations",
            "name": "[concat(parameters('CRGName'),'/',parameters('CR1Name'))]",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "zones": "[parameters('CR1Zone')]",  
            "dependsOn": ["[concat('Microsoft.Compute/capacityReservationGroups','/',parameters('CRGName'))]"],
            "sku":{
                "name": "[parameters('SKU')]",
                "capacity":"[parameters('quantityReserved')]"
            }  
        },
        {
            "type": "Microsoft.Compute/capacityReservationGroups/capacityReservations",
            "name": "[concat(parameters('CRGName'),'/',parameters('CR2Name'))]",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "zones": "[parameters('CR2Zone')]",
            "dependsOn": ["[concat('Microsoft.Compute/capacityReservationGroups','/',parameters('CRGName'))]"],
            "sku":{
                "name": "[parameters('SKU')]",
                "capacity":"[parameters('quantityReserved')]"
            }
        },
        {
            "type": "Microsoft.Compute/capacityReservationGroups/capacityReservations",
            "name": "[concat(parameters('CRGName'),'/',parameters('CR3Name'))]",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "zones": "[parameters('CR3Zone')]", 
            "dependsOn": ["[concat('Microsoft.Compute/capacityReservationGroups','/',parameters('CRGName'))]"],
            "sku":{
                "name": "[parameters('SKU')]",
                "capacity":"[parameters('quantityReserved')]"
            }
        }
    ]
}
```

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Do not remove them :) -->

## Check on your capacity reservation 

After the capacity reservation is successfully created, it's immediately available for use with VMs.

### [API](#tab/api2)

```rest
GET  
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/CapacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{capacityReservationName}?api-version=2021-04-01
```
 
```json
{ 
    "name": "<CapacityReservationName>", 
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}/capacityReservations/{CapacityReservationName}", 
    "type": "Microsoft.Compute/capacityReservationGroups/capacityReservations", 
    "location": "eastus", 
    "tags": { 
        "environment": "testing" 
    }, 
    "sku": { 
        "name": "Standard_D2s_v3", 
        "capacity": 5 
    }, 
    "properties": { 
        "reservationId": "<reservationId>", 
         "provisioningTime": "<provisioningTime>", 
         "provisioningState": "Updating" 
    } 
} 
```

### [CLI](#tab/cli3)

 ```azurecli-interactive
 az capacity reservation show 
 -c myCapacityReservationGroup 
 -n myCapacityReservation 
 -g myResourceGroup
 ```

### [PowerShell](#tab/powershell3)

Check on your capacity reservation:

```powershell-interactive
Get-AzCapacityReservation
-ResourceGroupName <"ResourceGroupName">
-ReservationGroupName <"CapacityReservationGroupName">
-Name <"CapacityReservationName">
```

To find the VM size and the quantity reserved, use the following command:

```powershell-interactive
$CapRes =
Get-AzCapacityReservation
-ResourceGroupName <"ResourceGroupName">
-ReservationGroupName <"CapacityReservationGroupName">
-Name <"CapacityReservationName">

$CapRes.sku
```

To learn more, see the Azure PowerShell command [Get-AzCapacityReservation](/powershell/module/az.compute/get-azcapacityreservation).

### [Portal](#tab/portal3)

1. Open the [Azure portal](https://portal.azure.com).
1. In the search bar, enter **capacity reservation groups**.
1. Select **Capacity reservation groups** from the options.
1. From the list, select the capacity reservation group name you created.
1. Select **Overview**.
1. Select **Reservations**.
1. In this view, you can see all the reservations in the group along with the VM size and quantity reserved.
--- 
<!-- The three dashes above show that your section of tabbed content is complete. Do not remove them :) -->

## Next step

> [!div class="nextstepaction"]
> [Learn how to modify your capacity reservation](capacity-reservation-modify.md)
