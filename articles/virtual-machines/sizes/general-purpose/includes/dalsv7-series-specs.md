---
title: Dalsv7 series specs include
description: Include file containing specifications of Dalsv7-series VM sizes.
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dalsv7 series VM sizes, so that I can determine the appropriate configuration for my application workload requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 160 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                               |
| Memory         | 4 - 320 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 10 - 64 Disks | 4,000 - 310,000 IOPS <br>118 - 10,356 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 2 - 15 NICs          | 16,000 - 80,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |
