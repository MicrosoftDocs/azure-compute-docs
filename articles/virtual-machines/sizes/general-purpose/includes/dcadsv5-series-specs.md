---
title: DCadsv5 series specs include
description: Include file containing specifications of DCadsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of DCadsv5-series VM sizes, so that I can select the appropriate configuration for my application workloads."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | AMD EPYC (Milan) [x86-64]                               |
| Memory         | 8 - 384 GiB          |                                  |
| Local Storage  | 1 Disk           | 75 - 3,600 GiB <br>9,000 - 450,000 IOPS <br>125 - 4,000 MBps                               |
| Remote Storage | 4 - 32 Disks | 4,000 - 80,000 IOPS <br>82 - 1,600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds) |
| Network        | 2 - 8 NICs          | 3,000 - 20,000 Mbps <br>Interfaces: NetVSC, ConnectX                          |
| Accelerators   | None              |                                   |
