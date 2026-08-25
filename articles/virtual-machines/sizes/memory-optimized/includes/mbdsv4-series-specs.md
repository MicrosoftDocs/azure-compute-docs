---
title: Mbdsv4 series specs include
description: Include file containing specifications of Mbdsv4-series VM sizes.
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 06/25/2026
ms.author: zhangjay
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to access the specifications of Mbdsv4-series VMs, so that I can determine the appropriate resources for my application's performance needs."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 16 - 304 vCPUs       | 6th generation Intel® Xeon® Scalable processors (Granite Rapids) [x86-64]                   |
| Memory         | 128 - 3892 GiB          |                      |
| Local Storage  | 2 Disks           | 800 - 12000 GiB <br>200,000 - 3,200,000 IOPS (RR) <br>2,000 - 16,000 MBps (SR)                   |
| Remote Storage | 64 Disks    | 64,000 - 780,000 IOPS <br>2,000 - 16,000 MBps |
| Network        | 8 NICs          | 16,000 - 100,000 Mbps              | Interfaces: NetVSC, MANA
| Accelerators   | None              |                       |
