---
title: Dasv4 series specs include
description: Include file containing specifications of Dasv4-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dasv4-series VMs, so that I can select the appropriate VM sizes for my workloads based on performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC 7452 (Rome) [x86-64] <br>AMD EPYC 7763v (Milan) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 4 - 32 Disks           | 16 - 768 GiB <br>4000 - 192000 IOPS (RR) <br>32 - 1020 MBps (RR)                               |
| Remote Storage | 4 - 32 Disks    | 3200 - 80000 IOPS <br>48 - 1200 MBps   |
| Network        | 2 - 8 NICs          | 2000 - 40000 Mbps                          |
| Accelerators   | None              |                                   |
