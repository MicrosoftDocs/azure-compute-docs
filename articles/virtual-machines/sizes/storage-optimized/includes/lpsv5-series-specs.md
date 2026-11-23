---
title: Lpsv5 series specs include
description: Include file containing specifications of Lpsv5-series VM sizes.
author: noahwood28
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/07/2026
ms.author: noahwood
ms.reviewer: mattmcinnes
ms.custom: include file
ai-usage: ai-generated
# Customer intent: "As a cloud architect, I want to review the specifications of Lpsv5-series VM sizes, so that I can select the appropriate resources for my workload."
---

| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup> |
| --- | --- | --- |
| Processor | 1 - 128 vCPUs | Azure Cobalt 200 [Arm64] |
| Memory | 8 - 1,024 GiB |  |
| Local Storage | 1 - 6 Disks | 240 - 3,840 GB <br>75,000 - 7,200,000 IOPS <br>380 - 36,000 MBps |
| Remote Storage | 4 - 64 Disks | 2,000 - 320,000 IOPS <br>60 - 8,770 MBps <br>Disk Types: [Standard SSD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2), [Ultra Disk](../../../disks-types.md#ultra-disks) |
| Network | 2 - 15 NICs | 12,500 - 85,000 Mbps <br>Interfaces: NetVSC, [MANA](https://aka.ms/ManaFAQ1) |
| Accelerators | None |  |
