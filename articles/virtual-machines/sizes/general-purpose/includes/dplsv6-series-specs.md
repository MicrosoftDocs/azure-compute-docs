---
title: Dplsv6-series specs include
description: Include file containing specifications of Dplsv6-series VM sizes.
author: tomvcassidy
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/22/2024
ms.author: tomcassidy
ms.reviewer: tomcassidy
ms.custom: include file, build-2024
# Customer intent: "As a cloud architect, I want to review the specifications of Dplsv6-series VMs, so that I can determine their suitability for my application's performance and storage requirements."
---
| Part | Quantity <br>Count Units | Specs <br>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor    | 2 - 96  vCPUs      | Azure Cobalt 100 [Arm64]                     |
| Memory       | 4 - 192  GiB          |                         |
| Local Storage  | None                |                       |
| Remote Storage | 8 - 64 Disks | 4,000 - 200,000 IOPS <br>106 - 5,850 MBps <br>Disk Types: [Standard SDD/HDD](../../../disks-types.md#standard-ssds), [Premium SSD](../../../disks-types.md#premium-ssds), [Premium SSD v2](../../../disks-types.md#premium-ssd-v2), [Ultra Disk](../../../disks-types.md#ultra-disks) |
| Network      | 2 - 8  NICs          | 12,500 - 60,000  Mbps <br>Interfaces: NetVSC, ConnectX |
| Accelerators | None        |                          |