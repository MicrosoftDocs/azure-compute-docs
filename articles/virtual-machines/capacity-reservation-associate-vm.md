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
# Customer intent: "As a cloud administrator, I want to associate a virtual machine with a capacity reservation group, so that I can ensure reserved resources are allocated for my applications and workloads."
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

To associate a VM with a Capacity Reservation Group, see the following ARM template. Replace the resource names with your own. The following example creates a VM of SKU D2s_v3 and associates it to a capacity reservation group.

Alternatively, you can remove the *zone* information to create a regional VM and associate it to a Capacity Reservation Group.

 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "adminUsername": {
            "type": "string",
            "metadata": {
                "description": "Username for the Virtual Machine."
            }
        },
        "adminPassword": {
            "type": "securestring",
            "minLength": 12,
            "metadata": {
                "description": "Password for the Virtual Machine."
            }
        },
        "dnsLabelPrefix": {
            "type": "string",
            "defaultValue": "[toLower(concat(parameters('vmName'),'-', uniqueString(resourceGroup().id, parameters('vmName'))))]",
            "metadata": {
                "description": "Unique DNS Name for the Public IP used to access the Virtual Machine."
            }
        },
        "publicIpName": {
            "type": "string",
            "defaultValue": "myPublicIP",
            "metadata": {
                "description": "Name for the Public IP used to access the Virtual Machine."
            }
        },
        "publicIPAllocationMethod": {
            "type": "string",
            "defaultValue": "Dynamic",
            "allowedValues": [
                "Dynamic",
                "Static"
            ],
            "metadata": {
                "description": "Allocation method for the Public IP used to access the Virtual Machine."
            }
        },
        "publicIpSku": {
            "type": "string",
            "defaultValue": "Basic",
            "allowedValues": [
                "Basic",
                "Standard"
            ],
            "metadata": {
                "description": "SKU for the Public IP used to access the Virtual Machine."
            }
        },
        "OSVersion": {
            "type": "string",
            "defaultValue": "2019-Datacenter",
            "allowedValues": [
                "2008-R2-SP1",
                "2012-Datacenter",
                "2012-R2-Datacenter",
                "2016-Nano-Server",
                "2016-Datacenter-with-Containers",
                "2016-Datacenter",
                "2019-Datacenter",
                "2019-Datacenter-Core",
                "2019-Datacenter-Core-smalldisk",
                "2019-Datacenter-Core-with-Containers",
                "2019-Datacenter-Core-with-Containers-smalldisk",
                "2019-Datacenter-smalldisk",
                "2019-Datacenter-with-Containers",
                "2019-Datacenter-with-Containers-smalldisk"
            ],
            "metadata": {
                "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version."
            }
        },
        "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
                "description": "Size of the virtual machine."
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "vmName": {
            "type": "string",
            "defaultValue": "myD2v3VM",
            "metadata": {
                "description": "Name of the virtual machine."
            }
        }
    },
    "variables": {
        "storageAccountName": "[concat('bootdiags', uniquestring(resourceGroup().id))]",
        "nicName": "myVMNic",
        "addressPrefix": "10.0.0.0/16",
        "subnetName": "Subnet",
        "subnetPrefix": "10.0.0.0/24",
        "virtualNetworkName": "MyVNET",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "networkSecurityGroupName": "default-NSG"
    },
    "resources": [{
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[variables('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        }, {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2020-06-01",
            "name": "[parameters('publicIPName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('publicIpSku')]"
            },
            "properties": {
                "publicIPAllocationMethod": "[parameters('publicIPAllocationMethod')]",
                "dnsSettings": {
                    "domainNameLabel": "[parameters('dnsLabelPrefix')]"
                }
            }
        }, {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-06-01",
            "name": "[variables('networkSecurityGroupName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [{
                        "name": "default-allow-3389",
                        "properties": {
                            "priority": 1000,
                            "access": "Allow",
                            "direction": "Inbound",
                            "destinationPortRange": "3389",
                            "protocol": "Tcp",
                            "sourcePortRange": "*",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*"
                        }
                    }
                ]
            }
        }, {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2020-06-01",
            "name": "[variables('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('addressPrefix')]"
                    ]
                },
                "subnets": [{
                        "name": "[variables('subnetName')]",
                        "properties": {
                            "addressPrefix": "[variables('subnetPrefix')]",
                            "networkSecurityGroup": {
                                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
                            }
                        }
                    }
                ]
            }
        }, {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2020-06-01",
            "name": "[variables('nicName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
            ],
            "properties": {
                "ipConfigurations": [{
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPName'))]"
                            },
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            }
                        }
                    }
                ]
            }
        }, {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2020-06-01",
            "name": "[parameters('vmName')]",
            "location": "[parameters('location')]",
            "zones": "[parameters('Zones')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
                "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('vmSize')]"
                },
                "capacityReservation": {
                    "capacityReservationGroup": {
                        "id": "subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}"
                    }
                },
                "osProfile": {
                    "computerName": "[parameters('vmName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "[parameters('OSVersion')]",
                        "version": "latest"
                    },
                    "osDisk": {
                        "createOption": "FromImage",
                        "managedDisk": {
                            "storageAccountType": "StandardSSD_LRS"
                        }
                    },
                    "dataDisks": [{
                            "diskSizeGB": 1023,
                            "lun": 0,
                            "createOption": "Empty"
                        }
                    ]
                },
                "networkProfile": {
                    "networkInterfaces": [{
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                        }
                    ]
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true,
                        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))).primaryEndpoints.blob]"
                    }
                }
            }
        }
    ],
    "outputs": {
        "hostname": {
            "type": "string",
            "value": "[reference(parameters('publicIPName')).dnsSettings.fqdn]"
        }
    }
}

```

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->


## Associate an existing virtual machine

Existing zonal virtual machines can simply be updated with Capacity Reservation Group property without the need of deallocation and it will automatically consume the capacity reservation. Existing regional virtual machines can include similar process but must be reallocated.

### Regional Virtual Machine

For the initial release of a capacity reservation, a virtual machine must be allocated to a capacity reservation:

1. Follow guidance to create a capacity reservation group and capacity reservation, if necessary. Or increment the quantity of an existing capacity reservation so there's unused reserved capacity.
1. Deallocate the virtual machine.
1. Update the capacity reservation group property on the VM.
1. Restart the VM.

Follow the steps to associate the virtual machine to a capacity reservation group.

#### [API](#tab/api2)

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


#### [Portal](#tab/portal2)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your VM.
1. Select **Overview**.
1. Select **Stop** at the top of the page to deallocate the VM.
1. Go to **Configurations** on the left.
1. In the **Capacity Reservation group** dropdown list, select the group that you want to associate to the VM.

#### [CLI](#tab/cli2)

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

#### [PowerShell](#tab/powershell2)

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

### Zonal Virtual Machine

To add an existing running zonal virtual machine to capacity reservation group, simply update the VM to use a matching capacity reservation group. A virtual machine must be allocated to a capacity reservation. 

> [!IMPORTANT]
> The feature to add an existing zonal VM to a CRG without the need of deallocation, is currently in **Preview**. See the [Preview Terms of Use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.
> The feature is currently not available in Azure for Government and Azure in China.

1. Follow guidance to create a capacity reservation group and capacity reservation, if necessary. Or increment the quantity of an existing capacity reservation so there's unused reserved capacity.
1. Update the capacity reservation group property on the VM.

Follow the steps to associate the virtual machine to a capacity reservation group.

#### [API](#tab/api2)

Add the `capacityReservationGroup` property to the VM. Construct the following `PUT` request to `Microsoft.Compute` provider:

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


#### [Portal](#tab/portal2)

1. Open the [Azure portal](https://portal.azure.com).
1. Go to your VM.
1. Select **Overview**.
1. Go to **Configurations** on the left.
1. In the **Capacity Reservation group** dropdown list, select the group that you want to associate to the VM.

#### [CLI](#tab/cli2)

Associate the VM to a capacity reservation group:

```azurecli-interactive
    az vm update 
    -g myresourcegroup 
    -n myVM 
    --capacity-reservation-group subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}
```

#### [PowerShell](#tab/powershell2)

Associate the VM to a capacity reservation group:

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

## View VM allocation with the Instance View

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
