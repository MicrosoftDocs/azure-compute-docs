---
title: HBv3 series specs include
description: Include file containing specifications of HBv3-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to review the specifications of HBv3-series VM sizes, so that I can select the appropriate virtual machine configuration for my workloads.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 16 - 120 vCPUs     | AMD EPYC 7V73X (Milan-X) [x86-64] |
| L3 Cache       | 1536 MB       |    |
| Memory         | 448 GB        | 350 GB/s   |
| Local Storage  | 1 Temp Disk <br> 2 NVMe Disks         | 480 GiB  <br> 960 GiB  |
| Remote Storage | 32 Disks      |    |
| Network        | 8 vNICs <br> 1 InfiniBand HDR NIC        | 40 Gb/s <br> 200 Gb/s |
| Accelerators   | None          |    |
