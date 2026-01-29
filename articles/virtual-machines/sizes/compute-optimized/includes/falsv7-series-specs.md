---
title: Falsv7 series specs include
description: Include file containing specifications of Falsv7-series VM sizes.
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Falsv7-series VM sizes, so that I can select the optimal configuration for my application's performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 1 - 80 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                               |
| Memory         | 2 - 160 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 10 - 64 Disks    | 4000 - 212000 IOPS <br>118 - 10344 MBps   |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 10 - 64 Disks    | 4400 - 310000 IOPS <br>136 - 10356 MBps   |
| Network        | 2 - 15 NICs          | 16000 - 80000 Mbps                          |
| Accelerators   | None              |                                   |
