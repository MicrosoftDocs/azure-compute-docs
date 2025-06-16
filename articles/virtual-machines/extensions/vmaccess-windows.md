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
    "settings": {},
    "protectedSettings": {
      "username": "<username>",
      "password": "<password>",
      "reset_password": true
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
| reset_password | boolean | Whether or not to reset the user password. |

## Deploy with PowerShell

You can use PowerShell to apply the VMAccess extension to a Windows VM. The following PowerShell is an example of a script to reset the administrator password:

```powershell
$resourceGroup = "<ResourceGroupName>"
$vmName = "<VMName>"
$location = "<Location>"
$username = "<Username>"
$password = "<NewPassword>"

$settings = @{}
$protectedSettings = @{
    "username" = $username
    "password" = $password
    "reset_password" = $true
}

Set-AzVMExtension -ResourceGroupName $resourceGroup `
                    -VMName $vmName `
                    -Location $location `
                    -Name "VMAccessAgent" `
                    -Publisher "Microsoft.Compute" `
                    -ExtensionType "VMAccessAgent" `
                    -TypeHandlerVersion "2.4" `
                    -Settings $settings `
                    -ProtectedSettings $protectedSettings
```

## Template deployment

Azure VM Extensions can be deployed with Azure Resource Manager (ARM) templates. The JSON schema detailed in the previous section can be used in an ARM template to run the VMAccess Extension during the template's deployment. You can find a sample template that includes the VMAccess extension on [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute).

The JSON configuration for a virtual machine extension must be nested inside the virtual machine resource fragment of the template, specifically `"resources": []` object for the virtual machine template and for a virtual machine scale set under `"virtualMachineProfile":"extensionProfile":{"extensions" :[]` object.

### Use Azure PowerShell VM/VMSS extension commands for Windows

You can use the [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) command to run the VMAccess Extension with the specified configuration for a Windows virtual machine. 

The following PowerShell is an example using the Set-AzVMExtension command:

```powershell
Set-AzVMExtension `
  -ResourceGroupName "myResourceGroup" `
  -VMName "myVM" `
  -Name "VMAccessAgent" `
  -Publisher "Microsoft.Compute" `
  -Type "VMAccessAgent" `
  -TypeHandlerVersion "2.0" `
  -Settings @{ "ResetPassword" = $true } `
  -ProtectedSettings @{ "Username" = "adminUser"; "Password" = "userPassword" }
```

The -Settings and -ProtectedSettings parameters also accept JSON file paths. For example, to update the local administrator password using a JSON file, create a file named update_admin_password.json with the following content. Replace the values with your own information:

```json
{
  "Username": "adminUser",
  "Password": "userPassword"
}
```

Then, run the extension with the JSON file as input:

```powershell
Set-AzVMExtension `
  -ResourceGroupName "myResourceGroup" `
  -VMName "myVM" `
  -Name "VMAccessAgent" `
  -Publisher "Microsoft.Compute" `
  -Type "VMAccessAgent" `
  -TypeHandlerVersion "2.0" `
  -ProtectedSettings (Get-Content -Path "update_admin_password.json" -Raw | ConvertFrom-Json)
```

## Azure PowerShell deployment for Windows

Azure PowerShell can be used to deploy the VMAccess Extension to an existing Windows virtual machine or virtual machine scale set. You can deploy the extension to a VM by running:

```powershell
$username = "<username>"
$password = "<password>"
$settings = @{ "ResetPassword" = $true }
$protectedSettings = @{ "Username" = $username; "Password" = $password }
Set-AzVMExtension -ResourceGroupName "<resource-group>" `
    -VMName "<vm-name>" `
    -Location "<location>" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "VMAccessAgent" `
    -Name "VMAccessAgent" `
    -TypeHandlerVersion "2.0" `
    -Settings $settings `
    -ProtectedSettings $protectedSettings
```

You can also provide and modify extension settings by using strings:

```powershell
$username = "<username>"
$password = "<password>"
$settingsString = '{"ResetPassword": true}';
$protectedSettingsString = '{"Username":"' + $username + '", "Password":"' + $password + '"}';
Set-AzVMExtension -ResourceGroupName "<resource-group>" `
    -VMName "<vm-name>" `
    -Location "<location>" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "VMAccessAgent" `
    -Name "VMAccessAgent" `
    -TypeHandlerVersion "2.0" `
    -SettingString $settingsString `
    -ProtectedSettingString $protectedSettingsString
```

To deploy to a virtual machine scale set, run the following command:

```powershell
$resourceGroupName = "<resource-group>"
$vmssName = "<vmss-name>"
$protectedSettings = @{
  "Username" = "adminUser"
  "Password" = "userPassword"
}
$publicSettings = @{
  "ResetPassword" = $true
}
$vmss = Get-AzVmss `
            -ResourceGroupName $resourceGroupName `
            -VMScaleSetName $vmssName
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
    -Name "VMAccessAgent" `
    -Publisher "Microsoft.Compute" `
    -Type "VMAccessAgent" `
    -TypeHandlerVersion "2.0" `
    -AutoUpgradeMinorVersion $true `
    -Setting $publicSettings `
    -ProtectedSetting $protectedSettings
Update-AzVmss `
    -ResourceGroupName $resourceGroupName `
    -Name $vmssName `
    -VirtualMachineScaleSet $vmss
```

## Troubleshoot and support

The VMAccess extension logs exist locally on the VM and are particularly useful for troubleshooting.

| Location | Description |
| --- | --- |
| `C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.VMAccessAgent\` | Contains logs from the Windows Agent that show when an update to the extension occurred. Verify these logs to ensure the extension ran successfully. |
| `C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.VMAccessAgent\<version>\` | The VMAccess Extension produces detailed logs in this directory. It includes `CommandExecution.log`, which records each command executed along with its result, and `extension.log`, which contains individual execution logs. |
| `C:\WindowsAzure\Logs\WaAppAgent\` | Contains configuration details and binary logs related to the VMAccess extension. |

You can also retrieve the execution state of the VMAccess Extension, along with other extensions on a given VM, by running the following command:

You can retrieve the execution state of the VM extensions on a given VM by running the following PowerShell command:

```powershell
Get-AzVMExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" | Format-Table
```

### Error messages

| Error  | Description |
| ---- | ---- |
| {"innererror": {"internalErrorCode": "CannotModifyExtensionsWhenVMNotRunning"}, "code": "OperationNotAllowed","message": "Cannot modify extensions in the VM when the VM is not running."} | The error indicates that the operation to modify extensions in the VM is not allowed because the VM is not running. Ensure that the VM is in a running state before attempting to modify extensions. |
| VM has reported a failure when processing extension 'enablevmAccess' (publisher 'Microsoft.Compute' and type 'VMAccessAgent'). Error message: 'VMAccess Extension does not support Domain Controller.'. More information on troubleshooting is available at https://aka.ms/vmextensionwindowstroubleshoot . | The error indicates that the VM extension 'enablevmAccess' failed because it does not support Domain Controller. Ensure that the VM is not configured as a Domain Controller when using this extension. For more information, see [Reset Remote Desktop Services or its administrator password in a Windows VM](/troubleshoot/azure/virtual-machines/windows/reset-rdp). |
| VM 'vmname' has not reported status for VM agent or extensions. Verify that the OS is up and healthy, the VM has a running VM agent, and that it can establish outbound connections to Azure storage. Please refer to https://aka.ms/vmextensionwindowstroubleshoot  for additional VM agent troubleshooting information. | See [Troubleshooting checklist](/troubleshoot/azure/virtual-machines/windows/windows-azure-guest-agent?source=recommendations#troubleshooting-checklist). |
| VM has reported a failure when processing extension 'enablevmAccess' (publisher 'Microsoft.Compute' and type 'VMAccessAgent'). Error message: 'Cannot update Remote Desktop Connection settings for Administrator account. Error: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.Runtime.InteropServices.COMException: The password does not meet the password policy requirements. Check the minimum password length, password complexity, and password history requirements. --- End of inner exception stack trace --- at System.DirectoryServices.DirectoryEntry.Invoke(String methodName, Object[] args) at Microsoft.WindowsAzure.GuestAgent.Plugins.WindowsUser.SetPassword(SecureString password, Boolean expirePassword) at Microsoft.WindowsAzure.GuestAgent.Plugins.RemoteAccessAccountManager.AddOrUpdateRemoteUserAccount(String userName, SecureString password, Boolean expirePassword) at Microsoft.WindowsAzure.GuestAgent.Plugins.JsonExtensions.VMAccess.VMAccessExtension.OnEnable()'. More information on troubleshooting is available at https://aka.ms/vmextensionwindowstroubleshoot. | The error indicates that the VM extension 'enablevmAccess' failed to update Remote Desktop Connection settings for the Administrator account due to a password policy violation. Ensure that the password meets Windows password policy requirements, including minimum length, complexity, and history. For more information, see [Troubleshoot VM extensions](/azure/virtual-machines/extensions/features-windows#troubleshoot-vm-extensions). |
| VM has reported a failure when processing extension 'enablevmAccess' (publisher 'Microsoft.Compute' and type 'VMAccessAgent'). Error message: 'The Admin User Account password cannot be null or empty if provided the username.'. More information on troubleshooting is available at https://aka.ms/vmextensionwindowstroubleshoot . | The error indicates that the VM extension 'enablevmAccess' failed because the Admin User Account password was not provided. Ensure that a non-null and non-empty password is specified for the Admin User Account to resolve this issue. |
|   Provisioning of VM extension enablevmaccess has timed out. Extension provisioning has taken too long to complete. The extension did not report a message. | The error message indicates that the provisioning of the VM extension ‘enablevmaccess’ has timed out due to taking too long to complete. Additionally, the extension did not provide any status message during the process. To resolve this issue, consider checking the VM’s performance and network conditions, and retry the provisioning operation. For more information, see [Troubleshooting Azure Windows VM extension failures](/azure/virtual-machines/extensions/troubleshoot). |
| VM has reported a failure when processing extension 'enablevmAccess' (publisher 'Microsoft.Compute' and type 'VMAccessAgent'). Error message: 'Cannot update Remote Desktop Connection settings for Administrator account. Error: System.Exception: User account scsadmin already exists but cannot be updated because it is not in the Administrators group. at Microsoft.WindowsAzure.GuestAgent.Plugins.RemoteAccessAccountManager.AddOrUpdateRemoteUserAccount(String userName, SecureString password, Boolean expirePassword) at Microsoft.WindowsAzure.GuestAgent.Plugins.JsonExtensions.VMAccess.VMAccessExtension.OnEnable()'. More information on troubleshooting is available at https://aka.ms/vmextensionwindowstroubleshoot . | The error indicates that the VM extension 'enablevmAccess' failed because the user account 'scsadmin' already exists but is not in the Administrators group. Ensure that the user account is added to the Administrators group to resolve this issue.|
| VM has reported a failure when processing extension 'enablevmaccess' (publisher 'Microsoft.Compute' and type 'VMAccessAgent'). Error message: 'Cannot update Remote Desktop Connection settings for Administrator account. Error: System.Runtime.InteropServices.COMException (0x800708C5): The password does not meet the password policy requirements. Check the minimum password length, password complexity, and password history requirements. at System.DirectoryServices.DirectoryEntry.CommitChanges() at Microsoft.WindowsAzure.GuestAgent.Plugins.WindowsUser.SetPassword(SecureString password, Boolean expirePassword) at Microsoft.WindowsAzure.GuestAgent.Plugins.WindowsUserManager.CreateUserInGroup(String userName, SecureString password, Boolean expirePassword, String[] groups) at Microsoft.WindowsAzure.GuestAgent.Plugins.RemoteAccessAccountManager.AddOrUpdateRemoteUserAccount(String userName, SecureString password, Boolean expirePassword) at Microsoft.WindowsAzure.GuestAgent.Plugins.JsonExtensions.VMAccess.VMAccessExtension.OnEnable()'. More information on troubleshooting is available at https://aka.ms/vmextensionwindowstroubleshoot . | The error message indicates that the VM failed to process the ‘enablevmaccess’ extension due to an issue with updating Remote Desktop Connection settings for the Administrator account. The specific error is related to the password not meeting the policy requirements, such as minimum length, complexity, and history. To resolve this issue, ensure that the password complies with the required policy standards. For more information, see [Troubleshoot VM extensions](/azure/virtual-machines/extensions/features-windows#troubleshoot-vm-extensions). |
| {"innererror": {"internalErrorCode": "MultipleExtensionsPerHandlerNotAllowed"}, "code": "BadRequest","message": "Multiple VMExtensions per handler not supported for OS type 'Windows'. VMExtension 'enablevmaccess' with handler 'Microsoft.Compute.VMAccessAgent' already added or specified in input."} | The error message indicates that multiple VM extensions per handler are not supported for the OS type ‘Windows’. The ‘enablevmaccess’ extension with the handler ‘Microsoft.Compute.VMAccessAgent’ has already been added or specified in the input. To resolve this issue, ensure that only one extension per handler is configured for the VM. <br><br> Manually remove extension and retry operation <br> ```Remove-AzVMExtension -ResourceGroupName "ResourceGroup11" -Name "ExtensionName" -VMName "VirtualMachineName"``` |

For more help, you can contact the Azure experts at [Azure Community Support](https://azure.microsoft.com/support/forums/). Alternatively, you can file an Azure support incident. Go to [Azure support](https://azure.microsoft.com/support/options/) and select **Get support**. For more information about Azure Support, read the [Azure support plans FAQ](https://azure.microsoft.com/support/faq/).

