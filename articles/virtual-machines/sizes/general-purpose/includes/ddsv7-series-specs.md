---
title: Ddsv7 series specs include
description: Include file containing specifications of Ddsv7-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to review the specifications of Ddsv7-series VM sizes, so that I can select the appropriate virtual machine configurations for my application's requirements.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 372 vCPUs       |Intel Xeon 6 6973PC (Granite Rapids) [x86-64]   |
| Memory         | 8 - 1,488 GiB          |                                  |
| Local Storage  | 1 - 6 Disks        |  110 - 3,520 GiB <br>50,000 - 9,600,000 IOPS  <br>280 - 53,760 MBps                               |
| Remote Storage | 10 - 64 Disks | 4,000 - 500,000 IOPS <br>115 - 16,000 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) |
| Network        | 3 - 15 NICs          | 16,000 - 200,000 Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators   | None              |                                   |
