---
title: Dadsv5 series specs include
description: Include file containing specifications of Dadsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/12/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dadsv5-series virtual machines, so that I can select the appropriate VM size for my application's performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC 7763v (Milan) [x86-64] , AMD EPYC 9004 (Genoa) [x86-64]                              |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 1 Disk           | 75 - 3600 GiB <br>9000 - 450000 IOPS (RR) <br>125 - 4000 MBps (RR)                               |
| Remote Storage | 4 - 32 Disks    | 3750 - 80000 IOPS <br>82 - 1600 MBps   |
| Network        | 2 - 8 NICs          | 12500 - 40000 Mbps                          |
| Accelerators   | None              |                                   |
