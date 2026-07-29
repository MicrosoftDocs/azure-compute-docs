---
title: ND GB200 v6 series specs include
description: Include file containing specifications of ND GB200 v6 series VM sizes.
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/20/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of the ND GB200 v6 series VMs, so that I can ensure they meet the performance requirements for my high-demand workloads."
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 128 vCPUs       | Nvidia Grace CPU                               |
| Memory         | 900 GiB        |  LPDDR5X                                |
| Local Storage  | 4 Disks           | 16 TiB (NVMe)                               |
| Remote Storage | 16 Disks    | 80,000 IOPS <br>1,200 MBps  |
| Network        | 1 NICs          |  160 Gbps Ethernet                         |
| Accelerators   | 4 GPUs              | Nvidia Blackwell GPU (192 GiB)                                  |