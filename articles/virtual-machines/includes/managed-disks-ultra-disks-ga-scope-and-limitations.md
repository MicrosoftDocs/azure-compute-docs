---
title: include file
description: include file
author: roygara
ms.service: azure-disk-storage
ms.topic: include
ms.date: 12/23/2024
ms.author: rogarana
ms.custom: include file
---

The following list contains Ultra Disk's limitations:
- Ultra Disks can't be used as an OS disk.
- Ultra Disks can't be used with Azure Compute Gallery.
- Currently, Ultra Disks only support Single VM and Availability zone infrastructure options.
- Ultra Disks don't support availability sets.
- Existing disks currently can't change their type to an Ultra Disk. They must be [migrated](/azure/virtual-machines/disks-convert-types?tabs=azure-powershell#migrate-to-premium-ssd-v2-or-ultra-disk-using-snapshots).
- Encrypting Ultra Disks with customer-managed keys using Azure Key Vaults stored in a different Microsoft Entra ID tenant isn't currently supported.
- Azure Disk Encryption isn't supported for VMs with Ultra Disks. Instead, you should use encryption at rest with platform-managed or customer-managed keys.
- Azure Site Recovery isn't supported for VMs with Ultra Disks.
- Ultra Disks don't support disk caching.
- Snapshots are supported with [other limitations](/azure/virtual-machines/disks-incremental-snapshots?tabs=azure-powershell#incremental-snapshots-of-premium-ssd-v2-and-ultra-disks).
- Azure Backup support for VMs with Ultra Disks is [generally available](/azure/backup/backup-support-matrix-iaas#vm-storage-support). Azure Backup has limitations when using Ultra Disks, see [VM storage support](/azure/backup/backup-support-matrix-iaas#vm-storage-support) for details.

Ultra Disks support a 4k physical sector size by default but also supports a 512E sector size. Most applications are compatible with 4k sector sizes, but some require 512-byte sector sizes. Oracle Database, for example, requires release 12.2 or later in order to support 4k native disks. For older versions of Oracle DB, 512-byte sector size is required.

The following table outlines the regions Ultra Disks are available in, and their corresponding availability options.

> [!NOTE]
> If a region in the following list lacks availability zones that support Ultra disks, then a VM in that region must be deployed without infrastructure redundancy to attach an Ultra Disk.

| Redundancy options | Regions |
|--------------------|---------|
| **Regional** | Australia Central<br/>Brazil Southeast<br/>Canada East<br/>Korea South<br/>UK West <br/>North Central US, West US<br/>US Gov Arizona, US Gov Texas |
| **One availability zone** | Brazil South<br/>Central India<br/>East Asia<br/>Germany West Central<br/>Korea Central<br/>South Central US<br/>US Gov Virginia |
| **Two availability zones** |  Qatar Central |
| **Three availability zones** | Australia East<br/>Canada Central<br/>China North 3<br/>North Europe, West Europe<br/>France Central</br>Italy North<br/>Japan East<br/>Poland Central<br/>South Africa North</br>Southeast Asia<br/>Sweden Central<br/>Switzerland North<br/>UAE North</br>UK South<br/>Central US, East US, East US 2, West US 2, West US 3 |

Not every VM size is available in every supported region with Ultra Disks. The following table lists VM series that are compatible with Ultra Disks.

|VM Type     |Sizes    |Description  |
|------------|---------|-------------|
| General purpose|[DSv3-series](/azure/virtual-machines/dv3-dsv3-series#dsv3-series), [Ddsv4-series](/azure/virtual-machines/ddv4-ddsv4-series#ddsv4-series), [Dsv4-series](/azure/virtual-machines/dv4-dsv4-series#dsv4-series), [Dasv4-series](/azure/virtual-machines/dav4-dasv4-series#dasv4-series), [Dsv5-series](/azure/virtual-machines/dv5-dsv5-series#dsv5-series), [Ddsv5-series](/azure/virtual-machines/ddv5-ddsv5-series#ddsv5-series), [Dasv5-series](/azure/virtual-machines/dasv5-dadsv5-series#dasv5-series)| Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.|
| Compute optimized|[FSv2-series](/azure/virtual-machines/fsv2-series)| High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.|
| Memory optimized|[ESv3-series](/azure/virtual-machines/ev3-esv3-series#esv3-series), [Easv4-series](/azure/virtual-machines/eav4-easv4-series#easv4-series), [Edsv4-series](/azure/virtual-machines/edv4-edsv4-series#edsv4-series), [Esv4-series](/azure/virtual-machines/ev4-esv4-series#esv4-series), [Esv5-series](/azure/virtual-machines/ev5-esv5-series#esv5-series), [Edsv5-series](/azure/virtual-machines/edv5-edsv5-series#edsv5-series), [Easv5-series](/azure/virtual-machines/easv5-eadsv5-series#easv5-series), [Ebsv5 series](/azure/virtual-machines/ebdsv5-ebsv5-series#ebsv5-series), [Ebdsv5 series](/azure/virtual-machines/ebdsv5-ebsv5-series#ebdsv5-series), [M-series](/azure/virtual-machines/m-series), [Mv2-series](/azure/virtual-machines/mv2-series), [Msv2/Mdsv2-series](/azure/virtual-machines/msv2-mdsv2-series)|High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.
| Storage optimized|[LSv2-series](/azure/virtual-machines/lsv2-series), [Lsv3-series](/azure/virtual-machines/lsv3-series), [Lasv3-series](/azure/virtual-machines/lasv3-series)|High disk throughput and IO ideal for Big Data, SQL, NoSQL databases, data warehousing, and large transactional databases.|
| GPU optimized|[NCv2-series](/azure/virtual-machines/ncv2-series), [NCv3-series](/azure/virtual-machines/ncv3-series), [NCasT4_v3-series](/azure/virtual-machines/nct4-v3-series), [ND-series](/azure/virtual-machines/nd-series), [NDv2-series](/azure/virtual-machines/ndv2-series), [NVv3-series](/azure/virtual-machines/nvv3-series), [NVv4-series](/azure/virtual-machines/nvv4-series), [NVadsA10 v5-series](/azure/virtual-machines/nva10v5-series)| Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs.|
| <nobr>Performance optimized</nobr> |[HB-series](/azure/virtual-machines/hb-series), [HC-series](/azure/virtual-machines/hc-series), [HBv2-series](/azure/virtual-machines/hbv2-series)|The fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).|
