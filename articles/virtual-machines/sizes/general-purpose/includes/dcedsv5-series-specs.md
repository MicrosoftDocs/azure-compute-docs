---
title: DCedsv5 series specs include
description: Include file containing specifications of DCedsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to access the specifications of DCedsv5-series VM sizes, so that I can determine the best VM configuration for my workload requirements.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | Intel Xeon (Sapphire Rapids) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 1 Disk           | 47 - 2,823 GiB <br>9,300 - 459,200 IOPS (RR) <br>100 - 4,000 MBps (RR)                               |
| Remote Storage <br />([Premium SSD](../../../disks-types.md#premium-ssds)) | 4 - 32 Disks    | 3,750 - 80,000 IOPS <br>80 - 2,600 MBps   |
| Network        | 2 - 8 NICs          | 3,000 - 30,000 Mbps <br>Interfaces: NetVSC, ConnectX                          |
| Accelerators   | None              |                                   |
