---
title: Eadsv6 series specs include
description: Include file containing specifications of Eadsv6-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/01/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Eadsv6-series VMs, so that I can select the appropriate virtual machine size for my application requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 96 vCPUs     | AMD EPYC 9004 (Genoa) [x86-64] |
| Memory         | 16 - 672 GiB        |    |
| Local Storage  | 1 - 6 Disks         | 110 - 880 GiB <br>37500 - 1800000 IOPS <br>180 - 8640 MBps |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 4 - 32 Disks        | 4000 - 175000 IOPS <br>90 - 4320 MBps |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 4 - 32 Disks        | 4000 - 175000 IOPS <br>90 - 4320 MBps |
| Network        | 2 - 8 NICs        | 12500 - 40000 Mbps |
| Accelerators   | None            |     |
