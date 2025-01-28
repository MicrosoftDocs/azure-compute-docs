---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 11/08/2024
 ms.author: rogarana
ms.custom:
  - include file
  - ignite-2023
---
- Premium SSD v2 disks can't be used as an OS disk.
- Premium SSD v2 disks can't be used with Azure Compute Gallery.
- Currently, Premium SSD v2 disks are only available in [select regions](/azure/virtual-machines/disks-deploy-premium-v2#regional-availability).
- For regions that support availability zones, Premium SSD v2 disks can only be attached to zonal VMs. When creating a new VM or Virtual Machine Scale Set, specify the availability zone you want before adding Premium SSD v2 disks to your configuration.
- Encrypting Premium SSD v2 disks with customer-managed keys using Azure Key Vaults stored in a different Microsoft Entra ID tenant isn't currently supported.
- Azure Disk Encryption (guest VM encryption via BitLocker/DM-Crypt) isn't supported for VMs with Premium SSD v2 disks. We recommend you to use encryption at rest with platform-managed or customer-managed keys, which is supported for Premium SSD v2. 
- Currently, Premium SSD v2 disks can't be attached to VMs in Availability Sets. 
- Azure Site Recovery isn't supported for VMs with Premium SSD v2 disks.
- Premium SSDv2 doesn't support host caching.
