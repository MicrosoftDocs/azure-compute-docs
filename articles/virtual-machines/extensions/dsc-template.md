---
title: Desired State Configuration extension with Azure Resource Manager templates
description: Learn about the Resource Manager template definition for the Desired State Configuration (DSC) extension in Azure.
services: virtual-machines
author: mgoedtel
ms.custom: devx-track-arm-template
keywords: 'dsc'
ms.assetid: b5402e5a-1768-4075-8c19-b7f7402687af
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: concept-article
ms.tgt_pltfrm: vm-windows
ms.date: 03/13/2022
ms.author: magoedte
# Customer intent: As a cloud administrator, I want to deploy the Desired State Configuration extension using Resource Manager templates so that I can manage the configuration of virtual machines effectively before transitioning to Azure Machine Configuration.
---
# Desired State Configuration extension with Azure Resource Manager templates

> [!NOTE]
> DSC extension will be retired on March 31, 2028. Please transition to
> [Azure Machine Configuration](/azure/governance/machine-configuration/overview) by that date.
> For more information, see the [blog post](https://azure.microsoft.com/updates/?id=485828)
> announcement. The Azure Machine Configuration service combines certain features of DSC Extension, Azure
> Automation State Configuration, and commonly requested features from customer feedback.
> Azure Machine Configuration also includes hybrid machine support through
> [Arc-enabled servers](/azure/azure-arc/servers/overview).

This article describes the Azure Resource Manager template for the [Desired State Configuration
(DSC) extension handler](dsc-overview.md).

> [!NOTE]
> You might encounter slightly different schema examples. The change in schema occurred in the October 2016 release. For details, see [Update from a previous format](#update-from-a-previous-format).

## Bicep example for a VM

The DSC extension inherits default extension properties.
For more information, see
[VirtualMachineExtension class](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension).

```bicep
@description('URI of the configuration package')
param configUri string

@description('first configuration parameter')
param arg1 string

@description('second configuration parameter')
@secure()
param arg2 string

var configName = split(split(configUri, '/')[4], '.')[0]

resource vm 'Microsoft.Compute/virtualMachines@2023-09-01' existing = {
  name: srvName
}

resource configuration_dscext 'Microsoft.Compute/virtualMachines/extensions@2024-07-01' = {
  name: 'configurationname_dscext'
  parent: vm
  location: location
  properties: {
    publisher: 'Microsoft.Powershell'
    type: 'DSC'
    typeHandlerVersion: '2.77'
    autoUpgradeMinorVersion: true
    settings: {
      wmfVersion: 'latest'
      configuration: {
        url: configUri
        script: 'configurationname_dscext.ps1'
        function: 'configurationname_dscext'
      }
      configurationArguments: {
        arg1: arg1
      }
      advancedOptions: {
        rebootNodeIfNeeded: true
      }
    }
    protectedSettings: {
      configurationArguments: {
        arg2: arg2
      }
    }
  }
}
```

## Bicep example for Windows virtual machine scale sets

A virtual machine scale set node has a section that includes an **extensionProfile** attribute.
Under **extensions**, add the details for DSC Extension.

For the latest details about authoring templates that deploy extensions for Virtual Machine Scale Sets,
see the document [Microsoft.Compute virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets?pivots=deployment-language-bicep)

The DSC extension inherits default extension properties.
For more information,
see [VirtualMachineScaleSetExtension class](/dotnet/api/microsoft.azure.management.compute.models.virtualmachinescalesetextension).

## Settings vs. protectedSettings

All settings are saved in a settings text file on the VM.
Properties listed under **settings** are public properties.
Public properties aren't encrypted in the settings text file.
Properties listed under **protectedSettings** are encrypted with a certificate
and are not shown in plain text in the settings file on the VM.

If the configuration needs credentials,
you can include the credentials in **protectedSettings**:

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
               "userName": "UsernameValue1",
               "password": "PasswordValue1"
        }
    }
}
```

## Example using the configuration script in Azure Storage

The following example is from the
[DSC extension handler overview](dsc-overview.md).
This example uses Resource Manager templates
instead of cmdlets to deploy the extension.
Save the IisInstall.ps1 configuration,
place it in a .zip file (example: `iisinstall.zip`),
and then upload the file in an accessible URL.
This example uses Azure Blob storage,
but you can download .zip files from any arbitrary location.

In the Resource Manager template,
the following code instructs the VM to download the correct file,
and then run the appropriate PowerShell function:

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/iisinstall.zip",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## Update from a previous format

Any settings in a previous format of the extension
(and which have the public properties **ModulesUrl**, **ModuleSource**,
**ModuleVersion**, **ConfigurationFunction**, **SasToken**, or **Properties**)
automatically adapt to the current format of the extension.
They run just as they did before.

The following schema shows what the previous settings schema looked like:

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties": {
        "ParameterToConfigurationFunction1": "Value1",
        "ParameterToConfigurationFunction2": "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1"
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": {
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

Here's how the previous format adapts to the current format:

| Current Property name | Previous schema equivalent |
| --- | --- |
| settings.wmfVersion |settings.WMFVersion |
| settings.configuration.url |settings.ModulesUrl |
| settings.configuration.script |First part of settings.ConfigurationFunction (before \\\\) |
| settings.configuration.function |Second part of settings.ConfigurationFunction (after \\\\) |
| settings.configuration.module.name | settings.ModuleSource |
| settings.configuration.module.version | settings.ModuleVersion |
| settings.configurationArguments |settings.Properties |
| settings.configurationData.url |protectedSettings.DataBlobUri (without SAS token) |
| settings.privacy.dataCollection |settings.Privacy.dataCollection |
| settings.advancedOptions.downloadMappings |settings.AdvancedOptions.DownloadMappings |
| protectedSettings.configurationArguments |protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken |settings.SasToken |
| protectedSettings.configurationDataUrlSasToken |SAS token from protectedSettings.DataBlobUri |

## Troubleshooting

Here are some of the errors you might run into and how you can fix them.

### Invalid values

"Privacy.dataCollection is '{0}'.
The only possible values are '', 'Enable', and 'Disable'".
"WmfVersion is '{0}'.
Only possible values are … and 'latest'".

**Problem**: A provided value isn't allowed.

**Solution**: Change the invalid value to a valid value.

### Invalid URL

"ConfigurationData.url is '{0}'. This is not a valid URL"
"DataBlobUri is '{0}'. This is not a valid URL"
"Configuration.url is '{0}'. This is not a valid URL"

**Problem**: A provided URL isn't valid.

**Solution**: Check all your provided URLs.
Ensure that all URLs resolve to valid locations
that the extension can access on the remote machine.

### Invalid ConfigurationArgument type

"Invalid configurationArguments type {0}"

**Problem**: The *ConfigurationArguments* property
can't resolve to a **Hash table** object.

**Solution**: Make your *ConfigurationArguments* property a **Hash table**.
Follow the format provided in the preceding examples. Watch for quotes,
commas, and braces.

### Duplicate ConfigurationArguments

"Found duplicate arguments '{0}' in both public
and protected configurationArguments"

**Problem**: The *ConfigurationArguments* in public settings
and the *ConfigurationArguments* in protected settings
have properties with the same name.

**Solution**: Remove one of the duplicate properties.

### Missing properties

"settings.Configuration.function requires that settings.configuration.url
or settings.configuration.module is specified"

"settings.Configuration.url requires that settings.configuration.script is specified"

"settings.Configuration.script requires that settings.configuration.url is specified"

"settings.Configuration.url requires that settings.configuration.function is specified"

"protectedSettings.ConfigurationUrlSasToken requires that settings.configuration.url is specified"

"protectedSettings.ConfigurationDataUrlSasToken requires that settings.configurationData.url is specified"

**Problem**: A defined property needs another property, which is missing.

**Solutions**:

- Provide the missing property.
- Remove the property that needs the missing property.

## Next steps

- Learn about [using virtual machine scale sets with the Azure DSC extension](../../virtual-machine-scale-sets/virtual-machine-scale-sets-dsc.md).
- Find more details about [DSC's secure credential management](dsc-credentials.md).
- Get an [introduction to the Azure DSC extension handler](dsc-overview.md).
- For more information about PowerShell DSC, go to the [PowerShell documentation center](/powershell/dsc/overview).
