---
title: HBv2 series specs include
description: Include file containing specifications of HBv2-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/28/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of HBv2-series VMs so that I can determine which VM size best meets the performance and resource requirements for my applications."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 16 - 120 vCPUs     | AMD EPYC 7V12 (Rome) [x86-64] |
| L3 Cache       | 1536 MB       |  |             
| Memory         | 456 GB        | 350 GB/s   |
| Local Storage  | 1 Temp Disk <br> 1 NVMe Disk         | 480 GiB  <br>960 GiB  |
| Remote Storage | 8 Disks        |  |
| Network        | 8 vNICs <br> 1 InfiniBand HDR NIC       | 40 Gb/s <br> 200 Gb/s |
| Accelerators   | None            |     |
