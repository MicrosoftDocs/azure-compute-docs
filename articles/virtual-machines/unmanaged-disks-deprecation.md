---
title: We're retiring Azure unmanaged disks by March 31, 2026
description: This article provides a high-level overview of the retirement of Azure unmanaged disks and how to migrate to Azure managed disks.
author: roygara
ms.service: azure-disk-storage
ms.topic: concept-article
ms.date: 08/28/2025
ms.author: rogarana
# Customer intent: As an Azure user managing virtual machines, I want to migrate my unmanaged disks to managed disks before the retirement deadline, so that I can ensure continued operation and take advantage of the enhanced reliability and features of managed disks.
---

# Migrate your Azure unmanaged disks by March 31, 2026

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs

In 2017, we launched [Azure managed disks](https://azure.microsoft.com/blog/announcing-general-availability-of-managed-disks-and-larger-scale-sets/). We've been enhancing capabilities ever since. Because Azure managed disks now have the full capabilities of unmanaged disks and other advancements, we announced the retirement for unmanaged disks on September 13, 2022. Unmanaged disks will be fully retired on March 31, 2026 (extended from previously published date of September 30, 2025).

By using managed disks, you don't need to manage storage accounts for creating a disk, because Azure manages the storage accounts under the hood. This abstraction reduces maintenance overhead for you. Also, it allows managed disks to provide numerous benefits over unmanaged disks, such as [high availability](disks-high-availability.md), better [scalability](disks-scalability-targets.md), large disks, [bursting](disk-bursting.md), and [shared disks](disks-shared-enable.md). If you use unmanaged disks, start planning your [Windows](windows/convert-unmanaged-to-managed-disks.md) or [Linux](linux/convert-unmanaged-to-managed-disks.md) migration now. Complete the migration by March 31, 2026, and take advantage of Azure managed disks.

## How am I affected?

After March 31, 2026, you can't start IaaS VMs that use unmanaged disks. Any VMs that use unmanaged disks and are running or allocated will be stopped and deallocated.

## What is being retired?

Unmanaged disks are a type of page blob in Azure that is used for storing Virtual Hard Disk (VHD) files associated with virtual machines (VM). When a page blob VHD is attached to a VM, it functions as a virtual disk for that VM. The VM's operating system can read from and write to the attached page blob as if it were a SCSI volume. This retirement only affects page blobs being used as virtual disks that are directly attached to VMs.

Page blobs accessed directly via HTTP or HTTPS REST APIs are standalone entities and have no dependencies on any specific VM. Clients can interact with these page blobs by using standard HTTP or HTTPS protocols, making requests to read from or write to the blobs by using Storage REST APIs. Since these page blobs aren't attached as virtual disks, this retirement doesn't affect them.

Third party storage offerings on Azure that use page blobs via HTTP or HTTPS REST APIs as their underlying storage solution might not be affected by this retirement.

## What actions should I take?

Start planning your migration to Azure managed disks today.

- Make a list of all affected VMs:
   - The VMs with **Uses managed disks** set to **No** on the [Azure portal's VM pane](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ClassicCompute%2FVirtualMachines) are all the affected VMs within the subscription.
   - You can also query Azure Resource Graph by using the [portal](https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/resources%0A%7C%20where%20type%20%3D%3D%20%22microsoft.classiccompute%2Fvirtualmachines%22) or [PowerShell](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/governance/resource-graph/concepts/work-with-data.md) to view the list of all flagged VMs and related information for the selected subscriptions.
   - On February 28, 2020, we sent out emails to subscription owners with a list of all subscriptions that contain these VMs. Use them to build this list.
- Now that you have a list of all the affected VMs, see [Migrate Azure VMs to Managed Disks in Azure](windows/migrate-to-managed-disks.md) to learn how to migrate your unmanaged disks to managed disks. If you have other questions, see [Frequently asked questions about migrating to managed disks](faq-for-disks.yml).

- If you have further technical questions, problems, and need help with adding subscriptions to the allow list, [contact support](https://portal.azure.com/#create/Microsoft.Support/Parameters/%7B%22pesId%22:%226f16735c-b0ae-b275-ad3a-03479cfa1396%22,%22supportTopicId%22:%228a82f77d-c3ab-7b08-d915-776b4ff64ff4%22%7D).

Complete your migration as soon as possible to prevent business impact and to take advantage of the improved reliability, scalability, security, and new features of Azure managed disks.

## What resources are available for this migration?

- [Microsoft Q&A](/answers/topics/azure-virtual-machines-migration.html): Microsoft and community support for migration.
- [Azure Migration Support](https://portal.azure.com/#create/Microsoft.Support/Parameters/%7B%22pesId%22:%226f16735c-b0ae-b275-ad3a-03479cfa1396%22,%22supportTopicId%22:%221135e3d0-20e2-aec5-4ef0-55fd3dae2d58%22%7D): Dedicated support team for technical assistance during migration.

- If your company or organization has partnered with Microsoft or works with Microsoft representatives such as cloud solution architects (CSAs) or technical account managers (TAMs), work with them for more resources for migration.
