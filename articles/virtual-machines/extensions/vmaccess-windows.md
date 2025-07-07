---
title: Reset access to an Azure Windows VM 
description: Learn how to manage administrative users and reset access on Windows VMs by using the VMAccess extension and the Azure PowerShell.
ms.topic: concept-article
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.collection: linux
ms.date: 02/25/2025
# Customer intent: As a system administrator, I want to reset administrative access to an Azure Windows VM using the VMAccess extension, so that I can regain control and manage user permissions effectively after encountering access issues.
---

# VMAccess Extension for Windows

The VMAccess Extension is used to manage administrative users, configure RDP, and check or repair disks on Azure Windows virtual machines. The extension integrates with Azure Resource Manager templates. It can also be invoked using Azure CLI, Azure PowerShell, the Azure portal, and the Azure Virtual Machines REST API.

This article describes how to run the VMAccess Extension from the Azure PowerShell and through an Azure Resource Manager template. This article also provides troubleshooting steps for Windows systems.

> [!NOTE]
> If you use the VMAccess extension to reset the password of your VM after you install the Microsoft Entra Login extension, rerun the Microsoft Entra Login extension to re-enable Microsoft Entra Login for your VM.

## Prerequisites

### Supported Windows versions

| OS Version | x64 | ARM64 |
|:-----|:-----:|:-----:|
| Windows 10 | Supported | Supported |
| Windows 11 | Supported | Supported |
| Windows Server 2016 | Supported | Supported |
| Windows Server 2016 Core | Supported | Supported |
| Windows Server 2019 | Supported | Supported |
| Windows Server 2019 Core | Supported | Supported |
| Windows Server 2022 | Supported | Supported |
| Windows Server 2022 Core | Supported | Supported |
| Windows Server 2025 | Supported | Supported |
| Windows Server 2025 Core | Supported | Supported |

### Tips

- VMAccess is designed for regaining access to a VM when access is lost. Based on this principle, it grants administrator privileges to the specified account in the username field. If you don't want a user to have admin permissions, log in to the VM and use built-in tools (such as `net user` and `Local Users and Groups`) to manage user privileges.
- You can only have one version of the extension applied to a VM. To run a second action, update the existing extension with a new configuration.
- When updating a user, VMAccess modifies the Remote Desktop settings to allow login.
- Not supported on Domain Controllers


## Extension schema

The VMAccess Extension configuration includes settings for username, passwords, and resetting the administrator password. You can store this information in configuration files, specify it in PowerShell, or include it in an Azure Resource Manager (ARM) template. The following JSON schema contains all the properties available to use in public and protected settings.

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "<name>",
  "apiVersion": "2023-09-01",
  "location": "<location>",
  "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "VMAccessAgent",
    "typeHandlerVersion": "2.4",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "username": "<username>"
    },
    "protectedSettings": {
      "password": "<password>"
    } 
  }
}
```

## Property values

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |
| apiVersion | 2023-09-01 | date |
| publisher | Microsoft.Compute | string |
| type | VMAccessAgent | string |
| typeHandlerVersion | 2.4 | int |

## Settings property values

| Name | Data Type | Description |
| ---- | ---- | ---- |
| username | string | The name of the user to manage (required for all actions on a user account). |
| password | string | The password to set for the user account. |

## Template deployment

Azure VM Extensions can be deployed with Azure Resource Manager (ARM) templates. The JSON schema detailed in the previous section can be used in an ARM template to run the VMAccess Extension during the template's deployment. 

The JSON configuration for a virtual machine extension must be nested inside the virtual machine resource fragment of the template, specifically `"resources": []` object for the virtual machine template and for a virtual machine scale set under `"virtualMachineProfile":"extensionProfile":{"extensions" :[]` object.

## PowerShell 

### Set-AzVMAccessExtension - Reset Password
[Set-AzVMAccessExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmaccessextension?view=azps-5.7.0) 

```powershell
Set-AzVMAccessExtension `
    -ResourceGroupName "myRG" `
    -Location "myLocation" `
    -VMName "myVM" `
    -Credential (get-credential) `
    -typeHandlerVersion "2.0" `
    -Name VMAccessAgent
```
### Set-AzVMAccessExtension - Reset RDP Configuration

```powershell
Set-AzVMAccessExtension `
    -ResourceGroupName "myRG" `
    -VMName "myVM" `
    -Name "myVMAccess" `
    -Location "myLocation" `
    -typeHandlerVersion "2.0" `
    -ForceRerun $true
```

### Set-AzVMExtension - Reset Password
[Set-AzVMExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmextension?view=azps-5.7.0)

```powershell
$Publicsettings = '{"UserName": "myuser"}'
$protectedSettings = '{"Password": "myPassWord"}'

Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -Location "myLocation" `
  -Name "VMAccessAgent" `
  -Publisher "Microsoft.Compute" `
  -ExtensionType "VMAccessAgent" `
  -TypeHandlerVersion "2.0" `
  -ProtectedSettingString $PrivateConf `
  -SettingString $settings
```

### Set-AzVMExtension - Reset RDP Configuration
[Set-AzVMExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmextension?view=azps-5.7.0)

```powershell
Set-AzVMExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -Location "myLocation" `
  -Name "VMAccessAgent" `
  -Publisher "Microsoft.Compute" `
  -ExtensionType "VMAccessAgent" `
  -TypeHandlerVersion "2.0" `
  -SettingString '{}' `
  -ForceRerun $true
```

## CLI

### Azure CLI - Reset Password
[az vm extension set](https://docs.microsoft.com/cli/azure/vm/extension?view=azure-cli-latest#az_vm_extension_set)

```
az vm extension set --name VMAccessAgent --publisher Microsoft.Compute --version 2.0 --vm-name "myVM" --resource-group "myRG" --settings '{"username":"myuser"}' --protected-settings '{"password":"myPassWord"}'
```

### Azure CLI - Reset RDP Configuration

```
az vm extension set --name VMAccessAgent --publisher Microsoft.Compute --version 2.0 --vm-name "myVM" --resource-group "myRG" --settings '{}'
```

## Virtual machine scale sets

### PowerShell

```powershell
$resourceGroupName = "<resource-group>"
$vmssName = "<vmss-name>"

$settings = @{"UserName"= "myuser"}
$protectedSettings = @{"Password"= "myPassWord"}

$vmss = Get-AzVmss `
    -ResourceGroupName $resourceGroupName `
    -VMScaleSetName $vmssName

Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
    -Name "VMAccessAgent" `
    -Publisher "Microsoft.Compute" `
    -Type "VMAccessAgent" `
    -TypeHandlerVersion "2.0" `
    -AutoUpgradeMinorVersion $true `
    -ProtectedSetting $protectedSettings
    -Setting $settings

Update-AzVmss `
    -ResourceGroupName $resourceGroupName `
    -Name $vmssName `
    -VirtualMachineScaleSet $vmss
```

### CLI
```
az vmss extension set --vmss-name my-vmss --name VMAccessAgent --resource-group my-group --version 2.0 --publisher Microsoft.Compute --settings '{"username":"myuser"}' --protected-settings '{"password":"myPassWord"}'
```


## Troubleshoot and support

The VMAccess extension logs exist locally on the VM and are particularly useful for troubleshooting.

| Location | Description |
| --- | --- |
| `C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.VMAccessAgent\` | Contains logs from the Windows Agent that show when an update to the extension occurred. Verify these logs to ensure the extension ran successfully. |
| `C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.VMAccessAgent\<version>\` | The VMAccess Extension produces detailed logs in this directory. It includes `CommandExecution.log`, which records each command executed along with its result, and `extension.log`, which contains individual execution logs. |
| `C:\WindowsAzure\Logs\WaAppAgent\` | Contains configuration details and binary logs related to the VMAccess extension. |

You can also retrieve the execution state of the VMAccess Extension by running the following command:

```powershell
Get-AzVMExtension -ResourceGroupName "RG Name" -VMName "VM Name" -Name "Extension Name"
```

### Error messages

| Error  | Description |
| ---- | ---- |
| "internalErrorCode": "CannotModifyExtensionsWhenVMNotRunning", "code": "OperationNotAllowed","message": "Cannot modify extensions in the VM when the VM is not running." | The error indicates that the operation to modify extensions in the VM is not allowed because the VM is not running. Ensure that the VM is in a running state before attempting to modify extensions. |
| Error message: 'VMAccess Extension does not support Domain Controller.' | The error indicates that the VM extension 'enablevmAccess' failed because it does not support Domain Controller. Ensure that the VM is not configured as a Domain Controller when using this extension. For more information, see [Reset Remote Desktop Services or its administrator password in a Windows VM](/troubleshoot/azure/virtual-machines/windows/reset-rdp). |
| VM 'vmname' has not reported status for VM agent or extensions. Verify that the OS is up and healthy, the VM has a running VM agent, and that it can establish outbound connections to Azure storage. | See [Troubleshoot Azure Windows VM Agent issues](/troubleshoot/azure/virtual-machines/windows/windows-azure-guest-agent?source=recommendations#troubleshooting-checklist). |
| Error message: 'Cannot update Remote Desktop Connection settings for Administrator account. The password does not meet the password policy requirements. Check the minimum password length, password complexity, and password history requirements. | The error indicates that the VM extension 'enablevmAccess' failed to update Remote Desktop Connection settings for the Administrator account due to a password policy violation. Ensure that the password meets Windows password policy requirements, including minimum length, complexity, and history. |
| The Admin User Account password cannot be null or empty if provided the username. | The error indicates that the VM extension 'enablevmAccess' failed because the Admin User Account password was not provided. Ensure that a non-null and non-empty password is specified for the Admin User Account to resolve this issue. |
| Provisioning of VM extension enablevmaccess has timed out. Extension provisioning has taken too long to complete. The extension did not report a message. | The error message indicates that the provisioning of the VM extension ‘enablevmaccess’ has timed out due to taking too long to complete. Additionally, the extension did not provide any status message during the process. To resolve this issue, consider checking the VM’s performance and network conditions, and retry the provisioning operation. For more information, see [Troubleshooting Azure Windows VM extension failures](/azure/virtual-machines/extensions/troubleshoot). |
| Error message: 'Cannot update Remote Desktop Connection settings for Administrator account. Error: User account username already exists but cannot be updated because it is not in the Administrators group. | The error indicates that the VM extension 'enablevmAccess' failed because the user account 'username' already exists but is not in the Administrators group. Ensure that the user account is added to the Administrators group to resolve this issue.|
| "internalErrorCode": "MultipleExtensionsPerHandlerNotAllowed", "code": "BadRequest","message": "Multiple VMExtensions per handler not supported for OS type 'Windows'. VMExtension 'enablevmaccess' with handler 'Microsoft.Compute.VMAccessAgent' already added or specified in input." | The error message indicates that multiple VM extensions per handler are not supported for the OS type ‘Windows’. The ‘enablevmaccess’ extension with the handler ‘Microsoft.Compute.VMAccessAgent’ has already been added or specified in the input. To resolve this issue, ensure that only one extension per handler is configured for the VM. <br><br> Manually remove extension and retry operation <br> ```Remove-AzVMExtension -ResourceGroupName "ResourceGroup11" -Name "ExtensionName" -VMName "VirtualMachineName"``` |

For more help, you can contact the Azure experts at [Azure Community Support](https://azure.microsoft.com/support/forums/). Alternatively, you can file an Azure support incident. Go to [Azure support](https://azure.microsoft.com/support/options/) and select **Get support**. For more information about Azure Support, read the [Azure support plans FAQ](https://azure.microsoft.com/support/faq/).

