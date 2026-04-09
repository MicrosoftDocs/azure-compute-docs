---
title: DCasv6 series specs include
description: Include file containing specifications of DCasv6-series VM sizes.
author: rakeshginjupalli
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: raginjup
ms.reviewer: raginjup
ms.custom: include file
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       |     AMD EPYC 9004 (Genoa) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 8 - 64 Disks | 4,000 - 175,000 IOPS <br>90 - 4,320 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2)|
| Network        | 2 - 8 NICs          | 12,500 - 34,000 Mbps <br>Interfaces: NetVSC, ConnectX                  |
| Accelerators   | None              |                                   |
