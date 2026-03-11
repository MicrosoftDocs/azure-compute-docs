---
title: DCasv5 series specs include
description: Include file containing specifications of DCasv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of the DCasv5 series VM sizes so that I can select the most suitable configuration for my application's performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC (Milan) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 4 - 32 Disks    | 4,000 - 80,000 IOPS <br>82 - 1,600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds)   |
| Network        | 2 - 8 NICs          | 3,000 - 20,000 Mbps <br>Interfaces: NetVSC, ConnectX                          |
| Accelerators   | None              |                                   |
