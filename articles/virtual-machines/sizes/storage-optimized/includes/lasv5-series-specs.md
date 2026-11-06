---
title: Lasv5 series specs include
description: Include file containing specifications of Lasv5-series VM sizes.
author: zhousarah
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
ms.custom: include file
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, and other details</sup>  |
|---|---|---|
| Processor      | 2 - 160 vCPUs       | AMD EPYC 9005 (Turin) [x86-64]                  |
| Memory         | 16 - 1,280 GiB          |                      |
| Local Storage  | 1 - 8 Disks           | 480 - 3,840 GB <br>150,000 - 9,600,000 IOPS <br>750 - 48,000 MBps                    |
| Remote Storage | 10 - 64 Disks    | 4,000 - 400,000 IOPS <br>118 - 12,000 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 2 - 15 NICs          | 16,000 - 200,000 Mbps <br>Interfaces: NetVSC, [MANA](https://aka.ms/ManaFAQ1)   |
| Accelerators   | None              |                       |
