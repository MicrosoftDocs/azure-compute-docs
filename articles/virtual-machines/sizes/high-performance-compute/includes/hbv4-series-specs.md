---
title: HBv4 series specs include
description: Include file containing specifications of HBv4-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/28/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect evaluating VM options, I want to access detailed specifications for the HBv4 series, so that I can determine the best configuration for my application’s performance and storage needs.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 24 - 176 vCPUs     | AMD EPYC 9V33X (Genoa-X) [x86-64] |
| L3 Cache       | 2304 MB       |    |
| Memory         | 768 GB        | 780 GB/s   |
| Local Storage  | 1 Temp Disk <br>2 NVMe Disks         |  480 GiB <br>1800 GiB  |
| Remote Storage | 32 Disks        |  |
| Network        | 8 vNICs <br> 1 InfiniBand NDR NIC       | 80 Gb/s <br> 400 Gb/s |
| Accelerators   | None            |     |
