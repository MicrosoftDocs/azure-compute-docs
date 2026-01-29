---
title: Azure Key Vault VM Extension for Linux
description: Deploy an agent performing automatic refresh of Key Vault certificates on virtual machines using a virtual machine extension.
services: virtual-machines
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: linux
ms.topic: concept-article
ms.date: 04/29/2025
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell, devx-track-azurecli, linux-related-content
# Customer intent: As a system administrator managing Linux virtual machines, I want to deploy the Key Vault VM extension so that I can automate the refresh of certificates stored in Azure Key Vault and ensure seamless certificate management.
---
# Key Vault virtual machine extension for Linux

The Key Vault VM extension provides automatic refresh of certificates stored in an Azure key vault. Specifically, the extension monitors a list of observed certificates stored in key vaults.  The extension retrieves and installs the corresponding certificates after detecting a change. This document details the supported platforms, configurations, and deployment options for the Key Vault VM extension for Linux.

## Operating system

The Key Vault VM extension supports:

- Ubuntu 22.04 and later
- [Azure Linux](https://github.com/microsoft/azurelinux)

### Supported certificate content types

- PKCS #12
- PEM

## Features

Key Vault VM extension for Linux version 3.0+ supports:

- Add ACL permissions for downloaded certificates to provide read access for users and groups
- Certificate installation location configuration
- Custom symbolic name support
- VM extension logging integration support through [Fluentd](https://www.fluentd.org/)

## Prerequisites
  - Key Vault instance with certificate. See [Create a Key Vault](/azure/key-vault/general/quick-create-portal)
  - Assigned [managed identity](/azure/active-directory/managed-identities-azure-resources/overview) on virtual machines/virtual machine scale sets.
  - The **Key Vault Secrets User** role at the Key Vault scope level for VMs and Azure Virtual Machine Scale Sets managed identity. This role retrieves a secret's portion of a certificate. For more information, see the following articles:
    - [Authentication in Azure Key Vault](/azure/key-vault/general/authentication)
    - [Use Azure RBAC secret, key, and certificate permissions with Azure Key Vault](/azure/key-vault/general/rbac-guide#using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault)
    - [Key Vault scope role assignment](/azure/key-vault/general/rbac-guide?tabs=azure-cli#key-vault-scope-role-assignment)
  -  Virtual machine scale sets should have the following identity setting:
  `
  "identity": {
  "type": "UserAssigned",
  "userAssignedIdentities": {
  "[parameters('userAssignedIdentityResourceId')]": {}
  }
  }
  `

 - AKV extension should have this setting:
  `
  "authenticationSettings": {
    "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
    "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
  }
   `
   
## Upgrading Key Vault VM extension

* If you need to upgrade from an older version to version 3.0+, you would need to delete previous version first, then install version 3.0.
```azurecli
  az vm extension delete --name KeyVaultForLinux --resource-group ${resourceGroup} --vm-name ${vmName}
  az vm extension set -n "KeyVaultForLinux" --publisher Microsoft.Azure.KeyVault --resource-group "${resourceGroup}" --vm-name "${vmName}" –settings .\akvvm.json –version 3.0
```

* If the VM has certificates downloaded by previous version, deleting VM extension doesn't delete the downloaded certificates. After installing the newer version, the existing certificates aren't modified. You would need to delete the certificate files or roll-over the certificate to get the PEM file with full-chain on the VM.

## Extension schema
The following JSON provides the schema for the Key Vault VM extension. All settings are specified as plain (unprotected) settings, as none are considered sensitive. To configure the extension, you must specify a list of certificates to monitor, how often to poll for updates, and the destination path for storing certificates. Specifically:

```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KVVMExtensionForLinux",
      "apiVersion": "2022-11-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "3.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
      "loggingSettings": <Optional logging settings, e.g.:
        {
              "logger": <Logger engine name. e.g.: "fluentd">,
              "endpoint": <Logger listening endpoint "tcp://localhost:24224">,
              "format": <Logging format. e.g.: "forward">,
              "servicename": <Service name used in logs. e.g.: "akvvm_service">
          }>,
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "linkOnRenewal": <Not available on Linux e.g.: false>,
          "requireInitialSync": <initial synchronization of certificates e..g: true>,
          "aclEnabled": <Enables ACLs for downloaded certificates, e.g.: true>,
          "observedCertificates": <An array of KeyVault URIs that represent monitored certificates, including certificate store location, ACL permission to certificate private key, and custom symbolic name. e.g.: 
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. e.g.: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreLocation": <disk path where certificate is stored, e.g.: "/var/lib/waagent/Microsoft.Azure.KeyVault/app1">,
                    "customSymbolicLinkName": <symbolic name for the certificate. e.g.: "app1Cert1">,
                    "acls": [
                        {
                            "user": "app1",
                            "group": "appGroup1"
                        },
                        {
                            "user": "service1"
                        }
                    ]
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate2">,
                    "certificateStoreLocation": <disk path where the certificate is stored, e.g.: "/var/lib/waagent/Microsoft.Azure.KeyVault/app2">,
                    "acls": [
                        {
                            "user": "app2",
                        }
                    ]
                }
             ]>
        },
        "authenticationSettings": <Optional msi settings, e.g.:
        {
          "msiEndpoint":  <Required when msiClientId is provided. MSI endpoint e.g. for most Azure VMs: "http://169.254.169.254/metadata/identity">,
          "msiClientId":  <Required when VM has any user assigned identities. MSI identity e.g.: "00001111-aaaa-2222-bbbb-3333cccc4444".>
        }>
       }
      }
    }
```

> [!NOTE]
> Your observed certificates URLs should be of the form `https://myVaultName.vault.azure.net/secrets/myCertName`.
>
> This is because the `/secrets` path returns the full certificate, including the private key, while the `/certificates` path doesn't. More information about certificates can be found here: [Key Vault Certificates](/azure/key-vault/general/about-keys-secrets-certificates)

> [!IMPORTANT]
> The `authenticationSettings` property is required only when your VM or VM scale set uses **user-assigned managed identities** or when using **Azure Arc-enabled VMs**. For **system-assigned managed identities**, omit the `authenticationSettings` section. Including it will cause deployment to fail. Without this section, a VM with user assigned identities will result in the Key Vault extension failing and being unable to download certificates. 
>Set `msiClientId` to the identity that will authenticate to Key Vault.
>
> Also **required** for **Azure Arc-enabled VMs**.
> Set msiEndpoint to `http://localhost:40342/metadata/identity`.

### Property values

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |
| `apiVersion` | 2022-07-01 | date |
| `publisher` | Microsoft.Azure.KeyVault | string |
| `type` | KeyVaultForLinux | string |
| `typeHandlerVersion` | 3.0 | int |
| `pollingIntervalInS` | 3600 | string |
| `certificateStoreName` | It's ignored on Linux | string |
| `linkOnRenewal` | false | boolean |
| `requireInitialSync` | true | boolean |
| `aclEnabled` | true | boolean |
| `certificateStoreLocation`  | /var/lib/waagent/Microsoft.Azure.KeyVault.Store | string |
| `observedCertificates`  | [{...}, {...}] | string array |
| `observedCertificates/url`  | "https://myvault.vault.azure.net/secrets/mycertificate1" | string |
| `observedCertificates/certificateStoreLocation` | "/var/lib/waagent/Microsoft.Azure.KeyVault/app1" | string |
| `observedCertificates/customSymbolicLinkName` (optional) | "app1Cert1" | string |
| `observedCertificates/acls` (optional) | "{...}, {...}" | string array |
| `authenticationSettings` (optional) | {...} | object |
| `authenticationSettings/msiEndpoint` | http://169.254.169.254/metadata/identity | string |
| `authenticationSettings/msiClientId` | 00001111-aaaa-2222-bbbb-3333cccc4444 | string |
| `loggingSettings` (optional) | {...} | object |
| `loggingSettings/logger` | "fluentd" | string |
| `loggingSettings/endpoint` | "tcp://localhost:24224" | string |
| `loggingSettings/format` | "forward" | string |
| `loggingSettings/servicename` | "akvvm_service" | string |

## Template deployment

Azure VM extensions can be deployed with Azure Resource Manager templates. Templates are ideal when deploying one or more virtual machines that require post deployment refresh of certificates. The extension can be deployed to individual VMs or virtual machine scale sets. The schema and configuration are common to both template types.

 > [!NOTE]
> The VM extension would require system or user managed identity to be assigned to authenticate to Key vault.  See [How to authenticate to Key Vault and assign a Key Vault access policy.](/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm)
>

```json
   {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KeyVaultForLinux",
      "apiVersion": "2022-11-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "3.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
          "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "requireInitialSync": <initial synchronization of certificates e..g: false>,
          "aclEnabled": <enables/disables acls on defined certificates e.g.: true>,
          "observedCertificates": <An array of KeyVault URIs that represent monitored certificates, including certificate store location and ACL permission to certificate private key. Example:
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. Example: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault.Store">,
                    "acls": <Optional. An array of preferred acls with read access to certificate private keys. Example: 
                    [
                        {
                            "user": "app1",
                            "group": "appGroup1"
                        },
                        {
                            "user": "service1"
                        }
                    ]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate2">,
                    "certificateStoreName": <ignored on linux>,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault.Store">,
                    "acls": <Optional. An array of preferred acls with read access to certificate private keys. Example: 
                    [
                        {
                            "user": "app2"
                        }
                    ]>
                }
               
             ]>   
          },
          "authenticationSettings": {
              "msiEndpoint":  <Required when msiClientId is provided. MSI endpoint e.g. for most Azure VMs: "http://169.254.169.254/metadata/identity">,
              "msiClientId":  <Required when VM has any user assigned identities. MSI identity e.g.: "00001111-aaaa-2222-bbbb-3333cccc4444">
          }
        } 
      }
    }
```

### Extension Dependency Ordering

The Key Vault VM extension supports extension ordering if configured. By default the extension reports successful start as soon as polling starts. However, you can configure it to wait until it successfully downloads the complete list of certificates before reporting a successful start. If other extensions depend on installed certificates before they start, then enabling this setting will allow those extensions to declare a dependency on the Key Vault extension. This will prevent those extensions from starting until all certificates they depend on have been installed. 

The extension will retry the initial download up to 25 times with increasing backoff periods, during which it remains in a `Transitioning` state. If the retries are exhausted, the extension will report an `Error` state.

To turn on extension dependency, set the following:
```
"secretsManagementSettings": {
    "requireInitialSync": true,
    ...
}
```
> [!NOTE]
> Using this feature is not compatible with an ARM template that creates a system assigned identity and updates a Key Vault access policy with that identity. Doing so will result in a deadlock as the vault access policy cannot be updated until all extensions have started. You should instead use a *single user assigned MSI identity* and pre-ACL your vaults with that identity before deploying.

## Azure PowerShell deployment
> [!WARNING]
> PowerShell clients often add `\` to `"` in the settings.json which will cause akvvm_service fails with error: `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`

The Azure PowerShell can be used to deploy the Key Vault VM extension to an existing virtual machine or virtual machine scale set.

* To deploy the extension on a VM:

The Azure Key Vault VM extension can be deployed with Azure PowerShell. Save Key Vault VM extension settings to a JSON file (settings.json). 

The following JSON snippets provide example settings for deploying the Key Vault VM extension with PowerShell.
    
```json
{
   "secretsManagementSettings": {
   "pollingIntervalInS": "3600",
   "linkOnRenewal": true,
   "aclEnabled": true,
   "observedCertificates":
   [
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate1",
          "certificateStoreLocation":  "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
          "acls": 
          [
              {
                  "user": "app1",
                  "group": "appGroup1"
              },
              {
                  "user": "service1"
              }
          ]
      },
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate2",
          "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
          "acls": 
          [
              {
                  "user": "app2"
              }
          ]
      }
   ]},
   "authenticationSettings": {
      "msiEndpoint":  "http://169.254.169.254/metadata/identity/oauth2/token",
      "msiClientId":  "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxx"
   }      
}
```

* To deploy the extension on a virtual machine:

```powershell
# Build settings
$settings = (get-content -raw ".\settings.json")
$extName =  "KeyVaultForLinux"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForLinux"
 
# Start the deployment
Set-AzVmExtension -TypeHandlerVersion "3.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings

```

* To deploy the extension on a virtual machine scale set:

```powershell
    # Build settings
    $settings = (get-content -raw ".\settings.json")
    $extName = "KeyVaultForLinux"
    $extPublisher = "Microsoft.Azure.KeyVault"
    $extType = "KeyVaultForLinux"
      
    # Add extension to Virtual Machine Scale Sets
    $vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
    Add-AzVmssExtension -VirtualMachineScaleSet $vmss  -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "3.0" -Setting $settings
    
    # Start the deployment
    Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss 
``` 

## Azure CLI deployment

The Azure CLI can be used to deploy the Key Vault VM extension to an existing virtual machine or virtual machine scale set.

* To deploy the extension on a VM:

The Azure Key Vault VM extension can be deployed by using the Azure CLI. Save Key Vault VM extension settings to a JSON file (settings.json). 

The following JSON snippets provide example settings for deploying the Key Vault VM extension with the Azure CLI.

```json
{
   "secretsManagementSettings": {
   "pollingIntervalInS": "3600",
   "linkOnRenewal": true,
   "aclEnabled": true,
   "observedCertificates":
   [
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate1",
          "certificateStoreLocation":  "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
          "acls": 
          [
              {
                  "user": "app1",
                  "group": "appGroup1"
              },
              {
                  "user": "service1"
              }
          ]
      },
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate2",
          "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
          "acls": 
          [
              {
                  "user": "app2"
              }
          ]
      }
   ]},
   "authenticationSettings": {
      "msiEndpoint":  "http://169.254.169.254/metadata/identity/oauth2/token",
      "msiClientId":  "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxx"
   }      
}

```

* To deploy the extension on a virtual machine

```azurecli

    # Start the deployment
      az vm extension set -n "KeyVaultForLinux" `
      --publisher Microsoft.Azure.KeyVault `
      -g "<resourcegroup>" `
      --vm-name "<vmName>" `
      --version 3.0 `
      --enable-auto-upgrade true `
      --settings "@settings.json"

```

* To deploy the extension on a virtual machine scale set:

```azurecli
    # Start the deployment
    az vmss extension set -n "KeyVaultForLinux" `
    --publisher Microsoft.Azure.KeyVault `
    -g "<resourcegroup>" `
    --vmss-name "<vmssName>" `
    --version 3.0 `
    --enable-auto-upgrade true `
    --settings "@settings.json"

```

Please be aware of the following restrictions/requirements:
- Key Vault restrictions:
  - It must exist at the time of the deployment
  - The Key Vault Secrets User role must be assigned to Key Vault for VM identity


## Troubleshoot and Support

Data about the state of extension deployments can be retrieved from the Azure portal, and by using the Azure PowerShell. To see the deployment state of extensions for a given VM, run the following command using the Azure PowerShell.

**Azure PowerShell**
```powershell
Get-AzVMExtension -VMName <vmName> -ResourceGroupname <resource group name>
```

**Azure CLI**
```azurecli
 az vm get-instance-view --resource-group <resource group name> --name  <vmName> --query "instanceView.extensions"
```

[!INCLUDE [azure-cli-troubleshooting.md](~/reusable-content/ce-skilling/azure/includes/azure-cli-troubleshooting.md)]

### Logs and configuration

The Key Vault VM extension logs exist locally on the VM and are most informative when it comes to troubleshooting. You can use optional logging section to integrate with logging provider through `fluentd`

|Location|Description|
|--|--|
| /var/log/waagent.log	| Shows when an update to the extension occurred. |
| /var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*	| Examine the Key Vault VM Extension service logs to determine the status of the akvvm_service service and certificate download. You can find the download location of PEM files in files with an entry called certificate file name. If certificateStoreLocation is not specified, it will default to /var/lib/waagent/Microsoft.Azure.KeyVault.Store/ |
| /var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-\<most recent version\>/config/*	| The configuration and binaries for Key Vault VM Extension service. |
|||

### Using Symlink

Symbolic links or Symlinks are advanced shortcuts. To avoid monitoring the folder and to get the latest certificate automatically, you can use this symlink `([VaultName].[CertificateName])` to get the latest version of certificate on Linux.

## Certificate Installation on Linux

The Key Vault VM extension for Linux installs certificates as PEM files with the full certificate chain included. When a certificate is downloaded from Key Vault, the extension:

1. Creates a storage folder based on the `certificateStoreLocation` setting (defaults to `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/` if not specified)
2. Installs the full certificate chain and private key in a PEM file following [RFC 5246 section 7.4.2](https://datatracker.ietf.org/doc/html/rfc5246#section-7.4.2) in this specific order:
   - Leaf certificate (end-entity certificate) comes first
   - Intermediate certificate(s) follow in order, where each certificate directly certifies the one before it
   - Root certificate, if present (though it's not required for validation if already trusted by the system)
   - Private key, corresponding to the leaf certificate, is placed at the end of the file
3. Automatically creates a symbolic link named `[VaultName].[CertificateName]` that points to the latest version of the certificate

This approach ensures that:
- Applications always have access to the complete certificate chain needed for validation
- The certificate chain is properly ordered for TLS handshakes according to RFC standards
- The private key is available for use by the service
- Applications can reference a stable symbolic link path that automatically updates when certificates are renewed
- No application reconfiguration is needed when certificates are rotated or renewed

### Example Certificate Path Structure

For a certificate from `exampleVault.vault.azure.net` with the name `myCertificate`, the directory structure would look like:

```
/var/lib/waagent/Microsoft.Azure.KeyVault.Store/
├── exampleVault.myCertificate -> exampleVault.myCertificate.1234567890abcdef
├── exampleVault.myCertificate.1234567890abcdef    # Full chain PEM file (current version)
└── exampleVault.myCertificate.0987654321fedcba    # Previous version (if exists)
```

Applications should be configured to use the symbolic link path (`/var/lib/waagent/Microsoft.Azure.KeyVault.Store/exampleVault.myCertificate`) to ensure they always access the most current certificate version.

When you use custom certificate store locations and the `customSymbolicLinkName` setting, the structure follows this pattern:

```
/path/to/custom/store/
├── customLinkName -> exampleVault.myCertificate.1234567890abcdef
└── exampleVault.myCertificate.1234567890abcdef    # Full chain PEM file
```

### Frequently Asked Questions

* Is there's a limit on the number of observedCertificates you can configure?
  No, Key Vault VM Extension doesn't have limit on the number of observedCertificates.

### Support

If you need more help at any point in this article, you can contact the Azure experts on the [MSDN Azure and Stack Overflow forums](https://azure.microsoft.com/support/forums/). Alternatively, you can file an Azure support incident. Go to the [Azure support site](https://azure.microsoft.com/support/options/) and select Get support. For information about using Azure Support, read the [Microsoft Azure support FAQ](https://azure.microsoft.com/support/faq/).
