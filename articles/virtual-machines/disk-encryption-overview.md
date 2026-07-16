---
title: Overview of managed disk encryption options
description: Compare Azure managed disk encryption options, including server-side encryption, encryption at host, Azure Disk Encryption, and confidential disk encryption.
author: msmbaldwin
ms.date: 07/14/2026
ms.topic: concept-article
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: references_regions
ai-usage: ai-assisted
# Customer intent: As a security administrator, I want to understand the different encryption options for managed disks, so that I can implement the most appropriate security measures for protecting data in Azure virtual machines.
---

# Overview of managed disk encryption options

Several encryption types are available for your managed disks, including Azure Disk Encryption (ADE), server-side encryption (SSE), and encryption at host.

- **Azure Disk Storage server-side encryption** (also referred to as encryption at rest or Azure Storage encryption) is always enabled. It automatically encrypts data stored on Azure managed disks (OS and data disks) when it persists data on the storage clusters. When you configure it with a disk encryption set (DES), it also supports customer-managed keys. It doesn't encrypt temp disks or disk caches. For full details, see [Server-side encryption of Azure Disk Storage](./disk-encryption.md).

- **Encryption at host** is a virtual machine option that enhances Azure Disk Storage server-side encryption. This option ensures that all temp disks and disk caches are encrypted at rest and flow encrypted to the storage clusters. For full details, see [Encryption at host - End-to-end encryption for your VM data](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

- **Confidential disk encryption** binds disk encryption keys to the virtual machine's TPM and makes the protected disk content accessible only to the VM. The TPM and VM guest state are always encrypted in attested code by using keys released by a secure protocol that bypasses the hypervisor and host operating system. Confidential OS disk encryption protects the OS disk. You can also enable [confidential temp disk encryption](/azure/confidential-computing/confidential-vm-overview#confidential-temp-disk-encryption) through an opt-in process. You can use encryption at host for other disks on a confidential VM in addition to confidential disk encryption. For full details, see [DCasv5 and ECasv5 series confidential VMs](/azure/confidential-computing/confidential-vm-overview#confidential-os-disk-encryption).

- **Azure Disk Encryption** helps protect and safeguard your data to meet your organizational security and compliance commitments. ADE encrypts the OS and data disks of Azure virtual machines (VMs) inside your VMs by using the [DM-Crypt](https://wikipedia.org/wiki/Dm-crypt) feature of Linux or the [BitLocker](https://wikipedia.org/wiki/BitLocker) feature of Windows. ADE is integrated with Azure Key Vault to help you control and manage the disk encryption keys and secrets, with the option to encrypt by using a key encryption key (KEK). For full details, see [Azure Disk Encryption for Linux VMs](./linux/disk-encryption-overview.md) or [Azure Disk Encryption for Windows VMs](./windows/disk-encryption-overview.md).

  [!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

Encryption is part of a layered approach to security. Use it with other recommendations to secure virtual machines and their disks. For full details, see [Compute security recommendations](/azure/defender-for-cloud/recommendations-reference-compute) and [Restrict import/export access to managed disks](/azure/virtual-machines/disks-enable-private-links-for-import-export-portal).

## Comparison

The following table compares Disk Storage SSE, ADE, encryption at host, and confidential disk encryption.

| &nbsp; | **Azure Disk Storage server-side encryption** | **Encryption at host** | **Azure Disk Encryption** | **Confidential disk encryption (for the OS disk only)** |
|--|--|--|--|--|
| Encryption at rest (OS and data disks) | &#x2705; | &#x2705; | &#x2705; | &#x2705; |
| Temp disk encryption | &#10060; | &#x2705; Supported only with platform-managed keys | &#x2705; | &#x2705; [Opt-in](/azure/confidential-computing/confidential-vm-overview#confidential-temp-disk-encryption) |
| Encryption of caches | &#10060; | &#x2705; | &#x2705; | &#x2705; |
| Data flows encrypted between compute and storage | &#10060; | &#x2705; | &#x2705; | &#x2705; |
| Customer control of keys | &#x2705; When configured with DES | &#x2705; When configured with DES | &#x2705; When configured with KEK | &#x2705; When configured with DES |
| HSM support | Azure Key Vault Premium and Managed HSM | Azure Key Vault Premium and Managed HSM | Azure Key Vault Premium | Azure Key Vault Premium and Managed HSM |
| Does not use your VM's CPU | &#x2705; | &#x2705; | &#10060; | &#10060; |
| Works for custom images | &#x2705; | &#x2705; | &#10060; Does not work for custom Linux images | &#x2705; |
| Enhanced key protection | &#10060; | &#10060; | &#10060; | &#x2705; |
| Microsoft Defender for Cloud disk encryption status* | Unhealthy | Healthy | Healthy | Not applicable |

> [!IMPORTANT]
> For confidential disk encryption, Microsoft Defender for Cloud doesn't currently have an applicable recommendation.

\* Microsoft Defender for Cloud has the following disk encryption recommendations:
- [Virtual machines and virtual machine scale sets should have encryption at host enabled](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffc4d8e41-e223-45ea-9bf5-eada37891d87) (detects only encryption at host).
- [Virtual machines should encrypt temp disks, caches, and data flows between Compute and Storage resources](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) (detects only Azure Disk Encryption).
- [Windows virtual machines should enable Azure Disk Encryption or EncryptionAtHost](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f3dc5edcd-002d-444c-b216-e123bbfa37c0) (detects both Azure Disk Encryption and EncryptionAtHost).
- [Linux virtual machines should enable Azure Disk Encryption or EncryptionAtHost](https://ms.portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fca88aadc-6e2b-416c-9de2-5a0f01d1693f) (detects both Azure Disk Encryption and EncryptionAtHost).


## Next steps

- [Azure Disk Encryption for Linux VMs](./linux/disk-encryption-overview.md)
- [Azure Disk Encryption for Windows VMs](./windows/disk-encryption-overview.md)
- [Server-side encryption of Azure Disk Storage](./disk-encryption.md)
- [Encryption at host](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)
- [Migrate from Azure Disk Encryption to encryption at host](./disk-encryption-migrate.md)
- [DCasv5 and ECasv5 series confidential VMs](/azure/confidential-computing/confidential-vm-overview#confidential-os-disk-encryption)
- [Azure encryption overview](/azure/security/fundamentals/encryption-overview)
