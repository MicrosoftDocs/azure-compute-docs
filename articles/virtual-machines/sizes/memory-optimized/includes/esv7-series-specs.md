---
title: Esv7 series specs include
description: Include file containing specifications of Esv7-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/01/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As an IT specialist, I want to access detailed specifications for the Esv6-series VMs, so that I can select the appropriate configuration for my workloads based on performance and capacity needs.
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 372 vCPUs     | Intel@ Xeon® 6 processor 6973P-C(Granite Rapids) [x86-64] |
| Memory         | 16 - 2,832 GiB        |    |
| Local Storage  | None         |  |
| Remote Storage <br /> [Premium SSD](../../../disks-types.md#premium-ssds) | 10 - 64 Disks        | 4,000 - 500,000 IOPS <br>115 - 16,000 MBps |
| Remote Storage <br /> [Premium SSD v2](../../../disks-types.md#premium-ssd-v2) / [Ultra Disks](../../../disks-types.md#ultra-disks) | 10 - 64 Disks        | 5,000 - 800,000 IOPS <br>135 - 20,000 MBps |
| Network        | 3 - 15 NICs        | 16,000 - 400,000 Mbps |
| Accelerators   | None            |     |
