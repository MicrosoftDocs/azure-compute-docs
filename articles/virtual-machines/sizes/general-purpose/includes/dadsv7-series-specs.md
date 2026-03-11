---
title: Dadsv7 series specs include
description: Include file containing specifications of Dadsv7-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dadsv7-series VM sizes, so that I can select the appropriate virtual machine configuration for my applications' performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 160 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                               |
| Memory         | 8 - 640 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 2,200 GiB <br>38,000 - 3,000,000 IOPS <br>280 - 22,400 MBps                               |
| Remote Storage | 10 - 64 Disks | 4,000 - 310,000 IOPS <br>118 - 10,356 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 2 - 15 NICs          | 16,000 - 80,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |
