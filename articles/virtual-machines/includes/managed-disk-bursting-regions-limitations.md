---
 title: include file
 description: include file
 author: albecker1
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 01/04/2021
 ms.author: albecker1
 ms.custom: include file
# Customer intent: As a cloud administrator, I want to understand the limitations and requirements of on-demand bursting for Premium SSDs, so that I can effectively manage disk performance and ensure optimal configuration for my virtual machines.
---
- On-demand bursting cannot be enabled on a Premium SSD that has less than or equal to 512 GiB. Premium SSDs less than or equal to 512 GiB will always use credit-based bursting.
- On-demand bursting is only supported on Premium SSDs. If a Premium SSD with on-demand bursting enabled is switched to another disk type, then disk bursting is disabled.
- On-demand bursting doesn't automatically disable itself when the performance tier is changed. If you want to change your performance tier but do not want to keep disk bursting, you must disable it.
- On-demand bursting can only be enabled when the disk is detached from a VM or when the VM is stopped. On-demand bursting can be disabled 12 hours after it has been enabled.
