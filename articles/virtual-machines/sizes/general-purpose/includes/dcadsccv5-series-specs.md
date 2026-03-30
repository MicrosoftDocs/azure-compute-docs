---
title: DCads_cc_v5 series specs include
description: Include file containing specifications of DCads_cc_v5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to access detailed specifications for DCads_cc_v5 series VMs, so that I can select the appropriate VM sizes to meet my application performance requirements.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 4 - 96 vCPUs       | AMD EPYC (Milan) [x86-64]                               |
| Memory         | 16 - 384 GiB          |                                  |
| Local Storage  | 1 Disk           | 150 - 3,600 GiB        |
| Remote Storage | 8 - 32 Disks | 6,000 - 80,000 IOPS <br>144 - 1,600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds) |
| Network        | 2 - 8 NICs          | Mbps <br>Interfaces: NetVSC, ConnectX                          |
| Accelerators   | None              |                                   |
