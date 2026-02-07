---
title: Fsv2 series specs include
description: Include file containing specifications of Fsv2-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/29/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Fsv2-series VM sizes, so that I can select the appropriate configurations for my workloads based on performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 72 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8272CL (Cascade Lake) [x86-64] <br>Intel Xeon Platinum 8168 (Skylake) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]  |
| Memory         | 4 - 144 GB          |                                  |
| Local Storage  | 1 Disk           | 16 - 576 GiB <br>4000 to 144000 IOPS (RR) and 31 to 1152 MBps (RR)                               |
| Remote Storage | 4 - 32 Disks    | 3200 to 80000 IOPS <br> and 47 to 1100 MBps   |
| Network        | 2 - 8 NICs          | 5000 to 30000 Mbps    <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ)   |
| Accelerators   | None              |                                   |
