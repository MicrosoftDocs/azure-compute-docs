---
title: Ddsv5 series specs include
description: Include file containing specifications of Ddsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07-29-2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a system architect planning infrastructure, I want to review the specifications of Ddsv5-series VMs, so that I can determine the most suitable size to meet my application performance and capacity requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64]                                                 |
| Memory         | 8 - 384 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 3600 GiB <br>9000 - 450000 IOPS (RR) <br>125 - 4000 MBps (RR)|
| Remote Storage | 4 - 32 Disks    | 3750 - 80000 IOPS <br>85 - 2600 MBps                     |
| Network        | 2 - 8 NICs          | 12500 - 35000 Mbps                                            |
| Accelerators   | None              |                                                     |
