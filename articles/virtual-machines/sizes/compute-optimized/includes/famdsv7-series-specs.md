---
title: Famdsv7 series specs include
description: Include file containing specifications of Famdsv7-series VM sizes.
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Famdsv7-series VM sizes, so that I can select the appropriate virtual machine configuration for my applications' performance and scalability needs."
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 1 - 80 vCPUs     | AMD EPYC 9005 (Turin)  [x86-64]                               |
| Memory         | 8 - 640 GiB          |                                  |
| Local Storage  | 1 - 6 Disks           | 110 - 2,200 GiB <br>37,500 - 3,000,000 IOPS <br>280 - 22,400 MBps                               |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds)  | 10 - 64 Disks    | 4,000 - 212,000 IOPS <br>118  - 10,344 MBps   |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 10 - 64 Disks    | 4,400 - 310,000 IOPS <br>136 - 10,356 MBps   |
| Network        | 2 - 15 NICs          | 16,000 - 80,000 Mbps <br>Interfaces: NetVSC, [MANA](https://aka.ms/ManaFAQ1)  |
| Accelerators   | None              |                                   |
