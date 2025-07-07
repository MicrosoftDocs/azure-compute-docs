---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 06/09/2025
 ms.author: rogarana
ms.custom:
  - include file
  - ignite-2023
# Customer intent: "As a cloud architect, I want to understand the limitations of Premium SSD v2 disks, so that I can effectively plan my virtual machine configurations and ensure compatibility with Azure services."
---
- Premium SSD v2 disks can't be used as an OS disk.
- Premium SSD v2 disks can't be used with Azure Compute Gallery.
- Currently, Premium SSD v2 disks are only available in [select regions](/azure/virtual-machines/disks-deploy-premium-v2#regional-availability).
- For regions that support availability zones, Premium SSD v2 disks can only be attached to zonal VMs. When creating a new VM or Virtual Machine Scale Set, specify the availability zone you want before adding Premium SSD v2 disks to your configuration.
- (Preview) You can encrypt Premium SSD v2 disks with customer-managed keys using Azure Key Vaults stored in a different Microsoft Entra ID tenant.
- Azure Disk Encryption (guest VM encryption via BitLocker/DM-Crypt) isn't supported for VMs with Premium SSD v2 disks. We recommend you to use encryption at rest with platform-managed or customer-managed keys, which is supported for Premium SSD v2. 
- Follow the instructions in [Use premium SSD v2 with VMs in availability set](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI) to deploy a Premium SSD v2 within an availability set.
- Azure Site Recovery for VMs with Premium SSD v2 disks is currently in [Public Preview](/azure/site-recovery/azure-to-azure-support-matrix).
- Premium SSD v2 doesn't support host caching.
