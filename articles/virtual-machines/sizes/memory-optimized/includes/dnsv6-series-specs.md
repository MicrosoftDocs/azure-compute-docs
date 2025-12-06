---
title: Dnsv6 series specs include
description: Include file containing specifications of Dnsv6-series VM sizes.
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 05/08/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As an IT administrator, I want to access detailed specifications for Dnsv6-series VM sizes, so that I can effectively select the appropriate virtual machine configuration for my organization's workload requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 128 vCPUs     | Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory         | 8 - 512 GiB        |    |
| Local Storage  |   None    |  |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 8 - 64 Disks        | 3750 - 204800 IOPS <br>106 - 6782 MBps |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 8 - 64 Disks        | 4167 - 266667 IOPS <br>124 - 7935 MBps |
| Network        | 4 - 15 NICs        | 25000 - 200000 Mbps |
| Accelerators   | None            |     |
