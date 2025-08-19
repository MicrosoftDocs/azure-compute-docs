---
title: Dadsv6 series specs include
description: Include file containing specifications of Dadsv6-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dadsv6-series VM sizes, so that I can select the appropriate virtual machine configuration for my applications' performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC 9004 (Genoa) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 880 GiB <br>37500 - 1800000 IOPS (RR) <br>180 - 8640 MBps (RR)                               |
| Remote Storage | 4 - 32 Disks    | 4000 - 175000 IOPS <br>90 - 4320 MBps   |
| Network        | 2 - 8 NICs          | 12500 - 40000 Mbps                          |
| Accelerators   | None              |                                   |
