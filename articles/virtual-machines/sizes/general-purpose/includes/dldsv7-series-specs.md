---
title: Dldsv7 series specs include
description: Include file containing specifications of Dldsv7-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/10/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dldsv7-series VMs, so that I can determine which sizes meet the performance and resource needs of my applications."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 248 vCPUs       |Intel Xeon 6 6973PC (Granite Rapids) [x86-64]       |
| Memory         | 4 - 496 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 1,760 GiB <br>50,000 - 6,400,000 IOPS <br>280 - 35,840 MBps                               |
| Remote Storage | 10 - 64 Disks | 4,000 - 397,000 IOPS <br>115 - 13,145 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 3 - 15 NICs          | 16,000 - 150,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |

