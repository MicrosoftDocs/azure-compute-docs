---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 06/05/2025
 ms.author: rogarana
ms.custom:
  - include file
  - ignite-2023
---
- Can't be enabled on virtual machines (VMs) or virtual machine scale sets that currently or ever had Azure Disk Encryption enabled. 
- Azure Disk Encryption can't be enabled on disks that have encryption at host enabled.
- The encryption can be enabled on existing virtual machine scale sets. However, only new VMs created after enabling the encryption are automatically encrypted.
- Existing VMs must be deallocated and reallocated in order to be encrypted.

The following restrictions only apply to Ultra Disks and Premium SSD v2:
- Disks using a 512e sector size need to have been created after 5/13/2023.
    - If your disk was created before this date, [snapshot your disk](/azure/virtual-machines/disks-incremental-snapshots), and create a new disk using the snapshot.