---
title: Dv5 series specs include
description: Include file containing specifications of Dv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/29/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dv5 series VM sizes, so that I can select the appropriate virtual machine configurations for my application's performance requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | Intel Xeon Platinum 8473C (Sapphire Rapids) [x86-64] <br>Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory         | 8 - 384 GiB          |                                                    |
| Local Storage  |  None     | |
| Remote Storage | 4 - 32 Disks | 4,000 - 80,000 IOPS <br>85 - 2,600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds) |
| Network        | 2 - 8 NICs          | 12,500 - 35,000 Mbps  <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1)  |
| Accelerators   | None              |                                                     |