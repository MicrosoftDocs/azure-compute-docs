---
title: Azure Key Vault VM extension for Windows
description: Learn how to deploy an agent for automatic refresh of Azure Key Vault secrets on virtual machines with a virtual machine extension.
services: virtual-machines
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: windows
ms.topic: how-to
ms.date: 07/14/2026
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator managing virtual machines, I want to deploy an Azure Key Vault extension for automatic certificate refresh, so that I can ensure my VM's certificates are up to date without manual intervention."
---

# Azure Key Vault virtual machine extension for Windows

The Azure Key Vault virtual machine (VM) extension provides automatic refresh of certificates stored in an Azure key vault. The extension monitors a list of observed certificates stored in key vaults. When it detects a change, the extension retrieves and installs the corresponding certificates. This article describes the supported platforms, configurations, and deployment options for the Key Vault VM extension for Windows.

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

## Operating systems

The Key Vault VM extension supports Windows Server 2022 and Windows Server 2025, on both AMD64 and ARM64. On Windows Server 2025, private keys are saved in KeyGuard.

> [!NOTE]
> Version 4.0 of the Key Vault VM extension doesn't install on Windows Server 2019 or earlier.

### Supported certificates

The Key Vault VM extension supports the following certificate content types:

- PKCS #12
- PEM

> [!NOTE]
> The Key Vault VM extension downloads all certificates to the Windows certificate store or to the location you specify in the `certificateStoreLocation` property in the VM extension settings.

## Features

The Key Vault VM extension for Windows version 4.0:

- Installs private keys into KeyGuard if running on Windows Server 2025.
- Installs the two newest versions of each certificate.
- Performs certificate chain validation before installing any certificate that contains the TLS Server Authentication Extended Key Usage (EKU), including certificates that carry other EKUs alongside it (such as Client Authentication). Chain validation errors result in a provisioning failure for the extension. Certificates without the Server Authentication EKU aren't subject to this check.

## Upgrading from 3.0

If you're updating from 3.0, the following features are changed or removed:

- `pollingIntervalInS` is now limited to between 5 and 60 minutes. By default, polling is performed once each hour.
- `linkOnRenewal` is removed. Linking always occurs.
- `keyExportable` is removed. Private keys are no longer exportable.
- `requireInitialSync` is removed. The extension only reports success if all configured certificates are installed.
- You can no longer configure a specific version of a certificate.
- The extension now always stores private keys by using [Cryptography API: Next Generation (CNG)](/windows/win32/seccng/cng-portal) instead of CAPI.

## Prerequisites

Review the following prerequisites for using the Key Vault VM extension for Windows:

- An Azure Key Vault instance with a certificate. For more information, see [Create a key vault by using the Azure portal](/azure/key-vault/general/quick-create-portal).

- A VM with an assigned [managed identity](/azure/active-directory/managed-identities-azure-resources/overview).

- Assign the **Key Vault Secrets User** role at the Key Vault scope level to the managed identity for the VM or Azure Virtual Machine Scale Sets. This role retrieves the secret portion of a certificate. For more information, see the following articles:
   - [Authentication in Azure Key Vault](/azure/key-vault/general/authentication)
   - [Use Azure RBAC secret, key, and certificate permissions with Azure Key Vault](/azure/key-vault/general/rbac-guide#using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault)
   - [Key Vault scope role assignment](/azure/key-vault/general/rbac-guide?tabs=azure-cli#key-vault-scope-role-assignment)

- Configure Virtual Machine Scale Sets with the following `identity` configuration:

   ```json
   "identity": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
         "[parameters('userAssignedIdentityResourceId')]": {}
      }
   }
   ```

- Configure the Key Vault VM extension with the following `authenticationSettings` configuration:

   ```json
   "authenticationSettings": {
      "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
      "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
   }
   ```

> [!NOTE]
> You can also use the old access policy permission model to provide access to VMs and Virtual Machine Scale Sets. This method requires a policy with **get** and **list** permissions on secrets. For more information, see [Assign a Key Vault access policy](/azure/key-vault/general/assign-access-policy).

## Extension schema

The following JSON shows the schema for the Key Vault VM extension. Before you consider the schema implementation options, review the following important notes.

- The extension doesn't require protected settings. All settings are public information.

- Use the form `https://myVaultName.vault.azure.net/secrets/myCertName` for observed certificate URLs.

   This form is preferred because the `/secrets` path returns the full certificate, including the private key, but the `/certificates` path doesn't. For more information about certificates, see [Azure Key Vault keys, secrets and certificates overview](/azure/key-vault/general/about-keys-secrets-certificates).

- The `authenticationSettings` property is **required** for VMs with any **user assigned identities**.

   This property specifies the identity to use for authentication to Key Vault. Define this property with a system-assigned identity to avoid problems with a VM extension with multiple identities.

```json
{
   "type": "Microsoft.Compute/virtualMachines/extensions",
   "name": "KVVMExtensionForWindows",
   "apiVersion": "2025-04-01",
   "location": "<location>",
   "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
   ],
   "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForWindows",
      "typeHandlerVersion": "4.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
         "secretsManagementSettings": {
             "observedCertificates": <An array of Key Vault URIs that represent monitored certificates, including certificate store location and ACL permission to certificate private key. Example:
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. Example: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreName": <The certificate store name. Example: "MY">,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "LocalMachine">,
                    "accounts": <Optional. An array of preferred accounts with read access to certificate private keys. Administrators and SYSTEM get Full Control by default. Example: ["Network Service", "Local Service"]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate2">,
                    "certificateStoreName": <Example: "MY">,
                    "certificateStoreLocation": <Example: "CurrentUser">,
                    "accounts": <Example: ["Local Service"]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate3">,
                    "certificateStoreName": <Example: "TrustedPeople">,
                    "certificateStoreLocation": <Example: "LocalMachine">
                }
             ]>
         },
         "authenticationSettings": {
             "msiEndpoint":  <Required when the msiClientId property is used. Specifies the MSI endpoint. Example for most Azure VMs: "http://169.254.169.254/metadata/identity/oauth2/token">,
             "msiClientId":  <Required when the VM has any user assigned identities. Specifies the MSI identity. Example:  "00001111-aaaa-2222-bbbb-3333cccc4444">
         }
      }
   }
}
```

## Property values

The JSON schema includes the following properties.

| Name | Value/Example | Data type |
| --- | --- | --- |
| `apiVersion` | 2025-04-01 | date |
| `publisher` | Microsoft.Azure.KeyVault | string |
| `type` | KeyVaultForWindows | string |
| `typeHandlerVersion` | "4.0" | string |
| `observedCertificates`  | [{...}, {...}] | string array |
| `observedCertificates/url` | "https://myvault.vault.azure.net/secrets/mycertificate" | string |
| `observedCertificates/certificateStoreName` | MY | string |
| `observedCertificates/certificateStoreLocation`  | LocalMachine or CurrentUser (case sensitive) | string |
| `observedCertificates/accounts` (optional) | ["Network Service", "Local Service"] | string array |
| `msiEndpoint` | "http://169.254.169.254/metadata/identity/oauth2/token" | string |
| `msiClientId` | 00001111-aaaa-2222-bbbb-3333cccc4444 | string |

## Template deployment

Deploy Azure VM extensions by using Azure Resource Manager (ARM) templates. Templates are ideal when you deploy one or more virtual machines that require post-deployment refresh of certificates. You can deploy the extension to individual VMs or Virtual Machine Scale Sets instances. The schema and configuration are common to both template types.

The JSON configuration for a key vault extension is nested inside the VM or Virtual Machine Scale Sets template. For a VM resource extension, the configuration is nested under the `"resources": []` virtual machine object. For a Virtual Machine Scale Sets instance extension, the configuration is nested under the `"virtualMachineProfile":"extensionProfile":{"extensions" :[]` object.

The following JSON snippets provide example settings for an ARM template deployment of the Key Vault VM extension.

```json
{
   "type": "Microsoft.Compute/virtualMachines/extensions",
   "name": "KeyVaultForWindows",
   "apiVersion": "2025-04-01",
   "location": "<location>",
   "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
   ],
   "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForWindows",
      "typeHandlerVersion": "4.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
         "secretsManagementSettings": {
             "observedCertificates": <An array of Key Vault URIs that represent monitored certificates, including certificate store location and ACL permission to certificate private key. Example:
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. Example: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreName": <The certificate store name. Example: "MY">,
                    "certificateStoreLocation": <The certificate store location, which currently works locally only. Example: "LocalMachine">,
                    "accounts": <Optional. An array of preferred accounts with read access to certificate private keys. Administrators and SYSTEM get Full Control by default. Example: ["Network Service", "Local Service"]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate2">,
                    "certificateStoreName": <Example: "MY">,
                    "certificateStoreLocation": <Example: "CurrentUser">,
                    "accounts": <Example: ["Local Service"]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate3">,
                    "certificateStoreName": <Example: "TrustedPeople">,
                    "certificateStoreLocation": <Example: "LocalMachine">
                }
             ]>
         },
         "authenticationSettings": {
            "msiEndpoint":  <Required when the msiClientId property is used. Specifies the MSI endpoint. Example for most Azure VMs: "http://169.254.169.254/metadata/identity/oauth2/token">,
            "msiClientId":  <Required when the VM has any user assigned identities. Specifies the MSI identity. Example: "00001111-aaaa-2222-bbbb-3333cccc4444">
         }
      }
   }
}
```

### Extension automatic upgrade

The Key Vault VM extension supports automatic extension upgrade for virtual machines and scale sets in Azure. Azure keeps the extension up to date automatically when you set the `autoUpgradeMinorVersion` and `enableAutomaticUpgrade` properties in the preceding examples to `true`.

### Extension dependency ordering

The Key Vault VM extension supports extension dependency ordering. The extension reports a successful start after it downloads and installs all certificates.

If you use other extensions that require installation of certificates before they start, you can use extension dependency ordering to declare a dependency on the Key Vault VM extension.

On startup, the Key Vault VM extension retries download and install of certificates up to 25 times with increasing backoff periods, during which it remains in a **Transitioning** state. If the retries are exhausted, the extension reports an **Error** state. After all certificates are successfully installed, the Key Vault VM extension reports a successful start.

For more information about setting up dependencies between extensions, see [Sequence extension provisioning in Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-extension-sequencing).

> [!IMPORTANT]
> The extension dependency ordering feature isn't compatible with an ARM template that creates a system-assigned identity and updates a Key Vault access policy with that identity. If you attempt to use the feature in this scenario, a deadlock occurs because the Key Vault access policy can't update until after all extensions start. Instead, use a _single user-assigned managed identity_ and grant that identity access to your key vaults before you deploy.

## Azure PowerShell deployment

Deploy the Azure Key Vault VM extension by using Azure PowerShell. Save Key Vault VM extension settings to a JSON file (settings.json).

The following JSON snippets provide example settings for deploying the Key Vault VM extension by using PowerShell.

```json
{
   "secretsManagementSettings": {
   "observedCertificates":
   [
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/certificate1",
          "certificateStoreName": "MY",
          "certificateStoreLocation": "LocalMachine",
          "accounts": [
             "Network Service"
          ]
      },
      {
          "url": "https://<examplekv>.vault.azure.net/secrets/certificate2",
          "certificateStoreName": "MY",
          "certificateStoreLocation": "LocalMachine",
          "accounts": [
             "Network Service",
             "Local Service"
          ]
      }
   ]},
   "authenticationSettings": {
      "msiEndpoint":  "http://169.254.169.254/metadata/identity/oauth2/token",
      "msiClientId":  "00001111-aaaa-2222-bbbb-3333cccc4444"
   }
}
```

### Deploy on a VM

```powershell
# Build settings
$settings = (get-content -raw ".\settings.json")
$extName =  "KeyVaultForWindows"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForWindows"

# Start the deployment
Set-AzVmExtension -TypeHandlerVersion "4.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
```

### Deploy on a Virtual Machine Scale Sets instance

```powershell
# Build settings
$settings = ".\settings.json"
$extName = "KeyVaultForWindows"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForWindows"

# Add extension to Virtual Machine Scale Sets
$vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
Add-AzVmssExtension -VirtualMachineScaleSet $vmss  -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "4.0" -Setting $settings

# Start the deployment
Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss
```

## Azure CLI deployment

Deploy the Azure Key Vault VM extension by using the Azure CLI. Save Key Vault VM extension settings to a JSON file (settings.json).

The following JSON snippets provide example settings for deploying the Key Vault VM extension by using the Azure CLI.

```json
   {
        "secretsManagementSettings": {
          "observedCertificates": [
            {
                "url": "https://<examplekv>.vault.azure.net/secrets/certificate1",
                "certificateStoreName": "MY",
                "certificateStoreLocation": "LocalMachine",
                "accounts": [
                    "Network Service"
                ]
            },
            {
                "url": "https://<examplekv>.vault.azure.net/secrets/certificate2",
                "certificateStoreName": "MY",
                "certificateStoreLocation": "LocalMachine",
                "accounts": [
                    "Network Service",
                    "Local Service"
                ]
            }
        ]
        },
          "authenticationSettings": {
          "msiEndpoint":  "http://169.254.169.254/metadata/identity/oauth2/token",
          "msiClientId":  "00001111-aaaa-2222-bbbb-3333cccc4444"
        }
     }
```

### Deploy on a VM

```azurecli
# Start the deployment
az vm extension set --name "KeyVaultForWindows" `
 --publisher Microsoft.Azure.KeyVault `
 --resource-group "<resourcegroup>" `
 --vm-name "<vmName>" `
 --settings "@settings.json" `
 --version "4.0"
```

### Deploy on a Virtual Machine Scale Sets instance

```azurecli
# Start the deployment
az vmss extension set --name "KeyVaultForWindows" `
 --publisher Microsoft.Azure.KeyVault `
 --resource-group "<resourcegroup>" `
 --vmss-name "<vmssName>" `
 --settings "@settings.json" `
 --version "4.0"
```

> [!TIP]
> If the extension deployment fails, you might need to delete the existing extension before reinstalling with the correct version. Azure doesn't allow extension downgrades, so you might need to remove the faulty extension first:
>
> ```azurecli
> az vm extension delete --name "KeyVaultForWindows" --resource-group "<resourcegroup>" --vm-name "<vmName>"
> ```

## <a name="troubleshoot-and-support"></a> Troubleshoot problems

Use these suggestions to troubleshoot deployment problems.

### Check frequently asked questions

#### Is there a limit on the number of observed certificates?

No. The Key Vault VM extension doesn't limit the number of observed certificates (`observedCertificates`).

#### What's the default permission when no account is specified?

By default, Administrators and SYSTEM receive Full Control.

#### How do you determine if a certificate key is CAPI1 or CNG?

Starting with Key Vault VM extension 4.0, the extension saves private keys for all certificates by using CNG.

#### Does the extension support certificate auto-rebinding?

Yes, the Azure Key Vault VM extension supports certificate auto-rebinding. The Key Vault VM extension supports S-channel binding on certificate renewal.

For IIS, you can configure auto-rebind by enabling automatic rebinding of certificate renewals in IIS. The Azure Key Vault VM extension generates Certificate Lifecycle Notifications when a certificate with a matching SAN is installed. IIS uses this event to auto-rebind the certificate. For more information, see [Certificate Rebind in IIS](/iis/get-started/whats-new-in-iis-85/certificate-rebind-in-iis85).

### View extension status

Check the status of your extension deployment in the Azure portal, or by using PowerShell or the Azure CLI.

To see the deployment state of extensions for a given VM, run the following commands.

- Azure PowerShell:

   ```powershell
   Get-AzVMExtension -ResourceGroupName <myResourceGroup> -VMName <myVM> -Name <myExtensionName>
   ```

- The Azure CLI:

   ```azurecli
   az vm get-instance-view --resource-group <myResourceGroup> --name <myVM> --query "instanceView.extensions"
   ```

### Review logs and configuration

The Key Vault VM extension logs exist only locally on the VM. Review the log details to help with troubleshooting.

| Log file | Description |
| --- | --- |
| `C:\WindowsAzure\Logs\WaAppAgent.log` | Shows when updates occur to the extension. |
| `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.KeyVault.KeyVaultForWindows\<_most recent version_>\` | Shows the status of certificate download. The download location is always the Windows computer's MY store (`certlm.msc`). |
| `C:\Packages\Plugins\Microsoft.Azure.KeyVault.KeyVaultForWindows\<_most recent version_>\RuntimeSettings\` | The Key Vault VM extension service logs show the status of the `akvvm_service` service. |
| `C:\Packages\Plugins\Microsoft.Azure.KeyVault.KeyVaultForWindows\<_most recent version_>\Status\` | The configuration and binaries for the Key Vault VM extension service. |

## Certificate installation on Windows

The Key Vault VM extension for Windows installs certificates into the Windows certificate store. When the extension downloads a certificate from Key Vault, it:

1. Installs all intermediate and leaf certificates, regardless of how many intermediate certificates are present. The extension doesn't install root certificates because it isn't authorized to install them. Make sure that the root certificate is trusted on the system.
   - Installs leaf certificates in the specified certificate store (`certificateStoreName`) and location (`certificateStoreLocation`).
   - Installs intermediate CA certificates in the Intermediate Certificate Authorities store.
1. Places the certificates in the specified certificate store (`certificateStoreName`) and location (`certificateStoreLocation`).
1. Applies appropriate permissions to the private key based on the `accounts` specified in the configuration.
1. Sets the `CERT_RENEWAL` property so certificate bindings in applications such as IIS automatically update when certificates are renewed.

### Default certificate stores

By default, the extension installs certificates in the following locations:

- Store name: MY (Personal).
- Store location: LocalMachine.

### Certificate access control

By default, Administrators and SYSTEM receive Full Control permissions on installed certificates. You can customize access by using the `accounts` array in the certificate configuration:

```json
"accounts": ["Network Service", "Local Service"]
```

This configuration grants read access to the specified accounts, which allows applications running under those identities to use the certificates.

### Certificate renewal

When certificates are renewed in Key Vault, the extension automatically:

1. Downloads the new certificate version.
1. Installs the certificate in the configured certificate store.
1. Maintains existing bindings by using the `CERT_RENEWAL` property.

### Managing certificate lifecycle

For applications such as IIS that support Certificate Services Lifecycle Notifications, the Key Vault VM extension raises **Event 1001** in the Windows Event Log when it installs a certificate with a matching Subject Alternative Name (SAN). IIS subscribes to this event to auto-rebind the renewed certificate without interrupting service. Other applications and teams can also listen for Event 1001 to act on certificate renewals as needed. For more information, see [Certificate Services Lifecycle Notifications](/archive/technet-wiki/14250.certificate-services-lifecycle-notifications).

### Get support

Microsoft provides support only for major version 3.0 and later of the Key Vault VM extension. If you're using version 1.0, upgrade to the latest version before requesting support.

Use these other options to help resolve deployment problems:

- For assistance, contact the Azure experts in [Microsoft Q&A](/answers/tags/133/azure-virtual-machines).

- If you don't find an answer on the site, you can post a question for input from Microsoft or other members of the community.

- You can also [Contact Microsoft Support](https://support.microsoft.com/contactus/). For information about using Azure support, see [How to create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
