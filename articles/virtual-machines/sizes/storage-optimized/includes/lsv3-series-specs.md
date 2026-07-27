---
title: Lsv3 series specs include
description: Include file containing specifications of Lsv3-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to review the specifications of Lsv3-series VM sizes, so that I can select the appropriate virtual machine configuration that meets the resource requirements of my applications.
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 8 - 80 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64]                               |
| Memory         | 64 - 640 GiB          |                                  |
| Local Storage  | 1 Temp Disk <br> 1 - 10 NVMe Disks          | 80 - 800 GiB Temp Disks <br> 1.92 TiB NVMe Disks                |
| Remote Storage | 16 - 32 Disks    | 12,800 - 80,000 IOPS <br>290 - 2,160 MBps   |
| Network        | 4 - 8 NICs          | 12,500 - 32,000 Mbps <br>Interfaces: NetVSC, ConnectX  |
| Accelerators   | None              |                                   |
