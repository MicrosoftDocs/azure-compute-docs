---
title: include file
description: include file
author: roygara
ms.service: azure-disk-storage
ms.topic: include
ms.date: 08/28/2026
ms.author: rogarana
ms.custom: include file
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to understand the limitations of per-disk resiliency so that I can plan and deploy supported scenarios.
---

Per-disk resiliency currently has the following limitations:

- Per-disk resiliency is unsupported on VMs created before August 15, 2026, in a [supported region](../disks-per-disk-resiliency.md#regions-that-support-per-disk-resiliency). Unsupported VMs continue to shut down when disk connectivity fails.
- Per-disk resiliency applies only to [data disks](../managed-disks-overview.md#data-disk). It has no effect on OS disks.
- Per-disk resiliency must be enabled either while the VM is deallocated or before you attach the disk to the VM.
- Per-disk resiliency can't be enabled while creating a VM or virtual machine scale set. Enable it on the data disk resource, and then attach the disk to the VM or scale set.
- VMs with [Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator) enabled don't support per-disk resiliency. Enabling the feature on those VMs doesn't change their behavior.
- The per-disk resiliency setting (the `actionOnDiskDelay` value) isn't preserved on VM restore points.
- Azure Backup and Azure Site Recovery don't preserve the per-disk resiliency setting. After a VM is restored by Azure Backup or Azure Site Recovery, set the property again on the restored disks.
- Currently unavailable through the Azure portal.