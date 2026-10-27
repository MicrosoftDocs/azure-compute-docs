---
title: Performance tiers for Azure Premium SSD managed disks
description: Learn how performance tiers for Azure Premium SSD managed disks work and which tiers are available for each disk size.
author: roygara
ms.service: azure-disk-storage
ms.topic: concept-article
ms.date: 09/11/2026
ms.author: rogarana
ms.custom: references_regions
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to understand how performance tiers for Premium SSD managed disks work, so that I can evaluate if they're a suitable option for my needs when optimizing disk performance and managing costs effectively during varying demand periods.
---

# Performance tiers for Azure Premium SSD managed disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

> [!NOTE]
> This article covers what performance tiers are, conceptually. If you want to learn how to change the performance of disks that don't use performance tiers, like Ultra Disks or Premium SSD v2, see either [Adjust the performance of an Ultra Disk](disks-enable-ultra-ssd.md#adjust-the-performance-of-an-ultra-disk) or [Adjust disk performance of a Premium SSD v2](disks-deploy-premium-v2.md#adjust-disk-performance)

When you set the provisioned size of an Azure Premium solid-state drive (SSD) managed disk, a performance tier is automatically selected based on the size you set. The performance tier determines the IOPS and throughput your managed disk has. You can change the performance tier when you deploy the disk or afterward without changing the disk size. Most performance tier changes don't require downtime. Exceptions apply to attached shared disks and changes made with Terraform, as described in [Restrictions](#restrictions).

Changing the performance tier lets you meet a temporary period of consistently higher demand without relying on disk bursting. Depending on the duration, changing tiers can be more cost-effective than bursting. Common scenarios include holiday shopping, performance testing, and training environments. When demand returns to normal, you can return the disk to its original tier.

To learn more about how the performance of a disk works with the performance of a virtual machine, see [Virtual machine and disk performance](disks-performance.md).

## Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](./includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## How it works

When you first deploy or provision a disk, the baseline performance tier for that disk is set based on the provisioned disk size. You can use a performance tier higher than the original baseline to meet higher demand. When you no longer need that performance level, you can return to the initial baseline performance tier.

### Billing impact

Disk billing changes as its performance tier changes. For example, if you provision a P10 disk (128 GiB), your baseline performance tier is set as P10 (500 IOPS and 100 MBps). Your disk is billed at the P10 rate. You can set the disk's performance tier to P50 (7,500 IOPS and 250 MBps) without increasing the disk size. While the disk's performance tier is set to P50, your disk is billed at the P50 rate. When you no longer need the higher performance, you can set the performance tier of the disk back to the P10 tier and your disk's billing will return to the P10 rate.

For billing information, see [managed disk pricing](https://azure.microsoft.com/pricing/details/managed-disks/).

## What tiers can be changed

Use the table to identify the baseline and higher performance tiers available for a Premium SSD managed disk based on its provisioned size.

| Disk size | Baseline performance tier | Can be upgraded to |
|----------------|-----|-------------------------------------|
| 4 GiB | P1 | P2, P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 8 GiB | P2 | P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 16 GiB | P3 | P4, P6, P10, P15, P20, P30, P40, P50 | 
| 32 GiB | P4 | P6, P10, P15, P20, P30, P40, P50 |
| 64 GiB | P6 | P10, P15, P20, P30, P40, P50 |
| 128 GiB | P10 | P15, P20, P30, P40, P50 |
| 256 GiB | P15 | P20, P30, P40, P50 |
| 512 GiB | P20 | P30, P40, P50 |
| 1 TiB | P30 | P40, P50 |
| 2 TiB | P40 | P50 |
| 4 TiB | P50 | None |
| 8 TiB | P60 |  P70, P80 |
| 16 TiB | P70 | P80 |
| 32 TiB | P80 | None |

## Next steps

To learn how to change your performance tier, see [Change your performance tier without downtime](disks-performance-tiers.md).