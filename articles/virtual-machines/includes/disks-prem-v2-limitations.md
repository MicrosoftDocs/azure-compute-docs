---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 08/20/2025
 ms.author: rogarana
ms.custom:
  - include file
  - ignite-2023
# Customer intent: "As a cloud architect, I want to understand the limitations of Premium SSD v2 disks, so that I can effectively plan my virtual machine configurations and ensure compatibility with Azure services."
---
- Premium SSD v2 disks can't be used as an OS disk or with [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery).
- For regions that support availability zones, Premium SSD v2 disks can only be attached to zonal VMs. When creating a new VM, specify the availability zone you want before adding Premium SSD v2 disks to your configuration. 
- Premium SSD v2 doesn't support host caching.
