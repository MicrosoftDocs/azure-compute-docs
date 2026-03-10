---
title: Dpldsv5 series specs include
description: Include file containing specifications of Dpldsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dpldsv5 series VMs, so that I can select the appropriate instance sizes for my workloads based on performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 64 vCPUs       | Ampere Altra [Arm64]                                               |
| Memory         | 4 - 128 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 2,400 GiB <br>9,375 - 300,000 IOPS (RR) <br>125 - 4,000 MBps (RR)|
| Remote Storage <br />([Premium SSD](../../../disks-types.md#premium-ssds)) | 4 - 32 Disks    | 3,750 - 80,000 IOPS <br>85 - 1,735 MBps                     |
| Network        | 2 - 8 NICs          | 12,500 - 40,000 Mbps <br>Interfaces: NetVSC, ConnectX                                            |
| Accelerators   | None              |                                                     |
