---
title: Dsv4 series specs include
description: Include file containing specifications of Dsv4-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/29/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Dsv4-series VMs, so that I can select the appropriate resources for applications requiring optimal performance and scalability."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 64 vCPUs       | Intel Xeon Platinum 8473C (Sapphire Rapids) [x86-64] <br>Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8272CL (Cascade Lake) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]  |
| Memory         | 8 - 256 GiB          |                                                    |
| Local Storage  |  None     |  |
| Remote Storage | 4 - 32 Disks | 3,000 - 80,000 IOPS <br>48 - 1,200 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2), [Ultra Disk](../../../disks-types.md#ultra-disks) |
| Network        | 2 - 8 NICs          | 5,000 - 30,000 Mbps  <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1)  |
| Accelerators   | None              |                                                     |