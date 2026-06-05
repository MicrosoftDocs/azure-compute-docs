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
    - A [small subset of regions](#regional-availability) that support availability zones support [nonzonal](/azure/reliability/availability-zones-zonal-resource-resiliency#resource-deployment-types) deployments of Premium SSD v2 disks, which have [extra limitations](#nonzonal-premium-ssd-v2-deployments).