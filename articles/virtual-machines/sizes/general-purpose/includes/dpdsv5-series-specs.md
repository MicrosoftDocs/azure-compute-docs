---
title: Dpdsv5 series specs include
description: Include file containing specifications of Dpdsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dpdsv5 series VM sizes, so that I can determine the appropriate resources for my application's performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 64 vCPUs         | Ampere Altra [Arm64]                                        |
| Memory         | 8 - 208 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 2,400 GiB <br>9,000 - 300,000 IOPS <br>125 - 4,000 MBps|
| Remote Storage | 4 - 32 Disks | 4,000 - 80,000 IOPS <br>85 - 1,735 MBps <br>Disk Types: [Premium SSD](../../../disks-types.md#premium-ssds) |
| Network        | 2 - 8 NICs          | 12,500 - 40,000 Mbps <br>Interfaces: NetVSC, ConnectX                                            |
| Accelerators   | None              |                                                     |
