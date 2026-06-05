---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 06/05/2026
 ms.author: rogarana
ms.custom:
  - include file
  - ignite-2023
# Customer intent: "As a cloud architect, I want to understand the limitations of Premium SSD v2 disks, so that I can effectively plan my virtual machine configurations and ensure compatibility with Azure services."
---
- Premium SSD v2 disks can't be used as an OS disk or with [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery).
- Premium SSD v2 doesn't support host caching.
- In most regions that support availability zones, you can only attach Premium SSD v2 disks to zonal VMs. When you create a new VM, specify the availability zone you want before adding Premium SSD v2 disks to your configuration.
    - A [small subset of regions](#regional-availability) that support availability zones support [nonzonal](/azure/reliability/availability-zones-zonal-resource-resiliency#resource-deployment-types) deployments of Premium SSD v2 disks, which have extra limitations.

### Nonzonal Premium SSD v2 deployments

The following restrictions only apply when you deploy a nonzonal Premium SSD v2. They apply because Azure selects an availability zone for you in the backend. The availability zone might not be the same as the availability zone of the virtual machine (VM) associated with the disk. When this happens, Azure performs a background copy to move the disk to the VM's availability zone for zone alignment and improved latency between the VM and disk.

- Only one background data copy can run per disk at a time.
- During the background copy, if you attempt to detach and reattach the disk, the operation fails.
- Nonzonal disks should only be attached to running nonzonal VMs. If you attach a nonzonal disk to a stopped or deallocated VM, it can cause the VM restart to fail if a different background copy operation is already in progress, and a restart triggers another background copy to ensure the availability zones are aligned.
- You can't attach a disk created from a snapshot to a nonzonal VM while its own background copy is in progress, even if the snapshot is an [instant access snapshot](/azure/virtual-machines/disks-instant-access-snapshots). To check the status of your snapshot's background copy process, see [Check snapshot status](/azure/virtual-machines/disks-incremental-snapshots?tabs=azure-cli#check-snapshot-status).
- You can't increase the size of a disk or change its customer-managed key while a background data copy for availability zone alignment is occurring.