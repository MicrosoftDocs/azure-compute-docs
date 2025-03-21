---
title: Reset access to an Azure Windows VM 
description: Learn how to manage administrative users and reset access on Windows VMs by using the VMAccess extension and the Azure PowerShell.
ms.topic: conceptual
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

Azure VM Extensions can be deployed with Azure Resource Manager (ARM) templates. The JSON schema detailed in the previous section can be used in an ARM template to run the VMAccess Extension during the template's deployment. You can find a sample template that includes the VMAccess extension on [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-password-reset).

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

For more help, you can contact the Azure experts at [Azure Community Support](https://azure.microsoft.com/support/forums/). Alternatively, you can file an Azure support incident. Go to [Azure support](https://azure.microsoft.com/support/options/) and select **Get support**. For more information about Azure Support, read the [Azure support plans FAQ](https://azure.microsoft.com/support/faq/).

