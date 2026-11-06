---
title: Laosv5 series specs include
description: Include file containing specifications of Laosv5-series VM sizes.
author: zhousarah
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
ms.custom: include file
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 2 - 160 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                  |
| Memory         | 16 - 1,040 GiB          |                      |
| Local Storage  | 1 - 9 Disks           | 1,440 - 15,360 GB <br>215,625 - 20,700,000 IOPS <br>1,220 - 117,000 MBps                    |
| Remote Storage | 10 - 64 Disks    | 4,400 - 400,000 IOPS <br>150 - 12,000 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 3 - 15 NICs          | 25,000 - 200,000 Mbps <br>Interfaces: NetVSC, [MANA](https://aka.ms/ManaFAQ1)   |
| Accelerators   | None              |                       |
