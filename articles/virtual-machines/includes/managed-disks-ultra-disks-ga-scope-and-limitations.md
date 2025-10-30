---
title: include file
description: include file
author: roygara
ms.service: azure-disk-storage
ms.topic: include
ms.date: 10/23/2025
ms.author: rogarana
ms.custom: include file
# Customer intent: As a cloud architect, I want to understand the limitations and requirements of Ultra Disks so that I can evaluate their suitability for my virtual machine deployments and ensure compliance with application needs and infrastructure capabilities.
---

The following list contains Ultra Disk's limitations:
- Only during deployment with Virtual Machine Scale Sets with Uniform orchestration mode: At time of deployment, Ultra Disks used with Uniform Virtual Machine Scale Sets use different formulas for IOPS and throughput for their performance. Exceeding the limits of these formulas results in a deployment failure. Once the deployment succeeds you can [increase the performance](../disks-enable-ultra-ssd.md#adjust-the-performance-of-an-ultra-disk) of these disks. See [Ultra Disk IOPS](../disks-types.md#ultra-disk-iops) and [Ultra Disk throughput](../disks-types.md#ultra-disk-throughput) for details. 
- Ultra Disks can't be used as an OS disk or with Azure Compute Gallery.
- Currently, Ultra Disks only support Single VM and Availability zone infrastructure options as locally redundant storage (LRS). Ultra Disks don't support zone redundant storage (ZRS).
- Ultra Disks don't support availability sets.
- Ultra Disks don't support disk caching.
- (Preview) You can encrypt Ultra Disks with customer-managed keys using Azure Key Vaults stored in a different Microsoft Entra ID tenant.

Ultra Disks support a 4k physical sector size by default but also supports a 512E sector size. Most applications are compatible with 4k sector sizes, but some require 512-byte sector sizes. Oracle Database, for example, requires release 12.2 or later in order to support 4k native disks. For older versions of Oracle DB, 512-byte sector size is required.

The following table outlines the regions Ultra Disks are available in, and their corresponding availability options.

> [!NOTE]
> If a region in the following list lacks availability zones that support Ultra Disks, then a VM in that region must be deployed without infrastructure redundancy to attach an Ultra Disk.

| Redundancy options | Regions |
|--------------------|---------|
| **Regional** | Australia Central, Australia Central 2<br/>Brazil Southeast<br/>Canada East<br/>Korea South<br/>Norway West<br/>UK West<br/>North Central US, West US<br/>US Gov Arizona, US Gov Texas |
| **One availability zone** | Brazil South<br/>Central India<br/>East Asia<br/>Germany West Central<br/>Korea Central<br/>South Central US<br/>Spain Central<br/>US Gov Virginia |
| **Two availability zones** | Indonesia Central<br/>Malaysia West<br/>New Zealand North<br/>Qatar Central |
| **Three availability zones** | Australia East<br/>Canada Central<br/>China North 3<br/>North Europe, West Europe<br/>France Central<br/>Italy North<br/>Japan East<br/>Poland Central<br/>South Africa North<br/>Southeast Asia<br/>Sweden Central<br/>Switzerland North<br/>UAE North<br/>UK South<br/>Central US, East US, East US 2, West US 2, West US 3 |

Not every VM size is available in every supported region with Ultra Disks. The following table lists VM series that are compatible with Ultra Disks.

|VM Type     |Sizes    |Description  |
|------------|---------|-------------|
| General purpose|[DSv3-series](/azure/virtual-machines/sizes/general-purpose/dsv3-series?tabs=sizebasic), [Ddsv4-series](/azure/virtual-machines/sizes/general-purpose/ddsv4-series?tabs=sizestorageremote), [Dsv4-series](/azure/virtual-machines/sizes/general-purpose/dsv4-series?tabs=sizebasic), [Dasv4-series](/azure/virtual-machines/sizes/general-purpose/dasv4-series?tabs=sizestorageremote), [Dsv5-series](/azure/virtual-machines/sizes/general-purpose/dsv5-series?tabs=sizestorageremote), [Ddsv5-series](/azure/virtual-machines/sizes/general-purpose/ddsv5-series?tabs=sizestorageremote), [Dasv5-series](/azure/virtual-machines/sizes/general-purpose/dasv5-series?tabs=sizestorageremote), [Dplsv6-series](/azure/virtual-machines/sizes/general-purpose/dplsv6-series?tabs=sizestorageremote), [Dpldsv6-series](/azure/virtual-machines/sizes/general-purpose/dpldsv6-series?tabs=sizestorageremote), [Dpsv6-series](/azure/virtual-machines/sizes/general-purpose/dpsv6-series?tabs=sizestorageremote), [Dpdsv6-series](/azure/virtual-machines/sizes/general-purpose/dpdsv6-series?tabs=sizestorageremote)| Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.|
| Compute optimized|[FSv2-series](/azure/virtual-machines/fsv2-series), [Famsv6-series](/azure/virtual-machines/sizes/compute-optimized/famsv6-series?tabs=sizestorageremote)| High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.|
| Memory optimized|[ESv3-series](/azure/virtual-machines/ev3-esv3-series#esv3-series), [Easv4-series](/azure/virtual-machines/sizes/memory-optimized/easv4-series?tabs=sizestorageremote), [Edsv4-series](/azure/virtual-machines/sizes/memory-optimized/edsv4-series?tabs=sizestorageremote), [Esv4-series](/azure/virtual-machines/sizes/memory-optimized/esv4-series?tabs=sizestorageremote), [Esv5-series](/azure/virtual-machines/sizes/memory-optimized/esv5-series?tabs=sizestorageremote), [Edsv5-series](/azure/virtual-machines/sizes/memory-optimized/edsv5-series?tabs=sizestorageremote), [Easv5-series](/azure/virtual-machines/easv5-eadsv5-series#easv5-series), [Ebsv5 series](/azure/virtual-machines/ebdsv5-ebsv5-series#ebsv5-series), [Ebdsv5 series](/azure/virtual-machines/ebdsv5-ebsv5-series#ebdsv5-series), [M-series](/azure/virtual-machines/sizes/memory-optimized/m-series?tabs=sizestorageremote), [Mv2-series](/azure/virtual-machines/sizes/memory-optimized/mv2-series?tabs=sizestorageremote), [Msv2](/azure/virtual-machines/sizes/memory-optimized/msv2-mm-series?tabs=sizestorageremote), [Mdsv2-series](/azure/virtual-machines/sizes/memory-optimized/mdsv2-mm-series?tabs=sizestorageremote), [Mbsv3](/azure/virtual-machines/sizes/memory-optimized/mbsv3-series?tabs=sizestorageremote), [Mbdsv3-series](/azure/virtual-machines/sizes/memory-optimized/mbdsv3-series?tabs=sizestorageremote), [Epsv6-series](/azure/virtual-machines/sizes/memory-optimized/epsv6-series?tabs=sizebasic), [Epdsv6-series](/azure/virtual-machines/sizes/memory-optimized/epdsv6-series?tabs=sizebasic) |High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.
| Storage optimized|[LSv2-series](/azure/virtual-machines/lsv2-series), [Lsv3-series](/azure/virtual-machines/lsv3-series), [Lasv3-series](/azure/virtual-machines/lasv3-series)|High disk throughput and IO ideal for Big Data, SQL, NoSQL databases, data warehousing, and large transactional databases.|
| GPU optimized| [NCv3-series](/azure/virtual-machines/ncv3-series), [NCasT4_v3-series](/azure/virtual-machines/nct4-v3-series), [ND-series](/azure/virtual-machines/nd-series), [NDv2-series](/azure/virtual-machines/ndv2-series), [NVv3-series](/azure/virtual-machines/nvv3-series), [NVv4-series](/azure/virtual-machines/nvv4-series), [NVadsA10 v5-series](/azure/virtual-machines/nva10v5-series)| Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs.|
| <nobr>Performance optimized</nobr> | [HC-series](/azure/virtual-machines/hc-series), [HBv2-series](/azure/virtual-machines/hbv2-series)|The fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).|
