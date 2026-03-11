---
title: "Bsv2-series specs include"
description: Include file containing specifications of Bsv2-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to access detailed specifications of Bsv2-series VM sizes, so that I can select the appropriate virtual machine configuration for my project's performance and resource requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 32 vCPUs       | Intel Xeon Platinum 8473C (Sapphire Rapids) [x86-64] <br>Intel Xeon Platinum 8370C (Ice Lake) [x86-64]  <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory         | 1 - 128 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 4 - 32 Disks | 4,000 - 51,000 IOPS <br>85 - 600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds) |
| Network        | 2 - 4 NICs          | 6,250 Mbps     <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1)   |
| Accelerators   | None              |                                   |
