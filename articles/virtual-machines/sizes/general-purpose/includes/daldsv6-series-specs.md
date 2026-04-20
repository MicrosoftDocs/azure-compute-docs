---
title: Daldsv6 series specs include
description: Include file containing specifications of Daldsv6-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As an IT infrastructure planner, I want to review the specifications of Daldsv6 series VMs, so that I can determine the appropriate VM size for my organization's workload requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC 9004 (Genoa) [x86-64]                               |
| Memory         | 4 - 192 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 880 GiB <br>38,000 - 1,800,000 IOPS <br>180 - 8,640 MBps                               |
| Remote Storage | 4 - 32 Disks | 4,000 - 175,000 IOPS <br>90 - 4,320 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 2 - 8 NICs          | 12,500 - 40,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |
