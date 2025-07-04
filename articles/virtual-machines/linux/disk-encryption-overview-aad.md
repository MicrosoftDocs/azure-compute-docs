---
title: Azure Disk Encryption with Microsoft Entra app prerequisites (previous release)
description: This article provides supplements to Azure Disk Encryption for Linux VMs with additional requirements and prerequisites for Azure Disk Encryption with Microsoft Entra ID.
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.author: mbaldwin
ms.date: 05/14/2025
# Customer intent: As a cloud administrator, I want to understand the prerequisites for enabling Azure Disk Encryption with Microsoft Entra ID on Linux VMs, so that I can successfully configure and secure my virtual machines without encountering errors during the encryption process.
---
# Azure Disk Encryption with Microsoft Entra ID (previous release)

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

The new release of Azure Disk Encryption eliminates the requirement for providing a Microsoft Entra application parameter to enable VM disk encryption. With the new release, you're no longer required to provide Microsoft Entra credentials during the enable encryption step. All new VMs must be encrypted without the Microsoft Entra application parameters by using the new release. For instructions on how to enable VM disk encryption by using the new release, see [Azure Disk Encryption for Linux VMs](disk-encryption-overview.md). VMs that were already encrypted with Microsoft Entra application parameters are still supported and should continue to be maintained with the Microsoft Entra syntax.

This article provides supplements to [Azure Disk Encryption for Linux VMs](disk-encryption-overview.md) with additional requirements and prerequisites for Azure Disk Encryption with Microsoft Entra ID (previous release).

The information in these sections remains the same:

- [Supported VMs and operating systems](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Additional VM requirements](disk-encryption-overview.md#additional-vm-requirements)


## Networking and Group Policy

To enable the Azure Disk Encryption feature by using the older Microsoft Entra parameter syntax, the infrastructure as a service (IaaS) VMs must meet the following network endpoint configuration requirements: 
  - To get a token to connect to your key vault, the IaaS VM must be able to connect to a Microsoft Entra endpoint, \[login.microsoftonline.com\].
  - To write the encryption keys to your key vault, the IaaS VM must be able to connect to the key vault endpoint.
  - The IaaS VM must be able to connect to an Azure storage endpoint that hosts the Azure extension repository and an Azure storage account that hosts the VHD files.
  -  If your security policy limits access from Azure VMs to the internet, you can resolve the preceding URI and configure a specific rule to allow outbound connectivity to the IPs. For more information, see [Azure Key Vault behind a firewall](/azure/key-vault/general/access-behind-firewall).
  - On Windows, if TLS 1.0 is explicitly disabled and the .NET version isn't updated to 4.6 or higher, the following registry change enables Azure Disk Encryption to select the more recent TLS version:

  ```config-registry
  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001
    
  [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
  "SystemDefaultTlsVersions"=dword:00000001
  "SchUseStrongCrypto"=dword:00000001` 
  ```

### Group Policy
 - The Azure Disk Encryption solution uses the BitLocker external key protector for Windows IaaS VMs. For domain-joined VMs, don't push any Group Policies that enforce TPM protectors. For information about the Group Policy for the option **Allow BitLocker without a compatible TPM**, see [BitLocker Group Policy reference](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#bkmk-unlockpol1).

- BitLocker policy on domain-joined virtual machines with a custom Group Policy must include the following setting: [Configure user storage of BitLocker recovery information -> Allow 256-bit recovery key](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings). Azure Disk Encryption fails when custom Group Policy settings for BitLocker are incompatible. On machines that don't have the correct policy setting, apply the new policy, force the new policy to update (gpupdate.exe /force), and then restart if it's required. 

## Encryption key storage requirements 

Azure Disk Encryption requires Azure Key Vault to control and manage disk encryption keys and secrets. Your key vault and VMs must reside in the same Azure region and subscription.

For more information, see [Creating and configuring a key vault for Azure Disk Encryption with Microsoft Entra ID (previous release)](disk-encryption-key-vault-aad.md).
 
## Next steps

- [Creating and configuring a key vault for Azure Disk Encryption with Microsoft Entra ID (previous release)](disk-encryption-key-vault-aad.md)
- [Enable Azure Disk Encryption with Microsoft Entra ID on Linux VMs (previous release)](disk-encryption-linux-aad.md)
- [Azure Disk Encryption prerequisites CLI script](https://github.com/ejarvi/ade-cli-getting-started)
- [Azure Disk Encryption prerequisites PowerShell script](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
