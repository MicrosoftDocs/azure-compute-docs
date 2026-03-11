---
title: Dasv4 series specs include
description: Include file containing specifications of Dasv4-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dasv4-series VMs, so that I can select the appropriate VM sizes for my workloads based on performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC 7452 (Rome) [x86-64] <br>AMD EPYC 7763v (Genoa) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 4 - 32 Disks           | 16 - 768 GiB <br>4,000 - 192,000 IOPS <br>32 - 1,020 MBps                               |
| Remote Storage | 4 - 32 Disks | 3,000 - 80,000 IOPS <br>48 - 1,200 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Ultra Disk](../../../disks-types.md#ultra-disks) |
| Network        | 2 - 8 NICs          | 2,000 - 40,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |
