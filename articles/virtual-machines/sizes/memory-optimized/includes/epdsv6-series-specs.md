---
title: Epdsv6-series specs include
description: Include file containing specifications of Epdsv6-series VM sizes.
author: tomvcassidy
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/22/2024
ms.author: tomcassidy
ms.reviewer: tomcassidy
ms.custom: include file, build-2024
# Customer intent: As a cloud architect, I want to access the detailed specifications of the Epdsv6-series VM sizes, so that I can determine the appropriate configurations for my application's performance and scalability needs.
---
| Part | Quantity <br>Count Units | Specs <br>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor    | 2 - 96  vCores      | Azure Cobalt 100 [ARM-64]                      |
| Memory       | 16 - 672  GiB          |                         |
| Local Storage  | 1 - 6 Disks        | 110 - 880 GiB/Disk <br>15000 - 1800000 IOPS <br> 90 - 8640 MBps  |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds)   | 8 - 64 Disks     | 3750 - 153600 IOPS <br>106 - 5000 MBps     |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks)   | 8 - 64 Disks     | 4163 - 199987 IOPS <br>124 - 5850 MBps     |
| Network      | 2 - 8  NICs          | 12500 - 60000  Mbps           |
| Accelerators | None         |                          |
