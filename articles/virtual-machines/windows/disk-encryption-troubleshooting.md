---
title: Azure Disk Encryption troubleshooting guide
description: This article provides troubleshooting tips for Microsoft Azure Disk Encryption for Windows VMs.
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: windows
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 05/14/2025
---
# Azure Disk Encryption troubleshooting guide

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

This guide is for IT professionals, information security analysts, and cloud administrators who use Azure Disk Encryption. This article helps troubleshoot disk encryption problems.

Before taking these steps, ensure the VMs you want to encrypt are among the [supported VM sizes and operating systems](disk-encryption-overview.md#supported-vms-and-operating-systems) and that you meet all the prerequisites:

- [Networking requirements](disk-encryption-overview.md#networking-requirements)
- [Group policy requirements](disk-encryption-overview.md#group-policy-requirements)
- [Encryption key storage requirements](disk-encryption-overview.md#encryption-key-storage-requirements)

## Troubleshooting 'failed to send DiskEncryptionData'

When encrypting a VM fails with the error message "Failed to send DiskEncryptionData...", it's usually caused by one of the following situations:

- The Key Vault exists in a different region or subscription than the virtual machine
- Advanced access policies in the Key Vault aren't set to allow Azure Disk Encryption
- The Key Encryption Key is disabled or deleted in the Key Vault
- A typo exists in the Resource ID or URL for the Key Vault or Key Encryption Key (KEK)
- Special characters are used in the names of the VM, data disks, or keys. For example, "_VMName" or "élite".
- The encryption scenario isn't supported
- Network issues prevent the VM or host from accessing the required resources

### Suggestions for resolving the issue

- Make sure the Key Vault exists in the same region and subscription as the Virtual Machine
- Ensure you [set key vault advanced access policies](disk-encryption-key-vault.yml#set-key-vault-advanced-access-policies) correctly
- If you are using KEK, ensure the key exists and is enabled in Key Vault
- Check that the VM name, data disks, and keys follow [key vault resource naming restrictions](/azure/azure-resource-manager/management/resource-name-rules#microsoftkeyvault).
- Check for typos in the Key Vault name or KEK name in PowerShell or CLI commands
>[!NOTE]
   > The syntax for the value of the `disk-encryption-keyvault` parameter is the full identifier string:
`/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]`</br>
   > The syntax for the value of the `key-encryption-key` parameter is the full URI to the KEK, such as:
`https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]`
- Ensure you are not violating any [restrictions](disk-encryption-windows.md#restrictions)
- Ensure you are meeting [network requirements](disk-encryption-overview.md#networking-requirements) and try again

## Troubleshooting Azure Disk Encryption behind a firewall

When connectivity is restricted by a firewall, proxy requirement, or network security group (NSG) settings, the extension may be unable to perform the needed tasks. This disruption can result in status messages such as "Extension status not available on the VM." In typical scenarios, the encryption doesn't finish. The sections that follow have some common firewall problems that you might investigate.

### Network security groups
Any network security group settings that are applied must still allow the endpoint to meet the documented network configuration [prerequisites](disk-encryption-overview.md#networking-requirements) for disk encryption.

### Azure Key Vault behind a firewall

When encryption is being enabled with [Microsoft Entra credentials](disk-encryption-windows-aad.md#), the target VM must allow connectivity to both Microsoft Entra endpoints and Key Vault endpoints. Current Microsoft Entra authentication endpoints are maintained in sections 56 and 59 of the [Microsoft 365 URLs and IP address ranges](/microsoft-365/enterprise/urls-and-ip-address-ranges) documentation. Key Vault instructions are provided in the documentation on how to [Access Azure Key Vault behind a firewall](/azure/key-vault/general/access-behind-firewall).

### Azure Instance Metadata Service 
The VM must be able to access the [Azure Instance Metadata service](../windows/instance-metadata-service.md) endpoint (`169.254.169.254`) and the [virtual public IP address](/azure/virtual-network/what-is-ip-address-168-63-129-16) (`168.63.129.16`) used for communication with Azure platform resources. Proxy configurations that alter local HTTP traffic to these addresses, like adding an X-Forwarded-For header, aren't supported.

## Troubleshooting Windows Server 2016 Server Core

On Windows Server 2016 Server Core, the `bdehdcfg` component isn't available by default. Azure Disk Encryption requires this component. It's used to split the system volume from the OS volume, which is done only once for the lifetime of the VM. These binaries aren't required during later encryption operations.

To work around this issue, copy the following four files from a Windows Server 2016 Data Center VM to the same location on Server Core:

   ```
   \windows\system32\bdehdcfg.exe
   \windows\system32\bdehdcfglib.dll
   \windows\system32\en-US\bdehdcfglib.dll.mui
   \windows\system32\en-US\bdehdcfg.exe.mui
   ```

1. Run the following command:

   ```
   bdehdcfg.exe -target default
   ```

1. This command creates a 550 MB system partition. Restart the system.

1. Use DiskPart to check the volumes. Then proceed.

For example:

```
DISKPART> list vol

  Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  Volume 0     C                NTFS   Partition    126 GB  Healthy    Boot
  Volume 1                      NTFS   Partition    550 MB  Healthy    System
  Volume 2     D   Temporary S  NTFS   Partition     13 GB  Healthy    Pagefile
```

## Troubleshooting encryption status

The portal might display a disk as encrypted even after it is unencrypted within the VM. This situation can occur when low-level commands are used to directly unencrypt the disk from within the VM instead of using the higher-level Azure Disk Encryption management commands. The higher-level commands not only unencrypt the disk from within the VM but also update important platform-level encryption settings and extension settings associated with the VM. If these are not kept in alignment, the platform is unable to report encryption status or provision the VM properly.

To disable Azure Disk Encryption with PowerShell, use [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) followed by [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). Running Remove-AzVMDiskEncryptionExtension before the encryption is disabled fails.

To disable Azure Disk Encryption with the CLI, use [az vm encryption disable](/cli/azure/vm/encryption). 

## Next steps

In this document, you learned more about some common problems in Azure Disk Encryption and how to troubleshoot those problems. For more information about this service and its capabilities, see the following articles:

- [Apply disk encryption in Microsoft Defender for Cloud](/azure/security-center/asset-inventory)
- [Azure data encryption at rest](/azure/security/fundamentals/encryption-atrest)
