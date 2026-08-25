---
title: Azure Key Vault virtual machine extension for Linux
description: Learn how to deploy an agent for automatic refresh of Azure Key Vault certificates on virtual machines by using a VM extension.
services: virtual-machines
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.collection: linux
ms.topic: how-to
ms.date: 08/24/2026
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell, devx-track-azurecli, linux-related-content
ai-usage: ai-assisted
# Customer intent: As a system administrator managing Linux virtual machines, I want to deploy the Key Vault VM extension so that I can automate the refresh of certificates stored in Azure Key Vault and ensure seamless certificate management.
---
# Azure Key Vault virtual machine extension for Linux

The Azure Key Vault virtual machine (VM) extension automatically refreshes certificates stored in an Azure key vault. The extension monitors a list of observed certificates stored in key vaults. When the extension detects a change, it retrieves and installs the corresponding certificates. This article describes the supported platforms, configurations, and deployment options for the Key Vault VM extension for Linux.

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

## Operating systems

The Key Vault VM extension for Linux supports the following distributions, on both AMD64 and ARM64:

- Ubuntu 24.04
- Azure Linux 3.0 and 4.0
- Red Hat Enterprise Linux (RHEL) 9

> [!NOTE]
> The extension selects a distribution-specific binary at install time from `/etc/os-release`. Installing on any other distribution fails with a "distribution is not supported" error that appears in the extension status.

### Supported certificate content types

The Key Vault VM extension supports the following certificate content types:

- PKCS #12
- PEM

> [!NOTE]
> The Key Vault VM extension downloads all certificates to the location you specify in the `certificateStoreLocation` property in the VM extension settings, or to the default store location `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/` when you don't specify one.

## Features

The Key Vault VM extension for Linux version 4.x:

- Installs the two newest versions of each certificate.
- Installs each certificate as split files: a full-chain `.pem` file and a separate `.keyid` private key file, each written as a versioned file with a stable symbolic link that points to the latest version.
- Performs certificate chain validation before installing any certificate that carries the TLS Server Authentication Extended Key Usage (EKU). Validation is fail-open: a certificate is still installed if validation can't complete because of transient network issues. Certificates without the Server Authentication EKU aren't subject to this check.
- Applies POSIX ACLs to grant configured users and groups read access to the private key. ACL enforcement is always on.
- Supports an optional per-certificate authentication override, which allows individual observed certificates to authenticate to Key Vault with a different managed identity than the extension default. For more information, see [Extension schema](#extension-schema).
- Supports VM extension logging integration through Fluentd. For more information, see [Logging with Fluentd](#logging-with-fluentd).

## Upgrading from 3.0

If you're updating from 3.0, the following features are changed or removed.

General breaking changes:

- `pollingIntervalInS` is now limited to between 5 and 60 minutes. By default, the extension polls once each hour.
- `requireInitialSync` is removed. The extension only reports success if it installs all configured certificates.
- You can no longer configure a specific version of a certificate. Observed certificate URLs must be versionless.
- The legacy schema where `observedCertificates` is a list of URL strings is no longer supported. Each entry must be an object with a `url` property.

Linux-specific breaking changes:

- The certificate chain and private key are now written to separate files. In 3.0, the full chain and the private key were combined into a single PEM file. In 4.x, the extension writes the full chain to `<vaultname>.<certname>.pem` and the private key to a separate `<vaultname>.<certname>.keyid` file. This change is breaking: applications that expect the key and chain in one file must be updated to read the chain from the `.pem` file and the private key from the `.keyid` file. The `.luma` file is updated after the symbolic links are updated, so applications should monitor changes to this metadata file.
- `customSymbolicLinkName` is removed. The extension always uses the default symbolic link name `<vaultname>.<certname>`.
- `aclEnabled` is removed. ACL functionality is now always enabled.
- `certificateStoreName` is ignored on Linux and has no effect.

> [!NOTE]
> Upgrading from the previous extension version doesn't delete certificates that were already downloaded to disk. Additionally, 4.x uses a different file name format, so any existing files remain untouched.

## Prerequisites

Review the following prerequisites for using the Key Vault VM extension for Linux:

- An Azure Key Vault instance with a certificate. For more information, see [Create a key vault by using the Azure portal](/azure/key-vault/general/quick-create-portal).

- A VM with an assigned [managed identity](/entra/identity/managed-identities-azure-resources/overview).

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

- Observed certificate URLs must use the form `https://myVaultName.vault.azure.net/secrets/myCertName`.

   This form is required because the `/secrets` path returns the full certificate, including the private key, but the `/certificates` path doesn't. For more information about certificates, see [Azure Key Vault keys, secrets and certificates overview](/azure/key-vault/general/about-keys-secrets-certificates). You can't specify a specific version of the certificate.

- The URL host must be a recognized Azure Key Vault host.

- The `authenticationSettings` property is **required** for VMs with any **user assigned identities**, and for Azure Arc-enabled VMs.

   Omit this property when you use a system-assigned identity. For Azure Arc-enabled VMs, set `msiEndpoint` to `http://localhost:40342/metadata/identity`.

```json
{
   "type": "Microsoft.Compute/virtualMachines/extensions",
   "name": "KVVMExtensionForLinux",
   "apiVersion": "2025-04-01",
   "location": "<location>",
   "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
   ],
   "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "4.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
         "secretsManagementSettings": {
             "pollingIntervalInS": <Optional. Polling interval in seconds, between 300 (5 min) and 3600 (60 min). Example: "3600">,
             "certificateStoreLocation": <Optional. Default disk path where certificates are stored. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault.Store">,
             "observedCertificates": <An array of Key Vault URIs that represent monitored certificates, including per-certificate store location and ACL permissions on the certificate private key. Example:
             [
                {
                    "url": <A Key Vault URI to the secret portion of the certificate. Example: "https://myvault.vault.azure.net/secrets/mycertificate1">,
                    "certificateStoreLocation": <The disk path where the certificate is stored. Example: "/var/lib/waagent/Microsoft.Azure.KeyVault/app1">,
                    "acls": <Optional. An array of users and groups to grant read access to the certificate private key. Example:
                    [
                       { "user": "app1", "group": "appGroup1" },
                       { "user": "service1" }
                    ]>
                },
                {
                    "url": <Example: "https://myvault.vault.azure.net/secrets/mycertificate2">,
                    "certificateStoreLocation": <Example: "/var/lib/waagent/Microsoft.Azure.KeyVault/app2">,
                    "authenticationOverride": <Optional. Overrides authenticationSettings for this certificate only, so it can authenticate with a different managed identity. Example: {"msiClientId": "11112222-bbbb-3333-cccc-4444dddd5555"}>
                }
             ]>
         },
         "authenticationSettings": {
             "msiEndpoint":  <Required when the msiClientId property is used. Specifies the MSI endpoint. Example for most Azure VMs: "http://169.254.169.254/metadata/identity">,
             "msiClientId":  <Required when the VM has any user assigned identities. Specifies the MSI identity. Example: "00001111-aaaa-2222-bbbb-3333cccc4444">
         }
      }
   }
}
```

### Property values

The JSON schema includes the following properties.

| Name | Value/Example | Data type |
| --- | --- | --- |
| `apiVersion` | 2025-04-01 | date |
| `publisher` | Microsoft.Azure.KeyVault | string |
| `type` | KeyVaultForLinux | string |
| `typeHandlerVersion` | "4.0" | string |
| `pollingIntervalInS` (optional) | "3600" (clamped to 300–3600) | string |
| `certificateStoreLocation` (optional) | "/var/lib/waagent/Microsoft.Azure.KeyVault.Store" | string |
| `observedCertificates` | [{...}, {...}] | array |
| `observedCertificates/url` | "https://myvault.vault.azure.net/secrets/mycertificate" | string |
| `observedCertificates/certificateStoreLocation` (optional) | "/var/lib/waagent/Microsoft.Azure.KeyVault/app1" | string |
| `observedCertificates/acls` (optional) | [{"user": "app1", "group": "appGroup1"}] | object array |
| `observedCertificates/authenticationOverride` (optional) | {"msiClientId": "00001111-aaaa-2222-bbbb-3333cccc4444"} | object |
| `authenticationSettings/msiEndpoint` | "http://169.254.169.254/metadata/identity" | string |
| `authenticationSettings/msiClientId` | "00001111-aaaa-2222-bbbb-3333cccc4444" | string |

> [!NOTE]
> The schema accepts `certificateStoreName` for compatibility but Linux ignores it. If you don't specify `certificateStoreLocation` for a certificate, the system uses the top-level `secretsManagementSettings.certificateStoreLocation`, and if that's not set, it uses the default `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`.

## Template deployment

Deploy Azure VM extensions by using Azure Resource Manager (ARM) templates. Templates are ideal when you deploy one or more virtual machines that require post-deployment refresh of certificates. You can deploy the extension to individual VMs or Virtual Machine Scale Sets instances. The schema and configuration are common to both template types.

The JSON configuration for a key vault extension is nested inside the VM or Virtual Machine Scale Sets template. For a VM resource extension, the configuration is nested under the `"resources": []` virtual machine object. For a Virtual Machine Scale Sets instance extension, the configuration is nested under the `"virtualMachineProfile":"extensionProfile":{"extensions" :[]` object.

The following JSON snippet provides example settings for an ARM template deployment of the Key Vault VM extension.

```json
{
   "type": "Microsoft.Compute/virtualMachines/extensions",
   "name": "KeyVaultForLinux",
   "apiVersion": "2025-04-01",
   "location": "<location>",
   "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
   ],
   "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "4.0",
      "autoUpgradeMinorVersion": true,
      "enableAutomaticUpgrade": true,
      "settings": {
         "secretsManagementSettings": {
             "pollingIntervalInS": "3600",
             "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
             "observedCertificates": [
                {
                    "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate1",
                    "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
                    "acls": [
                       { "user": "app1", "group": "appGroup1" },
                       { "user": "service1" }
                    ]
                },
                {
                    "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate2",
                    "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store"
                }
             ]
         },
         "authenticationSettings": {
            "msiEndpoint":  "http://169.254.169.254/metadata/identity",
            "msiClientId":  "00001111-aaaa-2222-bbbb-3333cccc4444"
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

> [!WARNING]
> PowerShell clients often add `\` before `"` in settings.json. This behavior causes `akvvm_service` to fail with the error `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`. Use the Azure CLI, or pass the settings as a raw string as shown in the following example.

The following JSON snippet provides example settings for deploying the Key Vault VM extension by using PowerShell.

```json
{
   "secretsManagementSettings": {
      "pollingIntervalInS": "3600",
      "observedCertificates": [
         {
            "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate1",
            "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
            "acls": [
               { "user": "app1", "group": "appGroup1" },
               { "user": "service1" }
            ]
         },
         {
            "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate2",
            "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store"
         }
      ]
   },
   "authenticationSettings": {
      "msiEndpoint":  "http://169.254.169.254/metadata/identity",
      "msiClientId":  "00001111-aaaa-2222-bbbb-3333cccc4444"
   }
}
```

### Deploy on a VM

```powershell
# Build settings
$settings = (Get-Content -Raw ".\settings.json")
$extName =  "KeyVaultForLinux"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForLinux"

# Start the deployment
Set-AzVmExtension -TypeHandlerVersion "4.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
```

### Deploy on a Virtual Machine Scale Sets instance

```powershell
# Build settings
$settings = (Get-Content -Raw ".\settings.json")
$extName = "KeyVaultForLinux"
$extPublisher = "Microsoft.Azure.KeyVault"
$extType = "KeyVaultForLinux"

# Add extension to Virtual Machine Scale Sets
$vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "4.0" -Setting $settings

# Start the deployment
Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss
```

## Azure CLI deployment

Deploy the Azure Key Vault VM extension by using the Azure CLI. Save Key Vault VM extension settings to a JSON file (settings.json).

The following JSON snippet provides example settings for deploying the Key Vault VM extension by using the Azure CLI.

```json
{
   "secretsManagementSettings": {
      "pollingIntervalInS": "3600",
      "observedCertificates": [
         {
            "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate1",
            "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store",
            "acls": [
               { "user": "app1", "group": "appGroup1" },
               { "user": "service1" }
            ]
         },
         {
            "url": "https://<examplekv>.vault.azure.net/secrets/mycertificate2",
            "certificateStoreLocation": "/var/lib/waagent/Microsoft.Azure.KeyVault.Store"
         }
      ]
   },
   "authenticationSettings": {
      "msiEndpoint":  "http://169.254.169.254/metadata/identity",
      "msiClientId":  "00001111-aaaa-2222-bbbb-3333cccc4444"
   }
}
```

### Deploy on a VM

```azurecli
# Start the deployment
az vm extension set --name "KeyVaultForLinux" \
  --publisher Microsoft.Azure.KeyVault \
  --resource-group "<resourcegroup>" \
  --vm-name "<vmName>" \
  --version "4.0" \
  --enable-auto-upgrade true \
  --settings "@settings.json"
```

### Deploy on a Virtual Machine Scale Sets instance

```azurecli
# Start the deployment
az vmss extension set --name "KeyVaultForLinux" \
  --publisher Microsoft.Azure.KeyVault \
  --resource-group "<resourcegroup>" \
  --vmss-name "<vmssName>" \
  --version "4.0" \
  --enable-auto-upgrade true \
  --settings "@settings.json"
```

> [!TIP]
> If the extension deployment fails, you might need to delete the existing extension before reinstalling with the correct version. Azure doesn't allow extension downgrades, so you might need to remove the faulty extension first:
>
> ```azurecli
> az vm extension delete --name "KeyVaultForLinux" --resource-group "<resourcegroup>" --vm-name "<vmName>"
> ```

## Logging with Fluentd

The Key Vault VM extension can forward its logs to a Fluentd log collector. Make sure your log collector is running and listening at the endpoint you specify in the settings.

Add the following section to your extension settings:

```json
"loggingSettings": {
   "logger": "fluentd",
   "endpoint": "unix:///var/run/azuremonitoragent/sometenant/default_fluent.socket",
   "format": "forward",
   "servicename": "akvvm_service"
}
```

| Name | Value/Example | Data type |
| --- | --- | --- |
| `loggingSettings/logger` | "fluentd" | string |
| `loggingSettings/endpoint` | "unix:///var/run/azuremonitoragent/sometenant/default_fluent.socket" or "tcp://localhost:24224" | string |
| `loggingSettings/format` | "forward" | string |
| `loggingSettings/servicename` | "akvvm_service" | string |

## <a name="troubleshoot-and-support"></a> Troubleshoot problems

Use these suggestions to troubleshoot deployment problems.

### Check frequently asked questions

#### Is there a limit on the number of observed certificates?

No. The Key Vault VM extension doesn't limit the number of observed certificates (`observedCertificates`).

#### What is the default location where certificates are installed?

If you don't specify `certificateStoreLocation`, the extension writes certificates to `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`.

#### How do I force the extension to pull a new certificate?

Restart the `akvvm_service` service (display name **Key Vault VM Extension**).

#### How do I use a different identity for a specific certificate?

Add an `authenticationOverride` object with the target `msiClientId` to that certificate's entry in `observedCertificates`. Certificates without an override use the top-level `authenticationSettings`.

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

[!INCLUDE [azure-cli-troubleshooting.md](~/reusable-content/ce-skilling/azure/includes/azure-cli-troubleshooting.md)]

### Review logs and configuration

The Key Vault VM extension logs exist only locally on the VM. Review the log details to help with troubleshooting.

| Log file | Description |
| --- | --- |
| `/var/log/waagent.log` | Shows when updates occur to the extension. |
| `/var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*` | Shows the status of the `akvvm_service` service and certificate download. The PEM file download location appears in entries named certificate file name. |
| `/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-<most recent version>/config/*` | The configuration and binaries for the Key Vault VM extension service. |

## Certificate installation on Linux

The Key Vault VM extension for Linux installs certificates as PEM files. When the extension downloads a certificate from Key Vault, it:

1. Creates a storage folder based on the `certificateStoreLocation` setting. If you don't specify this setting, the location defaults to `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`.
1. Writes the certificate chain (leaf, then intermediates, then root if present in Key Vault) to a versioned full-chain `.pem` file, and writes the corresponding private key to a versioned `.keyid` file.
1. Applies POSIX ACLs to the private key based on the `acls` specified in the configuration, which grants read access to the listed users and groups. Files are otherwise owner-only.
1. Creates or updates a stable symbolic link (`<vaultname>.<certname>.pem` and `<vaultname>.<certname>.keyid`) that points to the latest version of the certificate. Linking always occurs.

### Default certificate store location

If you don't specify a location, the extension installs certificates under `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/`. The extension ignores `certificateStoreName` on Linux.

### Certificate output files

For vault `mykv` and secret `server-tls`, a successful sync produces:

```output
/var/lib/waagent/Microsoft.Azure.KeyVault.Store/
├── mykv.server-tls.pem -> mykv.server-tls.<version>.pem.<timestamp>     # symlink to latest full chain
├── mykv.server-tls.keyid -> mykv.server-tls.<version>.keyid.<timestamp> # symlink to latest private key
├── mykv.server-tls.<version>.pem.<timestamp>                            # full chain PEM (mode 600)
├── mykv.server-tls.<version>.keyid.<timestamp>                          # private key (mode 600)
└── mykv.server-tls.luma                                                 # certificate management metadata (mode 644)
```

Configure applications to reference the stable symbolic link path (for example, `/var/lib/waagent/Microsoft.Azure.KeyVault.Store/mykv.server-tls.pem`) so they always read the most current certificate version without reconfiguration on renewal.

### Certificate access control

By default, certificate and private key files are readable only by their owner. Grant read access to additional users and groups by using the `acls` array in the certificate configuration:

```json
"acls": [
   { "user": "app1", "group": "appGroup1" },
   { "user": "service1" }
]
```

Each entry can specify a user, a group, or both. ACL enforcement is always enabled and currently grants read access.

### Certificate renewal

When certificates are renewed in Key Vault, the extension automatically performs the following actions on the next poll:

1. Downloads the new certificate version.
1. Writes the new versioned `.pem` and `.keyid` files.
1. Updates the stable symbolic link to point to the new version so existing application paths continue to resolve to the latest certificate.

### Get support

Microsoft provides support only for major version 3.0 and later of the Key Vault VM extension. If you're using version 1.0, upgrade to the latest version before requesting support.

Use these other options to help resolve deployment problems:

- For assistance, contact the Azure experts in [Microsoft Q&A](/answers/tags/133/azure-virtual-machines).

- If you don't find an answer on the site, you can post a question for input from Microsoft or other members of the community.

- You can also [Contact Microsoft Support](https://support.microsoft.com/contactus/). For information about using Azure support, see [How to create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
