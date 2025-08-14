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
# Customer intent: "As a cloud architect, I want to associate a virtual machine scale set using Uniform Orchestration with a capacity reservation group, so that I can efficiently manage resources and ensure availability for my workloads in Azure."
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

To associate a Virtual Machine Scale Set with a Capacity Reservation Group, see the following ARM template. Replace the resource names with your own. The following example creates a VM scale set of SKU D2s_v3 and associates to a capacity reservation group.

 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmSku": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
                "description": "Size of VMs in the VM Scale Set."
            }
        },
        "windowsOSVersion": {
            "type": "string",
            "defaultValue": "2019-Datacenter",
            "allowedValues": [
                "2008-R2-SP1",
                "2012-Datacenter",
                "2012-R2-Datacenter",
                "2016-Datacenter",
                "2019-Datacenter"
            ],
            "metadata": {
                "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version. Allowed values: 2008-R2-SP1, 2012-Datacenter, 2012-R2-Datacenter & 2016-Datacenter, 2019-Datacenter."
            }
        },
        "vmssName": {
            "type": "string",
            "minLength": 3,
            "maxLength": 61,
            "metadata": {
                "description": "String used as a base for naming resources. Must be 3-61 characters in length and globally unique across Azure. A hash is prepended to this string for some resources, and resource-specific information is appended."
            }
        },
        "instanceCount": {
            "type": "int",
            "defaultValue": 1,
            "minValue": 1,
            "maxValue": 100,
            "metadata": {
                "description": "Number of VM instances (100 or less)."
            }
        },
        "singlePlacementGroup": {
            "type": "bool",
            "defaultValue": false,
            "metadata": {
                "description": "When true this limits the scale set to a single placement group, of max size 100 virtual machines. NOTE: If singlePlacementGroup is true, it may be modified to false. However, if singlePlacementGroup is false, it may not be modified to true."
            }
        },
        "adminUsername": {
            "type": "string",
            "defaultValue": "vmssadmin",
            "metadata": {
                "description": "Admin username on all VMs."
            }
        },
        "adminPassword": {
            "type": "securestring",
            "metadata": {
                "description": "Admin password on all VMs."
            }
        },
        "_artifactsLocation": {
            "type": "string",
            "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/",
            "metadata": {
                "description": "The base URI where artifacts required by this template are located. For example, if stored on a public GitHub repo, you'd use the following URI: https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/."
            }
        },
        "_artifactsLocationSasToken": {
            "type": "securestring",
            "defaultValue": "",
            "metadata": {
                "description": "The sasToken required to access _artifactsLocation.  If your artifacts are stored on a public repo or public storage account you can leave this blank."
            }
        },
        "powershelldscZip": {
            "type": "string",
            "defaultValue": "DSC/InstallIIS.zip",
            "metadata": {
                "description": "Location of the PowerShell DSC zip file relative to the URI specified in the _artifactsLocation, i.e. DSC/IISInstall.ps1.zip"
            }
        },
        "webDeployPackage": {
            "type": "string",
            "defaultValue": "WebDeploy/DefaultASPWebApp.v1.0.zip",
            "metadata": {
                "description": "Location of the  of the WebDeploy package zip file relative to the URI specified in _artifactsLocation, i.e. WebDeploy/DefaultASPWebApp.v1.0.zip"
            }
        },
        "powershelldscUpdateTagVersion": {
            "type": "string",
            "defaultValue": "1.0",
            "metadata": {
                "description": "Version number of the DSC deployment. Changing this value on subsequent deployments will trigger the extension to run."
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "platformFaultDomainCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
                "description": "Fault Domain count for each placement group."
            }
        }
    },
    "variables": {
        "namingInfix": "[toLower(substring(concat(parameters('vmssName'), uniqueString(resourceGroup().id)), 0, 9))]",
        "longNamingInfix": "[toLower(parameters('vmssName'))]",
        "addressPrefix": "10.0.0.0/16",
        "subnetPrefix": "10.0.0.0/24",
        "virtualNetworkName": "[concat(variables('namingInfix'), 'vnet')]",
        "publicIPAddressName": "[concat(variables('namingInfix'), 'pip')]",
        "subnetName": "[concat(variables('namingInfix'), 'subnet')]",
        "loadBalancerName": "[concat(variables('namingInfix'), 'lb')]",
        "publicIPAddressID": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]",
        "lbProbeID": "[resourceId('Microsoft.Network/loadBalancers/probes',variables('loadBalancerName'), 'tcpProbe')]",
        "natPoolName": "[concat(variables('namingInfix'), 'natpool')]",
        "bePoolName": "[concat(variables('namingInfix'), 'bepool')]",
        "lbPoolID": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools',variables('loadBalancerName'),variables('bePoolName'))]",
        "natStartPort": 50000,
        "natEndPort": 50119,
        "natBackendPort": 3389,
        "nicName": "[concat(variables('namingInfix'), 'nic')]",
        "ipConfigName": "[concat(variables('namingInfix'), 'ipconfig')]",
        "frontEndIPConfigID": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations',variables('loadBalancerName'),'loadBalancerFrontEnd')]",
        "osType": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "[parameters('windowsOSVersion')]",
            "version": "latest"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "[parameters('windowsOSVersion')]",
            "version": "latest"
          },
        "webDeployPackageFullPath": "[uri(parameters('_artifactsLocation'), concat(parameters('webDeployPackage'), parameters('_artifactsLocationSasToken')))]",
        "powershelldscZipFullPath": "[uri(parameters('_artifactsLocation'), concat(parameters('powershelldscZip'), parameters('_artifactsLocationSasToken')))]"
    },
    "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2020-06-01",
            "name": "[variables('loadBalancerName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]"
            ],
            "properties": {
                "frontendIPConfigurations": [
                    {
                        "name": "LoadBalancerFrontEnd",
                        "properties": {
                            "publicIPAddress": {
                                "id": "[variables('publicIPAddressID')]"
                            }
                        }
                    }
                ],
                "backendAddressPools": [
                    {
                        "name": "[variables('bePoolName')]"
                    }
                ],
                "inboundNatPools": [
                    {
                        "name": "[variables('natPoolName')]",
                        "properties": {
                            "frontendIPConfiguration": {
                                "id": "[variables('frontEndIPConfigID')]"
                            },
                            "protocol": "Tcp",
                            "frontendPortRangeStart": "[variables('natStartPort')]",
                            "frontendPortRangeEnd": "[variables('natEndPort')]",
                            "backendPort": "[variables('natBackendPort')]"
                        }
                    }
                ],
                "loadBalancingRules": [
                    {
                        "name": "LBRule",
                        "properties": {
                            "frontendIPConfiguration": {
                                "id": "[variables('frontEndIPConfigID')]"
                            },
                            "backendAddressPool": {
                                "id": "[variables('lbPoolID')]"
                            },
                            "protocol": "Tcp",
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 5,
                            "probe": {
                                "id": "[variables('lbProbeID')]"
                            }
                        }
                    }
                ],
                "probes": [
                    {
                        "name": "tcpProbe",
                        "properties": {
                            "protocol": "Tcp",
                            "port": 80,
                            "intervalInSeconds": 5,
                            "numberOfProbes": 2
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "apiVersion": "2020-06-01",
            "name": "[variables('namingInfix')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('vmSku')]",
                "tier": "Standard",
                "capacity": "[parameters('instanceCount')]"
            },
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', variables('loadBalancerName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
            ],
            "properties": {
                "overprovision": true,
                "upgradePolicy": {
                    "mode": "Automatic"
                },
                "singlePlacementGroup": "[parameters('singlePlacementGroup')]",
                "platformFaultDomainCount": "[parameters('platformFaultDomainCount')]",
                "virtualMachineProfile": {
                    "storageProfile": {
                        "osDisk": {
                            "caching": "ReadWrite",
                            "createOption": "FromImage"
                        },
                        "imageReference": "[variables('imageReference')]"
                    },
                    "osProfile": {
                        "computerNamePrefix": "[variables('namingInfix')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]"
                    },
                    "networkProfile": {
                        "networkInterfaceConfigurations": [
                            {
                                "name": "[variables('nicName')]",
                                "properties": {
                                    "primary": true,
                                    "ipConfigurations": [
                                        {
                                            "name": "[variables('ipConfigName')]",
                                            "properties": {
                                                "subnet": {
                                                    "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
                                                },
                                                "loadBalancerBackendAddressPools": [
                                                    {
                                                        "id": "[variables('lbPoolID')]"
                                                    }
                                                ],
                                                "loadBalancerInboundNatPools": [
                                                    {
                                                        "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatPools', variables('loadBalancerName'),  variables('natPoolName'))]"
                                                    }
                                                ]
                                            }
                                        }
                                    ]
                                }
                            }
                        ]
                    },
                    "capacityReservation": {
                      "capacityReservationGroup": {
                         "id":"subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName}"
                      }
                    }
                }
            }
        },
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2020-06-01",
            "name": "[variables('publicIPAddressName')]",
            "location": "[parameters('location')]",
            "properties": {
                "publicIPAllocationMethod": "Static",
                "dnsSettings": {
                    "domainNameLabel": "[variables('longNamingInfix')]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2020-06-01",
            "name": "[variables('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('addressPrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[variables('subnetName')]",
                        "properties": {
                            "addressPrefix": "[variables('subnetPrefix')]"
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Insights/autoscaleSettings",
            "apiVersion": "2015-04-01",
            "name": "autoscalehost",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Compute/virtualMachineScaleSets/', variables('namingInfix'))]"
            ],
            "properties": {
                "name": "autoscalehost",
                "targetResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', variables('namingInfix'))]",
                "enabled": true,
                "profiles": [
                    {
                        "name": "Profile1",
                        "capacity": {
                            "minimum": "1",
                            "maximum": "10",
                            "default": "1"
                        },
                        "rules": [
                            {
                                "metricTrigger": {
                                    "metricName": "Percentage CPU",
                                    "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', variables('namingInfix'))]",
                                    "timeGrain": "PT1M",
                                    "statistic": "Average",
                                    "timeWindow": "PT5M",
                                    "timeAggregation": "Average",
                                    "operator": "GreaterThan",
                                    "threshold": 50
                                },
                                "scaleAction": {
                                    "direction": "Increase",
                                    "type": "ChangeCount",
                                    "value": "1",
                                    "cooldown": "PT5M"
                                }
                            },
                            {
                                "metricTrigger": {
                                    "metricName": "Percentage CPU",
                                    "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', variables('namingInfix'))]",
                                    "timeGrain": "PT1M",
                                    "statistic": "Average",
                                    "timeWindow": "PT5M",
                                    "timeAggregation": "Average",
                                    "operator": "LessThan",
                                    "threshold": 30
                                },
                                "scaleAction": {
                                    "direction": "Decrease",
                                    "type": "ChangeCount",
                                    "value": "1",
                                    "cooldown": "PT5M"
                                }
                            }
                        ]
                    }
                ]
            }
        }
    ],
    "outputs": {
        "applicationUrl": {
            "type": "string",
            "value": "[concat('http://', reference(variables('publicIPAddressName')).dnsSettings.fqdn, '/MyApp')]"
        }
    }
}

```
### [Portal](#tab/portal1)
	
<!-- no images necessary if steps are straightforward --> 
	
1. Open the [Azure portal](https://portal.azure.com).
1. In the search bar, enter **virtual machine scale sets**.
1. On the **Virtual machines scale sets** page, select **Create** 
1. On the **Basics** tab, under **Project details**, select the correct subscription. Then choose to create a new resource group or use an existing one.
1. Under **Scale set details**, enter the Virtual Machine Scale Set name and choose your region.
1. Under **Orchestration mode**, choose the orchestration needed.
1. Select an **Image** and the VM size.
1. Under **Administrator account**, enter a username and a password. The password must be at least 12 characters long and meet the defined complexity requirements.
1.Go to the **Management section** and select the **Upgrade mode** needed
1. Go to the **Advanced section**.
1. In the **Capacity Reservations** dropdown list, select the capacity reservation group that you want to associate to the scale set.
1. Select **Review + create**.
1. After validation runs, select **Create**.
1. After the deployment is finished, select **Go to resource**.

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

### [Portal](#tab/portal2)
	
<!-- no images necessary if steps are straightforward --> 
	
1. Open the [Azure portal](https://portal.azure.com).
1. Go to your Virtual Machine Scale Set.
1. Select **Overview**.
1. Select **Stop** at the top of the page to deallocate the scale set.
1. Go to **Configurations** on the left.
1. In the **Capacity Reservation group** dropdown list, select the group that you want to associate to the scale set.

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
