---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 04/14/2023
 ms.author: rogarana
 ms.custom: include file
# Customer intent: As a cloud architect, I want to understand the limitations of shared disks across different disk types, so that I can effectively design and implement scalable virtual machine solutions in the cloud environment.
---

### General limitations

Shared disks have general limitations that apply to all shared disks, regardless of disk type. They also have more limitations that only apply to specific types of shared disks. The following list is the list of general limitations:

- Currently, only Ultra Disks, Premium SSD v2, Premium SSD, and Standard SSDs can be used as a shared disk
- Shared disks can be attached to individual Virtual Machine Scale Sets but can't be defined in the Virtual Machine Scale Set models or automatically deployed
- A shared disk can't be expanded without either deallocating all VMs the disk is attached to, or detaching the disk from all of these VMs
- Write accelerator isn't supported for shared disks
- Host caching isn't supported for shared disks

Each managed disk that has shared disks enabled are also subject to the following limitations, organized by disk type:

### Ultra disks

Ultra disks have their own separate list of limitations, unrelated to shared disks. For ultra disk limitations, refer to [Using Azure ultra disks](/azure/virtual-machines/disks-enable-ultra-ssd).

When sharing ultra disks, they have the following additional limitations:

- Only basic disks can be used with some versions of Windows Server Failover Cluster, for details see [Failover clustering hardware requirements and storage options](/windows-server/failover-clustering/clustering-requirements).
- Can't be shared across availability zones.


### Premium SSD v2

Premium SSD v2 managed disks have their own separate list of limitations, unrelated to shared disks. For these limitations, see [Premium SSD v2 limitations](/azure/virtual-machines/disks-types#premium-ssd-v2-limitations).

When sharing Premium SSD v2 disks, they have the following additional limitation:

- Only basic disks can be used with some versions of Windows Server Failover Cluster, for details see [Failover clustering hardware requirements and storage options](/windows-server/failover-clustering/clustering-requirements).
- Can't be shared across availability zones.

### Premium SSD

- Can only be enabled on data disks, not OS disks.
- Host caching isn't available for premium SSD disks with `maxShares>1`.
- Disk bursting isn't available for premium SSD disks with `maxShares>1`.
- When using Availability sets or Virtual Machine Scale Sets with Azure shared disks, [storage fault domain alignment](/azure/virtual-machines/availability) with virtual machine fault domain isn't enforced for the shared data disk.
- When using [proximity placement groups (PPG)](/azure/virtual-machines/windows/proximity-placement-groups), all virtual machines sharing a disk must be part of the same PPG.
- Only basic disks can be used with some versions of Windows Server Failover Cluster, for details see [Failover clustering hardware requirements and storage options](/windows-server/failover-clustering/clustering-requirements).
- Azure Site Recovery support isn't yet available.
- Azure Backup is available through [Azure Disk Backup](/azure/backup/disk-backup-overview).
- Only [server-side encryption](/azure/virtual-machines/disk-encryption) is supported, [Azure Disk Encryption](/azure/virtual-machines/disk-encryption-overview) isn't currently supported.
- Can only be shared across availability zones if using [Zone-redundant storage for managed disks](/azure/virtual-machines/disks-redundancy#zone-redundant-storage-for-managed-disks).

### Standard SSDs

- Can only be enabled on data disks, not OS disks.
- Host caching isn't available for standard SSDs with `maxShares>1`.
- When using Availability sets and Virtual Machine Scale Sets with Azure shared disks, [storage fault domain alignment](/azure/virtual-machines/availability) with virtual machine fault domain isn't enforced for the shared data disk.
- When using [proximity placement groups (PPG)](/azure/virtual-machines/windows/proximity-placement-groups), all virtual machines sharing a disk must be part of the same PPG.
- Only basic disks can be used with some versions of Windows Server Failover Cluster, for details see [Failover clustering hardware requirements and storage options](/windows-server/failover-clustering/clustering-requirements).
- Azure Site Recovery support isn't yet available.
- Azure Backup is available through [Azure Disk Backup](/azure/backup/disk-backup-overview).
- Only [server-side encryption](/azure/virtual-machines/disk-encryption) is supported, [Azure Disk Encryption](/azure/virtual-machines/disk-encryption-overview) isn't currently supported.
- Can only be shared across availability zones if using [Zone-redundant storage for managed disks](/azure/virtual-machines/disks-redundancy#zone-redundant-storage-for-managed-disks).
