---
title: Azure Windows VM Agent overview
description: Learn how to install and detect the Azure Windows VM Agent to manage your virtual machine's interaction with the Azure fabric controller.
ms.topic: how-to
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.collection: windows
ms.date: 08/18/2025
# Customer intent: "As a cloud administrator, I want to install and manage the Azure Windows VM Agent on my virtual machines, so that I can enable and utilize necessary extensions for configuration and recovery tasks effectively."
---

# Azure Windows VM Agent overview

The Microsoft Azure Windows VM Agent is a secure, lightweight process that manages virtual machine (VM) interaction with the Azure fabric controller. The Azure Windows VM Agent has a primary role in enabling and executing Azure virtual machine extensions. VM extensions enable post-deployment configuration of VMs, such as installing and configuring software. VM extensions also enable recovery features such as resetting the administrative password of a VM. Without the Azure Windows VM Agent, you can't run VM extensions.

This article describes how to install and detect the Azure Windows VM Agent.

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

## Prerequisites

The Azure Windows VM Agent supports the x64 and ARM64 architecture for these Windows operating systems:

| OS Version | x64 | ARM64 |
|:-----|:-----:|:-----:|
| Windows 10 | Supported | Unsupported |
| Windows 11 | Supported | Supported |
| Windows Server 2016 | Supported | Unsupported |
| Windows Server 2016 Core | Supported | Unsupported |
| Windows Server 2019 | Supported | Unsupported |
| Windows Server 2019 Core | Supported | Unsupported |
| Windows Server 2022 | Supported | Unsupported |
| Windows Server 2022 Core | Supported | Unsupported |
| Windows Server 2025 | Supported | Unsupported |
| Windows Server 2025 Core | Supported | Unsupported |

> [!IMPORTANT]
> - The Azure Windows VM Agent needs at least Windows Server 2016 (64-bit) to run, with the .NET Framework 4.0. See [Minimum version support for virtual machine agents in Azure](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).
>
> - Ensure that your VM has access to IP address 168.63.129.16. For more information, see [What is IP address 168.63.129.16?](/azure/virtual-network/what-is-ip-address-168-63-129-16).
>
> - If you only have one private IP on your VM's NIC, then we highly recommend having DHCP enabled in the guest VM.  If you need a static private IP address, you should configure it through the Azure portal or PowerShell, and make sure the DHCP option inside the VM is enabled. [Learn more](/azure/virtual-network/ip-services/virtual-networks-static-private-ip-arm-ps) about setting up a static IP address by using PowerShell. This will ensure that the IP configuration will always match what is configured on the VM in Azure.
>
> - If you have multiple private IPs assigned to your VM's NIC, then ensure that you carefully follow the steps to [assign the IP configurations correctly](/azure/virtual-network/ip-services/virtual-network-multiple-ip-addresses-portal#os-config). After this, if the Guest Agent isn't able to communicate with 168.63.129.16, then please check that the primary IP in Windows [matches the primary IP in your VM's NIC in Azure](/troubleshoot/azure/virtual-machines/windows/no-internet-access-multi-ip).
>
> - Running the Azure Windows VM Agent in a nested virtualization VM might lead to unpredictable behavior, so it's not supported in that dev/test scenario.

## Install the Azure Windows VM Agent

### Azure Marketplace image

The Azure Windows VM Agent is installed by default on any Windows VM deployed from an Azure Marketplace image. When you deploy an Azure Marketplace image from the Azure portal, PowerShell, the Azure CLI, or an Azure Resource Manager template, the Azure Windows VM Agent is also installed.

The Azure Windows VM Agent package has two parts:

- Azure Windows Provisioning Agent (PA)
- Azure Windows Guest Agent (WinGA)

To boot a VM, you must have the PA installed on the VM. However, the WinGA does not need to be installed. At VM deploy time, you can select not to install the WinGA. The following example shows how to select the `provisionVmAgent` option with an Azure Resource Manager template:

```json
{
	"resources": [{
		"name": ["parameters('virtualMachineName')"],
		"type": "Microsoft.Compute/virtualMachines",
		"apiVersion": "2016-04-30-preview",
		"location": ["parameters('location')"],
		"dependsOn": ["[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"],
		"properties": {
			"osProfile": {
				"computerName": ["parameters('virtualMachineName')"],
				"adminUsername": ["parameters('adminUsername')"],
				"adminPassword": ["parameters('adminPassword')"],
				"windowsConfiguration": {
					"provisionVmAgent": "false"
				}
			}
		}
	}]
}
```

If you don't have the agents installed, you can't use some Azure services, such as Azure Backup or Azure Security. These services require an extension to be installed. If you deploy a VM without the WinGA, you can install the latest version of the agent later.

### Manual installation

To manually install the Azure Windows VM Agent, [download the installer](https://github.com/Azure/WindowsVMAgent) and select the latest release. You can also search for a specific version in the [GitHub page for Azure Windows VM Agent releases](https://github.com/Azure/WindowsVMAgent/releases). The Azure Windows VM Agent is supported on Windows Server 2016 (64 bit) and later.

> [!NOTE]
> - If a VM was created from a custom or Marketplace image without `ProvisionVMAgent` enabled, then it is important to update the `AllowExtensionOperations` to `true` after you manually install the Azure Windows VM Agent.
> - This is NOT required if the VM was created from a specialized disk. For example, if the VM was created directly from an existing disk.

```powershell
$vm = Get-AzVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
$vm.OSProfile.AllowExtensionOperations = $true
$vm | Update-AzVM
```

## Detect the Azure Windows VM Agent

### PowerShell

You can use the Azure Resource Manager PowerShell module to get information about Azure VMs. To see information about a VM, such as the provisioning state for the Azure Windows VM Agent, use [Get-AzVM](/powershell/module/az.compute/get-azvm):

```powershell
Get-AzVM
```

The following condensed example output shows the `ProvisionVMAgent` property nested inside `OSProfile`. You can use this property to determine if the VM agent has been deployed to the VM.

```powershell
OSProfile                  :
  ComputerName             : myVM
  AdminUsername            : myUserName
  WindowsConfiguration     :
    ProvisionVMAgent       : True
    EnableAutomaticUpdates : True
```

Use the following script to return a concise list of VM names (running Windows OS) and the state of the Azure Windows VM Agent:

```powershell
$vms = Get-AzVM

foreach ($vm in $vms) {
    $agent = $vm | Select -ExpandProperty OSProfile | Select -ExpandProperty Windowsconfiguration | Select ProvisionVMAgent
    Write-Host $vm.Name $agent.ProvisionVMAgent
}
```

Use the following script to return a concise list of VM names (running Linux OS) and the state of the Azure Windows VM Agent:

```powershell
$vms = Get-AzVM

foreach ($vm in $vms) {
    $agent = $vm | Select -ExpandProperty OSProfile | Select -ExpandProperty Linuxconfiguration | Select ProvisionVMAgent
    Write-Host $vm.Name $agent.ProvisionVMAgent
}
```

### Manual detection

When you're logged in to a Windows VM, you can use Task Manager to examine running processes. To check for the Azure Windows VM Agent, open Task Manager, select the **Details** tab, and look for a process named *WindowsAzureGuestAgent.exe*. The presence of this process indicates that the VM agent is installed.

## Upgrade the Azure Windows VM Agent

The Azure Windows VM Agent for Windows is automatically upgraded on images deployed from Azure Marketplace. The new versions are stored in Azure Storage, so ensure that you don't have firewalls blocking access. As new VMs are deployed to Azure, they receive the latest VM agent at VM provision time. If you installed the agent manually or are deploying custom VM images, you need to manually update to include the new VM agent at image creation time.

## Azure Windows Guest Agent automatic log collection

The Azure Windows Guest Agent has a feature to automatically collect some logs. The *CollectGuestLogs.exe* process controls this feature. It exists for both platform as a service (PaaS) cloud services and infrastructure as a service (IaaS) VMs. Its goal is to quickly and automatically collect diagnostics logs from a VM, so they can be used for offline analysis.

The collected logs are event logs, OS logs, Azure logs, and some registry keys. The agent produces a ZIP file that's transferred to the VM's host. Engineering teams and support professionals can then use this ZIP file to investigate issues on the request of the customer who owns the VM.

## Azure Windows Guest Agent and OSProfile certificates

The Azure Windows VM Agent installs the certificates referenced in the `OSProfile` value of a VM or a virtual machine scale set. If you manually remove these certificates from the Microsoft Management Console (MMC) Certificates snap-in inside the guest VM, the Azure Windows Guest Agent will add them back. To permanently remove a certificate, you have to remove it from `OSProfile`, and then remove it from within the guest operating system.

For a virtual machine, use [Remove-AzVMSecret](/powershell/module/az.compute/remove-azvmsecret) to remove certificates from `OSProfile`.

For more information on certificates for virtual machine scale sets, see [Azure Virtual Machine Scale Sets - How do I remove deprecated certificates?](../../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.yml#how-do-i-remove-deprecated-certificates-).

## Next steps

For more information about VM extensions, see [Azure virtual machine extensions and features](overview.md).
