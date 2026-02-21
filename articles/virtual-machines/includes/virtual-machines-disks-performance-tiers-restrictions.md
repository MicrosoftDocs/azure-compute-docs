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
# Customer intent: As a cloud administrator, I want to change the performance tier of managed disks for virtual machines, so that I can optimize storage performance based on workload requirements without disrupting running instances.
---

- Changing the performance tier is currently only supported for Premium SSD managed disks.
- Performance tiers of shared disks can't be changed while attached to running VMs.
    - To change the performance tier of a shared disk, stop all the VMs the disk is attached to.
- Only disks larger than 4,096 GiB can use the P60, P70, and P80 performance tiers.
- A disk's performance tier can be downgraded only once every 12 hours.
- The system doesn't return `Performance Tier` for disks created before June 2020. You can take advantage of `Performance Tier` for an older disk by updating it with the baseline Tier.
- You can't set a disk's performance tier to a tier below its baseline tier.


> [!CAUTION]
> If you use [Terraform](https://www.terraform.io/) to change the performance tier while a VM is running, Terraform shuts down your VM and attempts to restart it after the process is completed. Changing the performance tier with other methods avoids this.

