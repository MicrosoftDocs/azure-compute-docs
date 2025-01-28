---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 01/16/2025
 ms.author: rogarana
 ms.custom: include file
---

- Changing the performance tier is currently only supported for Premium SSD managed disks.
- Performance tiers of shared disks can't be changed while attached to running VMs.
    - To change the performance tier of a shared disk, stop all the VMs the disk is attached to.
- Only disks larger than 4,096 GiB can use the P60, P70, and P80 performance tiers.
- A disk's performance tier can be downgraded only once every 12 hours.
- The system doesn't return `Performance Tier` for disks created before June 2020. You can take advantage of `Performance Tier` for an older disk by updating it with the baseline Tier.
