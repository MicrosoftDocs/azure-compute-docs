---
title: Azure Key Vault virtual machine extension for Linux
description: Learn how to deploy an agent for automatic refresh of Azure Key Vault certificates on virtual machines by using a VM extension.
services: virtual-machines
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: linux
ms.topic: how-to
ms.date: 07/14/2026
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell, devx-track-azurecli, linux-related-content
ai-usage: ai-assisted
# Customer intent: As a system administrator managing Linux virtual machines, I want to deploy the Key Vault VM extension so that I can automate the refresh of certificates stored in Azure Key Vault and ensure seamless certificate management.
---
# Azure Key Vault virtual machine extension for Linux

The Azure Key Vault virtual machine (VM) extension automatically refreshes certificates stored in an Azure key vault. The extension monitors a list of observed certificates stored in key vaults. When the extension detects a change, it retrieves and installs the corresponding certificates. This article describes the supported platforms, configurations, and deployment options for the Key Vault VM extension for Linux.

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

## Operating systems

The Key Vault VM extension supports:

- Ubuntu 22.04 and later.
- [Azure Linux](https://github.com/microsoft/azurelinux).

### Supported certificate content types

- PKCS #12
- PEM

## Features

The Key Vault VM extension for Linux version 3.0 and later supports:

- ACL permissions for downloaded certificates to provide read access for users and groups.
- Certificate installation location configuration.
- Custom symbolic name support.
- VM extension logging integration through [Fluentd](https://www.fluentd.org/).

## Prerequisites

- An Azure Key Vault instance with a certificate. For more information, see [Create a key vault by using the Azure portal](/azure/key-vault/general/quick-create-portal).
- Virtual machines or Azure Virtual Machine Scale Sets with an assigned [managed identity](/entra/identity/managed-identities-azure-resources/overview).
- The **Key Vault Secrets User** role at the Key Vault scope level for the managed identity on VMs and Azure Virtual Machine Scale Sets. This role retrieves a secret's portion of a certificate. For more information, see the following articles:
  - [Authentication in Azure Key Vault](/azure/key-vault/general/authentication).
  - [Use Azure RBAC secret, key, and certificate permissions with Azure Key Vault](/azure/key-vault/general/rbac-guide#using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault).
  - [Key Vault scope role assignment](/azure/key-vault/general/rbac-guide?tabs=azure-cli#key-vault-scope-role-assignment).
- Azure Virtual Machine Scale Sets should have the following identity setting:

  ```json
  "identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "[parameters('userAssignedIdentityResourceId')]": {}
    }
  }
  ```

- The Key Vault VM extension should have the following setting:

  ```json
  "authenticationSettings": {
    "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
    "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
  }
  ```

## Upgrade the Key Vault VM extension

- To upgrade from an older version to version 3.0 or later, delete the previous version, and then install version 3.0.

```azurecli
  az vm extension delete --name KeyVaultForLinux --resource-group ${resourceGroup} --vm-name ${vmName}
  az vm extension set -n "KeyVaultForLinux" --publisher Microsoft.Azure.KeyVault --resource-group "${resourceGroup}" --vm-name "${vmName}" --settings "@akvvm.json" --version "3.0"
```

- If the VM has certificates downloaded by the previous version, deleting the VM extension doesn't delete the downloaded certificates. After you install the newer version, the extension doesn't modify the existing certificates. Delete the certificate files or roll over the certificate to get the PEM file with the full chain on the VM.

## Extension schema

The following JSON provides the schema for the Key Vault VM extension. All settings use plain, unprotected settings because the settings don't contain sensitive information. To configure the extension, specify a list of certificates to monitor, how often to poll for updates, and the destination path for storing certificates.

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
          "requireInitialSync": <initial synchronization of certificates, e.g.: true>,
          "aclEnabled": <Enables ACLs for downloaded certificates, e.g.: true>,
          "observedCertificates": <An array of Key Vault URIs that represent monitored certificates, including certificate store location, ACL permission to certificate private key, and custom symbolic name. e.g.:
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
          "msiClientId":  <Required when VM has any user-assigned identities. MSI identity e.g.: "00001111-aaaa-2222-bbbb-3333cccc4444".>
        }>
       }
      }
    }
```

> [!NOTE]
> Your observed certificate URLs should use the form `https://myVaultName.vault.azure.net/secrets/myCertName`.
>
> The `/secrets` path returns the full certificate, including the private key, but the `/certificates` path doesn't. For more information about certificates, see [Azure Key Vault keys, secrets and certificates overview](/azure/key-vault/general/about-keys-secrets-certificates).

> [!IMPORTANT]
> The `authenticationSettings` property is required only when your VM uses **user-assigned managed identities**, your Azure Virtual Machine Scale Sets use **user-assigned managed identities**, or you use **Azure Arc-enabled VMs**. For **system-assigned managed identities**, omit the `authenticationSettings` section. Including this section causes deployment to fail. Without this section, a VM with user-assigned identities can't use the Key Vault extension to download certificates.
> Set `msiClientId` to the identity that will authenticate to Key Vault.
>
> The `msiEndpoint` property is also **required** for **Azure Arc-enabled VMs**. Set `msiEndpoint` to `http://localhost:40342/metadata/identity`.

### Property values

| Name | Value or example | Data type |
| ---- | ---- | ---- |
| `apiVersion` | 2022-11-01 | date |
| `publisher` | Microsoft.Azure.KeyVault | string |
| `type` | KeyVaultForLinux | string |
| `typeHandlerVersion` | "3.0" | string |
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

You can deploy Azure VM extensions by using Azure Resource Manager templates. Templates are ideal when you deploy one or more virtual machines that require post-deployment refresh of certificates. You can deploy the extension to individual VMs or Azure Virtual Machine Scale Sets. The schema and configuration are common to both template types.

> [!NOTE]
> The VM extension requires a system-assigned or user-assigned managed identity to authenticate to Key Vault. For more information, see [Configure managed identities for Azure resources on an Azure VM by using the Azure portal](/entra/identity/managed-identities-azure-resources/qs-configure-portal-windows-vm).

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
          "requireInitialSync": <initial synchronization of certificates, e.g.: false>,
          "aclEnabled": <enables or disables ACLs on defined certificates e.g.: true>,
          "observedCertificates": <An array of Key Vault URIs that represent monitored certificates, including certificate store location and ACL permission to certificate private key. Example:
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. Example: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault.Store">,
                    "acls": <Optional. An array of preferred ACLs with read access to certificate private keys. Example:
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
                    "certificateStoreName": <ignored on Linux>,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault.Store">,
                    "acls": <Optional. An array of preferred ACLs with read access to certificate private keys. Example:
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
              "msiClientId":  <Required when VM has any user-assigned identities. MSI identity e.g.: "00001111-aaaa-2222-bbbb-3333cccc4444">
          }
        }
      }
    }
```

### Extension dependency ordering

The Key Vault VM extension supports extension ordering if you configure it. By default, the extension reports a successful start as soon as polling starts. You can configure it to wait until it successfully downloads the complete list of certificates before it reports a successful start. If other extensions depend on installed certificates before they start, enable this setting. Those extensions can then declare a dependency on the Key Vault extension. This setting prevents those extensions from starting until all certificates they depend on are installed.

The extension retries the initial download up to 25 times with increasing backoff periods, during which it remains in a `Transitioning` state. If the retries are exhausted, the extension reports an `Error` state.

To turn on extension dependency, set the following:

```json
"secretsManagementSettings": {
    "requireInitialSync": true,
    ...
}
```

> [!NOTE]
> The extension dependency feature isn't compatible with an ARM template that creates a system-assigned identity and updates a Key Vault access policy with that identity. This configuration creates a deadlock because the vault access policy can't update until all extensions start. Instead, use a *single user-assigned MSI identity* and pre-ACL your vaults with that identity before you deploy.

## Azure PowerShell deployment

> [!WARNING]
> PowerShell clients often add `\` to `"` in `settings.json`. This behavior causes `akvvm_service` to fail with the error `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`

Use Azure PowerShell to deploy the Key Vault VM extension to an existing virtual machine or virtual machine scale set.

Save Key Vault VM extension settings to a JSON file named `settings.json`.

The following JSON snippets provide example settings for deploying the Key Vault VM extension by using PowerShell.

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

### Deploy on a VM by using Azure PowerShell

```powershell
# Build settings
$settings = (Get-Content -Raw ".\settings.json")
$extName =  "KeyVaultForLinux"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForLinux"

# Start the deployment
Set-AzVmExtension -TypeHandlerVersion "3.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
```

### Deploy on a virtual machine scale set by using Azure PowerShell

```powershell
# Build settings
$settings = (Get-Content -Raw ".\settings.json")
$extName = "KeyVaultForLinux"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForLinux"

# Add extension to Virtual Machine Scale Sets
$vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "3.0" -Setting $settings

# Start the deployment
Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss
```

## Azure CLI deployment

Use the Azure CLI to deploy the Key Vault VM extension to an existing virtual machine or virtual machine scale set.

Save Key Vault VM extension settings to a JSON file named `settings.json`.

The following JSON snippets provide example settings for deploying the Key Vault VM extension by using the Azure CLI.

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

### Deploy on a VM by using the Azure CLI

```azurecli
# Start the deployment
az vm extension set --name "KeyVaultForLinux" \
  --publisher Microsoft.Azure.KeyVault \
  --resource-group "<resourcegroup>" \
  --vm-name "<vmName>" \
  --version "3.0" \
  --enable-auto-upgrade true \
  --settings "@settings.json"
```

### Deploy on a virtual machine scale set by using the Azure CLI

```azurecli
# Start the deployment
az vmss extension set --name "KeyVaultForLinux" \
  --publisher Microsoft.Azure.KeyVault \
  --resource-group "<resourcegroup>" \
  --vmss-name "<vmssName>" \
  --version "3.0" \
  --enable-auto-upgrade true \
  --settings "@settings.json"
```

Review the following restrictions and requirements:

- Key Vault restrictions:
  - The key vault must exist at the time of the deployment.
  - The **Key Vault Secrets User** role must be assigned to Key Vault for the VM identity.

## Troubleshoot and support

Retrieve data about the state of extension deployments from the Azure portal, or by using Azure PowerShell or the Azure CLI. To see the deployment state of extensions for a given VM, run the following commands.

- Azure PowerShell:

```powershell
Get-AzVMExtension -VMName <vmName> -ResourceGroupname <resource group name>
```

- Azure CLI:

```azurecli
az vm get-instance-view --resource-group <resource group name> --name <vmName> --query "instanceView.extensions"
```

[!INCLUDE [azure-cli-troubleshooting.md](~/reusable-content/ce-skilling/azure/includes/azure-cli-troubleshooting.md)]

### Logs and configuration

The Key Vault VM extension logs exist locally on the VM and are most informative for troubleshooting. Use the optional logging section to integrate with a logging provider by using `fluentd`.

| Location | Description |
| --- | --- |
| `/var/log/waagent.log` | Shows when updates occur to the extension. |
| `/var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*` | Shows the status of the `akvvm_service` service and certificate download. You can find the PEM file download location in files with an entry named `certificate file name`. If `certificateStoreLocation` isn't specified, the location defaults to `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`. |
| `/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-<most recent version>/config/*` | Contains the configuration and binaries for the Key Vault VM extension service. |

### Use symbolic links

Symbolic links are advanced shortcuts. To avoid monitoring the folder and to get the latest certificate automatically, use the `[VaultName].[CertificateName]` symbolic link to get the latest certificate version on Linux.

## Certificate installation on Linux

The Key Vault VM extension for Linux installs certificates as PEM files. When the extension downloads a certificate from Key Vault, it:

1. Creates a storage folder based on the `certificateStoreLocation` setting. If you don't specify this setting, the location defaults to `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`.
1. Installs the certificate chain and private key as stored in Key Vault. The PEM file follows [RFC 5246 section 7.4.2](https://datatracker.ietf.org/doc/html/rfc5246#section-7.4.2) ordering:
   - Leaf certificate, or end-entity certificate, comes first.
   - Intermediate certificates follow in order, where each certificate directly certifies the one before it, if present in Key Vault.
   - Root certificate, if present. The root certificate isn't required for validation if the system already trusts it.
   - The extension places the private key that corresponds to the leaf certificate at the end of the file.
1. Automatically creates a symbolic link named `[VaultName].[CertificateName]` that points to the latest version of the certificate.

This installation approach ensures that:

- Applications have access to the certificate chain as stored in Key Vault.
- The certificate chain is ordered for TLS handshakes according to RFC standards.
- The private key is available for use by the service.
- Applications can reference a stable symbolic link path that automatically updates when certificates are renewed.
- No application reconfiguration is needed when certificates are rotated or renewed.

### Example certificate path structure

For a certificate from `exampleVault.vault.azure.net` with the name `myCertificate`, the directory structure looks like:

```output
/var/lib/waagent/Microsoft.Azure.KeyVault.Store/
├── exampleVault.myCertificate -> exampleVault.myCertificate.1234567890abcdef
├── exampleVault.myCertificate.1234567890abcdef    # Full chain PEM file (current version)
└── exampleVault.myCertificate.0987654321fedcba    # Previous version (if exists)
```

Configure applications to use the symbolic link path (`/var/lib/waagent/Microsoft.Azure.KeyVault.Store/exampleVault.myCertificate`). This configuration ensures that applications always access the most current certificate version.

When you use custom certificate store locations and the `customSymbolicLinkName` setting, the structure follows this pattern:

```output
/path/to/custom/store/
├── customLinkName -> exampleVault.myCertificate.1234567890abcdef
└── exampleVault.myCertificate.1234567890abcdef    # Full chain PEM file
```

### Frequently asked questions

#### Is there a limit on the number of observed certificates that I can configure?

No. The Key Vault VM extension doesn't limit the number of observed certificates (`observedCertificates`).

### Get support

Microsoft provides support only for major version 3.0 and later of the Key Vault VM extension. If you're using version 1.0, upgrade to the latest version before requesting support.

Here are some other options to help you resolve deployment problems:

- For assistance, contact the Azure experts in [Microsoft Q&A](/answers/tags/133/azure-virtual-machines).
- If you don't find an answer on the site, post a question for input from Microsoft or other members of the community.
- You can also [contact Microsoft Support](https://support.microsoft.com/contactus/). For information about using Azure support, see [How to create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
