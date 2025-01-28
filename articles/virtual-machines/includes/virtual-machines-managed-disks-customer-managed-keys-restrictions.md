---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 11/08/2024
 ms.author: rogarana
 ms.custom: include file
---
- If this feature is enabled for a disk with incremental snapshots, it can't be disabled on that disk or its snapshots.
    To work around this, copy all the data to an entirely different managed disk that isn't using customer-managed keys. You can do that with either the [Azure CLI](/azure/virtual-machines/linux/disks-upload-vhd-to-managed-disk-cli#copy-a-managed-disk) or the [Azure PowerShell module](/azure/virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell#copy-a-managed-disk).
- A disk and all of its associated incremental snapshots must have the same disk encryption set.
- Only [software and HSM RSA keys](/azure/key-vault/keys/about-keys) of sizes 2,048-bit, 3,072-bit and 4,096-bit are supported, no other keys or sizes.
    - [HSM](/azure/key-vault/keys/hsm-protected-keys) keys require the **premium** tier of Azure Key vaults.
- For Ultra Disks and Premium SSD v2 disks only:
    - Disks created from snapshots that are encrypted with server-side encryption and customer-managed keys must be encrypted with the same disk encryption set.
    - User-assigned managed identities aren't supported for Ultra Disks and Premium SSD v2 disks encrypted with customer-managed keys.
    - Encrypting Ultra Disks and Premium SSD v2 disks with customer-managed keys using Azure Key Vaults stored in a different Microsoft Entra ID tenant isn't currently supported.
- Most resources related to your customer-managed keys (disk encryption sets, VMs, disks, and snapshots) must be in the same subscription and region.
    - Azure Key Vaults may be used from a different subscription but must be in the same region as your disk encryption set. As a preview, you can use Azure Key Vaults from [different Microsoft Entra tenants](/azure/virtual-machines/disks-cross-tenant-customer-managed-keys).
- Disks encrypted with customer-managed keys can only move to another resource group if the VM they are attached to is deallocated.
- Disks, snapshots, and images encrypted with customer-managed keys can't be moved between subscriptions.
- Managed disks currently or previously encrypted using Azure Disk Encryption can't be encrypted using customer-managed keys.
- Can only create up to 5000 disk encryption sets per region per subscription.
- For information about using customer-managed keys with shared image galleries, see [Preview: Use customer-managed keys for encrypting images](/azure/virtual-machines/image-version-encryption).
