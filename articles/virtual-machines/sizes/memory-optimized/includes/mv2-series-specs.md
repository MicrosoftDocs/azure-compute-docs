---
title: Mv2 series specs include
description: Include file containing specifications of Mv2-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 04/09/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Mv2-series virtual machines, so that I can determine their suitability for our application's performance and scalability requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 208 - 416 vCPUs       | Intel Xeon Platinum 8180M (Skylake) [x86-64]                   |
| Memory         | 2,850 - 11,400 GiB          |                      |
| Local Storage  | 1 Disk           | 4096 - 8192 GiB <br>80000 - 250000 IOPS (RR) <br>800 - 1600 MBps (RR)                   |
| Remote Storage | 64 Disks    | 40000 - 80000 IOPS <br>1000 - 2000 MBps |
| Network        | 8 NICs          | 16000 - 32000 Mbps              |
| Accelerators   | None              |                       |
