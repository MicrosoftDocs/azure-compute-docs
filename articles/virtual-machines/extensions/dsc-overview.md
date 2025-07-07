---
title: Desired State Configuration for Azure overview
description: Learn how to use the Microsoft Azure extension handler for PowerShell Desired State Configuration (DSC), including prerequisites, architecture, and cmdlets.
services: virtual-machines
author: mgoedtel
keywords: 'dsc'
ms.assetid: bbacbc93-1e7b-4611-a3ec-e3320641f9ba
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: how-to
ms.tgt_pltfrm: vm-windows
ms.date: 03/28/2023
ms.author: magoedte
ms.custom: devx-track-azurecli, devx-track-arm-template
ms.devlang: azurecli
# Customer intent: "As an IT administrator managing Azure virtual machines, I want to implement Desired State Configuration using Azure extensions, so that I can automate and maintain consistent VM configurations efficiently."
---

# Introduction to the Azure Desired State Configuration extension handler

> [!NOTE]
> DSC extension will be retired on March 31, 2028. Please transition to
> [Azure Machine Configuration](/azure/governance/machine-configuration/overview) by that date.
> For more information, see the [blog post](https://azure.microsoft.com/updates/?id=485828)
> announcement. The Azure Machine Configuration service combines certain features of DSC Extension, Azure
> Automation State Configuration, and commonly requested features from customer feedback.
> Azure Machine Configuration also includes hybrid machine support through
> [Arc-enabled servers](/azure/azure-arc/servers/overview).

The Azure VM Extension for Azure virtual machines (VM) and the associated extensions are part of Microsoft Azure infrastructure services. Azure VM extensions are software components that extend VM functionality and simplify various VM management operations.

The DSC extension only pushes a configuration to the VM. No ongoing reporting is available, other than locally in the VM.

## Available DSC versions

The DSC extension supports configurations from version 1.1 of the DSC platform. For more information, see [PSDesiredStateConfiguration v1.1](/powershell/dsc/overview?view=dsc-1.1&preserve-view=true).

## Prerequisites

- **Developer workstation**: To interact with the Azure DSC extension, you must use either the Azure portal or the Azure PowerShell/CLI SDK.

- **Guest agent**: The Azure VM that's prepared by the DSC configuration must use an operating system that supports Windows Management Framework (WMF) 4.0 or later. For the full list of supported operating system versions, see the [Azure DSC extension version history](/azure/automation/automation-dsc-extension-history).

## Terms and concepts

This article assumes familiarity with the following concepts:

- **Configuration** refers to a DSC configuration document.

- **Node** identifies a target for a DSC configuration. In this article, *node* always refers to an Azure VM.

- **Configuration data** is stored in a PowerShell DSC format file (.psd1) that has environmental data for a configuration.

## Architecture

The Azure DSC extension uses the Azure VM Extension framework to deliver, enact, and report on DSC configurations running on Azure VMs. The DSC extension accepts a configuration document and a set of parameters.

When the extension is called the first time, it installs a version of WMF by using the following logic:

- If the Azure VM operating system is Windows Server 2016, no action is taken. Windows Server 2016 already has the latest version of PowerShell installed.

- If the `wmfVersion` property is specified, the specified version of WMF is installed, unless the specified version is incompatible with the operating system on the VM.

- If no `wmfVersion` property is specified, the latest applicable version of WMF is installed.

The WMF installation process requires a restart. After you restart, the extension downloads the .zip file that's specified in the `modulesUrl` property, if provided. If this location is in Azure Blob Storage, you can specify an SAS token in the `sasToken` property to access the file. After the .zip downloads and unpacks, the configuration function defined in `configurationFunction` runs to generate a [Managed Object Format (MOF)](/windows/win32/wmisdk/managed-object-format--mof-) file (.mof). The extension then runs the `Start-DscConfiguration -Force` command by using the generated .mof file. The extension captures output and writes it to the Azure status channel.

### Node configuration name

For the `NodeConfigurationName` parameter, be sure to provide the name of the node configuration and not the Configuration.

The Configuration is defined in a script that's used to compile the node configuration (MOF file). The name of the node configuration is always the name of the Configuration followed by a period `.` and either `localhost` or a specific computer name.

## ARM template deployment

The most common approach for deploying the DSC extension is to use Azure Resource Manager templates. For more information and for examples of how to include the DSC extension in ARM templates, see [Desired State Configuration extension with ARM templates](dsc-template.md).

## PowerShell cmdlet deployment

PowerShell cmdlets for managing the DSC extension are ideal for interactive troubleshooting and information-gathering scenarios. You can use the cmdlets to package, publish, and monitor DSC extension deployments.

Here are some of the PowerShell cmdlets that are available:

- The **Publish-AzVMDscConfiguration** cmdlet takes in a configuration file, scans it for dependent DSC resources, and then creates a .zip file. The .zip file contains the configuration and DSC resources that are needed to enact the configuration. The cmdlet can also create the package locally by using the `-OutputArchivePath` parameter. Otherwise, the cmdlet publishes the .zip file to Blob Storage, and then secures it with an SAS token.

   The PowerShell configuration script (.ps1) created by the cmdlet is in the .zip file at the root of the archive folder. The module folder is placed in the archive folder in resources.

- The **Set-AzVMDscExtension** cmdlet injects the settings that the PowerShell DSC extension requires into a VM configuration object.

- The **Get-AzVMDscExtension** cmdlet retrieves the DSC extension status of a specific VM.

- The **Get-AzVMDscExtensionStatus** cmdlet retrieves the status of the DSC configuration that's enacted by the DSC extension handler. This action can be performed on a single VM or a group of VMs.

- The **Remove-AzVMDscExtension** cmdlet removes the extension handler from a specific VM. Keep in mind that this cmdlet doesn't remove the configuration, uninstall WMF, or change the applied settings on the VM. The cmdlet only removes the extension handler.

### Important considerations

There are several considerations to keep in mind when working with Azure Resource Manager cmdlets.

- Azure Resource Manager cmdlets are synchronous.

- Several parameters are required, including `ResourceGroupName`, `VMName`, `ArchiveStorageAccountName`, `Version`, and `Location`.

- `ArchiveResourceGroupName` is an optional parameter. Specify this parameter when your storage account belongs to a different resource group than the one where the VM is created.

- Use the `AutoUpdate` switch to automatically update the extension handler to the latest version when it's available. This parameter has the potential to cause restarts on the VM when a new version of WMF is released.

### Configuration with PowerShell cmdlets

The Azure DSC extension can use DSC configuration documents to directly configure Azure VMs during deployment. This step doesn't register the node to Automation or Machine Configuration. Keep in mind that the node isn't centrally managed.

The following code shows a simple example configuration. To work with this example, save this configuration locally as the **iisInstall.ps1** script file.

```powershell
configuration IISInstall
{
    node "localhost"
    {
        WindowsFeature IIS
        {
            Ensure = "Present"
            Name = "Web-Server"
        }
    }
}
```

The following PowerShell commands place the iisInstall.ps1 script on the specified VM. The commands also execute the configuration, and then report back on status.

```powershell
$resourceGroup = 'dscVmDemo'
$vmName = 'myVM'
$storageName = 'demostorage'
#Publish the configuration script to user storage
Publish-AzVMDscConfiguration -ConfigurationPath .\iisInstall.ps1 -ResourceGroupName $resourceGroup -StorageAccountName $storageName -force
#Set the VM to run the DSC configuration
Set-AzVMDscExtension -Version '2.76' -ResourceGroupName $resourceGroup -VMName $vmName -ArchiveStorageAccountName $storageName -ArchiveBlobName 'iisInstall.ps1.zip' -AutoUpdate -ConfigurationName 'IISInstall'
```

## Azure CLI deployment

The Azure CLI can be used to deploy the DSC extension to an existing VM. The following examples show how to deploy a VM on Windows.

For a VM running Windows, use the following command:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name DSC \
  --publisher Microsoft.Powershell \
  --version 2.77 --protected-settings '{}' \
  --settings '{}'
```

## Azure portal deployment

To set up the DSC extension in the Azure portal, follow these steps:

1. Go to a VM.

1. Under **Settings**, select **Extensions + Applications**.

1. Under **Extensions**, select **+ Add**.

1. Select **PowerShell Desired State Configuration**, then select **Next**.

1. Configure the following parameters for the DSC extension.

   - **Configuration Modules or Script**: (Required) Provide the Configuration modules or script file for your VM.
   
      Configuration modules and scripts require a .ps1 file that has a configuration script or a .zip file with a .ps1 configuration script at the root. If you use a .zip file, all dependent resources must be included in module folders in the .zip file. You can create the .zip file by using the **Publish-AzureVMDscConfiguration -OutputArchivePath** cmdlet that's included in the Azure PowerShell SDK. The .zip file is uploaded to your user Blob Storage and secured by an SAS token.

   - **Module-qualified Name of Configuration**: (Required) Specify this setting to include multiple configuration functions in a single .ps1 script file. For this setting, enter the name of the configuration .ps1 script file followed by a slash `\` and then the name of the configuration function. For example, if the .ps1 script file has the name **configuration.ps1** and the configuration name is **IisInstall**, enter the value `configuration.ps1\IisInstall` for the setting.

   - **Configuration Arguments**: If the configuration function takes arguments, enter the values by using the format `argumentName1=value1,argumentName2=value2`. Notice that this format differs from the format that's used to specify configuration arguments in PowerShell cmdlets or ARM templates.

   - **Configuration Data PSD1 File**: If your configuration requires a configuration data file in .psd1 format, use this setting to select the data file and upload it to your user Blob Storage. The configuration data file is secured with an SAS token in Blob Storage.

   - **WMF Version**: Specify the version of Windows Management Framework to install on your VM. If you choose **latest**, which is the default value, the system installs the most recent version of WMF. Other possible values include 4.0, 5.0, and 5.1. The possible values are subject to updates.

   - **Data Collection**: Enable this setting if you want the DSC extension to collect telemetry about your VM. For more information, see [Azure DSC extension data collection](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/).

   - **Version**: (Required) Specify the version of the DSC extension to install. For information about versions, see [Azure DSC extension version history](/azure/automation/automation-dsc-extension-history).

   - **Auto Upgrade Minor Version**: This setting maps to the `AutoUpdate` switch in the cmdlets. Configure this setting to enable the DSC extension to automatically update to the latest version during installation. **Yes** instructs the DSC extension handler to use the latest available version. **No** (default) forces installation of the version you specify in the **Version** setting.

1. After you configure the parameters, select **Review + Create**, and then select **Create**.

## DSC extension logs

You can view logs for the Azure DSC extension on the VM under `C:\WindowsAzure\Logs\Plugins\Microsoft.Powershell.DSC\<version number>`.

## Next steps

- For more information about PowerShell DSC, go to the [PowerShell documentation center](/powershell/dsc/overview).
- Examine the [ARM template for the Azure DSC extension](dsc-template.md).
- For more functionality that you can manage by using PowerShell DSC, and for more DSC resources, browse the [PowerShell gallery](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).
- For details about passing sensitive parameters into configurations, see [Manage credentials securely with the Azure DSC extension handler](dsc-credentials.md).
