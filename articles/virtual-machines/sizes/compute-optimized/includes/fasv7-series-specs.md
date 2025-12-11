---
title: Fasv7 series specs include
description: Include file containing specifications of Fasv7-series VM sizes.
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to access the specifications of Fasv7 series VM sizes, so that I can determine the best configuration for my application's performance and resource needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 1 - 80 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                               |
| Memory         | 4 - 320 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 10 - 64 Disks    | 4000 - 212000 IOPS <br>118 - 10344 MBps   |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 10 - 64 Disks    | 44000 - 242640 IOPS <br>1412 - 11410 MBps   |
| Network        | 2 - 15 NICs          | 16000 - 80000 Mbps                          |
| Accelerators   | None              |                                   |
