---
title: Dpdsv5 series specs include
description: Include file containing specifications of Dpdsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07-28-2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dpdsv5 series VM sizes, so that I can determine the appropriate resources for my application's performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 64 vCPUs         | Ampere Altra [Arm64]                                        |
| Memory         | 8 - 208 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 2400 GiB <br>9375 - 300000 IOPS (RR) <br>125 - 4000 MBps (RR)|
| Remote Storage | 4 - 32 Disks    | 3750 - 80000 IOPS <br>85 - 1735 MBps                     |
| Network        | 2 - 8 NICs          | 12500 - 40000 Mbps                                            |
| Accelerators   | None              |                                                     |
