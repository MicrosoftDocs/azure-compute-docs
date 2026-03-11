---
title: Dlsv6 series specs include
description: Include file containing specifications of Dlsv6-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to review the specifications of Dlsv6 series VM sizes, so that I can assess their suitability for my project's performance and resource requirements.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 128 vCPUs       | Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]                               |
| Memory         | 4 - 256 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 8 - 64 Disks | 4,000 - 267,000 IOPS <br>106 - 7,935 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 2 - 8 NICs          | 12,500 - 54,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |