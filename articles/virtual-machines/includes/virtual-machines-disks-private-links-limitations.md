---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 11/04/2025
 ms.author: rogarana
 ms.custom: include file
# Customer intent: "As a cloud engineer, I want to understand the limitations of disk and snapshot import/export operations, so that I can effectively manage disk access and encryption settings within my virtual machine environment."
---

- You can't import or export more than 100 disks or snapshots at the same time with the same disk access resource
- You can't upload to a disk with both a disk access resource and a disk encryption set
- In addition to the scale targets that apply to individual disks, disk access resources have more scale targets that are centered around ingress/egress of data. These limits apply cumulatively to all disks associated with a disk access resource. For details see [here](/azure/storage/common/scalability-targets-standard-account#scale-targets-for-standard-storage-accounts-and-disk-access-resources)
