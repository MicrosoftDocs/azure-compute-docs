---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 03/04/2024
 ms.author: rogarana
 ms.custom: include file
# Customer intent: As a cloud administrator, I want to change the performance tier of my Azure managed disks, so that I can optimize IOPS and throughput based on workload needs without incurring downtime.
---

> [!NOTE]
> This article focuses on how to change performance tiers. To learn how to change the performance of disks that don't use performance tiers, like Ultra Disks or Premium SSD v2, see either [Adjust the performance of an ultra disk](/azure/virtual-machines/disks-enable-ultra-ssd?tabs=azure-portal#adjust-the-performance-of-an-ultra-disk) or [Adjust disk performance of a Premium SSD v2](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli#adjust-disk-performance)

The performance of your Azure managed disk is set when you create your disk, in the form of its performance tier. The performance tier determines the IOPS and throughput your managed disk has. When you set the provisioned size of your disk, a performance tier is automatically selected. The performance tier can be changed at deployment or afterwards, without changing the size of the disk and without downtime. To learn more about performance tiers, see [Performance tiers for managed disks](/azure/virtual-machines/disks-change-performance).

Changing your performance tier has billing implications. See [Billing impact](/azure/virtual-machines/disks-change-performance#billing-impact) for details.