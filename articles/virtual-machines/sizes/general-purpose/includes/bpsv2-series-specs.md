---
title: Bpsv2 series specs include
description: Include file
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/09/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of Bpsv2-series VMs so that I can select the appropriate size for my application workloads efficiently."
---

| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 16 vCPUs       | Ampere Altra [Arm64]                              |
| Memory         | 1 - 64 GiB          |                                  |
| Local Storage  | None           |                                |
| Remote Storage | 4 - 32 Disks | 4,000 - 26,000 IOPS <br>85 - 600 MBps <br>Disk Types: [Premium SSD](../../../disks-types.md#premium-ssds) |
| Network        | 2 - 4 NICs          | 6,250 Mbps <br>Interfaces: NetVSC, ConnectX                          |
| Accelerators   | None              |                                   |
