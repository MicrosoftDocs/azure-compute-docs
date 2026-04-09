---
title: Ddsv5 series specs include
description: Include file containing specifications of Ddsv5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a system architect planning infrastructure, I want to review the specifications of Ddsv5-series VMs, so that I can determine the most suitable size to meet my application performance and capacity requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory         | 8 - 384 GiB          |                                                    |
| Local Storage  | 1 Disk     | 75 - 3,600 GiB <br>9,000 - 450,000 IOPS <br>125 - 4,000 MBps|
| Remote Storage | 4 - 32 Disks | 4,000 - 80,000 IOPS <br>85 - 2,600 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2), [Ultra Disk](../../../disks-types.md#ultra-disks) |
| Network        | 2 - 8 NICs          | 12,500 - 35,000 Mbps  <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1) |
| Accelerators   | None              |                                                     |
