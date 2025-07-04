---
title: Tutorial - Install applications in a scale set with Azure PowerShell
description: Learn how to use Azure PowerShell to install applications into Virtual Machine Scale Sets with the Custom Script Extension
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: azure-virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 06/14/2024
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell

# Customer intent: "As a cloud administrator, I want to automate the installation and update of applications in Virtual Machine Scale Sets using scripts, so that I can efficiently manage and deploy applications across multiple VM instances."
---
# Tutorial: Install applications in Virtual Machine Scale Sets with Azure PowerShell
To run applications on virtual machine (VM) instances in a scale set, you first need to install the application components and required files. In a previous tutorial, you learned how to create and use a custom VM image to deploy your VM instances. This custom image included manual application installs and configurations. You can also automate the install of applications to a scale set after each VM instance is deployed, or update an application that already runs on a scale set. In this tutorial you learn how to:

> [!div class="checklist"]
> * Automatically install applications to your scale set
> * Use the Azure Custom Script Extension
> * Update a running application on a scale set

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## What is the Azure Custom Script Extension?
The Custom Script Extension downloads and executes scripts on Azure VMs. This extension is useful for post deployment configuration, software installation, or any other configuration / management task. Scripts can be downloaded from Azure storage or GitHub, or provided to the Azure portal at extension run-time.

The Custom Script extension integrates with Azure Resource Manager templates. It can also be used with the Azure CLI, Azure PowerShell, Azure portal, or the REST API. For more information, see the [Custom Script Extension overview](../virtual-machines/extensions/custom-script-windows.md).

To see the Custom Script Extension in action, create a scale set that installs the IIS web server and outputs the hostname of the scale set VM instance. The Custom Script Extension definition downloads a sample script from GitHub, installs the required packages, then writes the VM instance hostname to a basic HTML page.

## Create a scale set

Create a resource group with [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). The following example creates a resource group named *myResourceGroup* in the *East US* location:

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location "East US"
```

Now create a Virtual Machine Scale Set with [New-AzVmss](/powershell/module/az.compute/new-azvmss). To distribute traffic to the individual VM instances, a load balancer is also created. The load balancer includes rules to distribute traffic on TCP port 80. It also allows remote desktop traffic on TCP port 3389 and PowerShell remoting on TCP port 5985. When prompted, you can set your own administrative credentials for the VM instances in the scale set:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -OrchestrationMode "Flexible" `
  -Location "EastUS" `
  -UpgradePolicyMode "Manual" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" 
```

It takes a few minutes to create and configure all the scale set resources and VMs.

## Create Custom Script Extension definition
Azure PowerShell uses a hashtable to store the file to download and the command to execute. In the following example, a sample script from GitHub is used. First, create this configuration object as follows:

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```

Now, apply the Custom Script Extension with [Add-AzVmssExtension](/powershell/module/az.Compute/Add-azVmssExtension). The configuration object previously defined is passed to the extension. Update the extension on the scale set profile instances with [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

# Update the scale set
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss

```

## Add the extension to the existing scale set instances
Perform a manual upgrade to apply the updated extension to all the existing scale set instances. The update may take a couple of minutes to complete. 

```azurepowershell-interactive
Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "*"
```

Each VM instance in the scale set downloads and runs the script from GitHub. In a more complex example, multiple application components and files could be installed. If the scale set is scaled up, the new VM instances automatically apply the same Custom Script Extension definition and install the required application.

## Allow traffic to application
To allow access to the basic web application, create a network security group with [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) and [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). For more information, see [Networking for Azure Virtual Machine Scale Sets](virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive
#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroup" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroup" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet
```

## Test your scale set
To see your web server in action, get the public IP address of your load balancer with [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). The following example displays the IP address created in the *myResourceGroup* resource group:

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

Enter the public IP address of the load balancer in to a web browser. The load balancer distributes traffic to one of your VM instances, as shown in the following example:

![Basic web page in IIS](media/tutorial-install-apps-powershell/running-iis.png)

Leave the web browser open so that you can see an updated version in the next step.

## Change the upgrade policy
In the previous section, in order to apply the updated application to all the scale set instances, a manual upgrade was needed. To enable updates to be applied automatically to all existing scale set instances, update the upgrade policy from manual to automatic. For more information on upgrade policies, see [Upgrade policies for Virtual Machine Scale Sets](virtual-machine-scale-sets-upgrade-policy.md).

```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

Update-Azvmss `
    -ResourceGroupName "myResourceGroup" `
    -Name "myScaleSet" `
    -UpgradePolicyMode "Automatic" `
    -VirtualMachineScaleSet $vmss
```

## Update app deployment
Throughout the lifecycle of a scale set, you may need to deploy an updated version of your application. With the Custom Script Extension, you can reference an updated deploy script and then reapply the extension to your scale set.

Create a new config definition named *customConfigv2*. This definition runs an updated *v2* version of the application install script:

```azurepowershell-interactive
$customConfigv2 = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

Update the Custom Script Extension configuration to the VM instances in your scale set. The *customConfigv2* definition is used to apply the updated version of the application to the scale set:

```azurepowershell-interactive
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet"
 
$vmss.VirtualMachineProfile.ExtensionProfile[0].Extensions[0].Settings = $customConfigv2
 
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Because the scale set is now using an automatic upgrade policy, the updated application will automatically be applied to existing scale set instances. Refresh your web browser to see the updated application. To see the updated version, refresh the web site in your browser:

![Updated web page in IIS](media/tutorial-install-apps-powershell/running-iis-updated.png)

## Clean up resources
To remove your scale set and additional resources, delete the resource group and all its resources with [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). The `-Force` parameter confirms that you wish to delete the resources without an additional prompt to do so. The `-AsJob` parameter returns control to the prompt without waiting for the operation to complete.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```

## Next steps
In this tutorial, you learned how to automatically install and update applications on your scale set with Azure PowerShell:

> [!div class="checklist"]
> * Automatically install applications to your scale set
> * Use the Azure Custom Script Extension
> * Update a running application on a scale set

Advance to the next tutorial to learn how to automatically scale your scale set.

> [!div class="nextstepaction"]
> [Automatically scale your scale sets](tutorial-autoscale-powershell.md)
