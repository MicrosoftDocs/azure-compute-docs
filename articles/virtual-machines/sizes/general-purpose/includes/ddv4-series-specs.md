---
title: Ddv4 series specs include
description: Include file containing specifications of Ddv4-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/29/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to review the specifications of Ddv4-series virtual machines so that I can select the optimal VM size for my application workloads based on performance and resource requirements.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 64 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8272CL (Cascade Lake) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]  |
| Memory         | 8 - 256 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 2400 GiB <br>9000 - 300000 IOPS (RR) <br>125 - 4000 MBps (RR)|
| Remote Storage | 4 - 32 Disks    | 3200 - 80000 IOPS <br>48 - 1200 MBps                     |
| Network        | 2 - 8 NICs          | 5000 - 30000 Mbps   <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1)    |
| Accelerators   | None              |                                                     |
