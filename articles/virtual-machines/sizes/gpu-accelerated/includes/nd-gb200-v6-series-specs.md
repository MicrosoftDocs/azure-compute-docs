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
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 128 vCPUs       | Nvidia Grace CPU                               |
| Memory         | 900GB VM         |  LPDDR                                |
| Local Storage  | 4 Disks           | 16TB NVME Direct                               |
| Remote Storage | 16 Disks    | 80000 IOPS/1200 MBps  |
| Network        | 1 NICs          |  160Gb/s Ethernet                         |
| Accelerators   | 4 GPUs              | Nvidia Blackwell GPU (192GB)                                  |