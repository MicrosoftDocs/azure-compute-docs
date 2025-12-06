---
title: Fadsv7 series specs include
description: Include file containing specifications of Fadsv7-series VM sizes.
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Fadsv7-series VM sizes, so that I can select the appropriate virtual machine configuration for my applications' performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 1 - 80 cores       | AMD EPYC 9005 (Turin) [x86-64]                               |
| Memory         | 4 - 320 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 1760 GiB <br>37500 - 3000000 IOPS (RR) <br>180 - 14400 MBps (RR)                               |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 10 - 64 Disks    | 4000 - 212000 IOPS <br>118 - 10344 MBps   |
| Remote Storage <br /> <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 10 - 64 Disks    | 44000 - 242640 IOPS <br>1412 - 11410 MBps   |
| Network        | 2 - 15 NICs          | 16000 - 80000 Mbps                          |
| Accelerators   | None              |                                   |
