---
title: NVadsA10_v5 series specs include
description: Include file containing specifications of NVadsA10_v5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to analyze the specifications of NVadsA10_v5-series VMs, so that I can determine the appropriate configuration for my workload requirements."
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 6 - 72 vCPUs     | AMD EPYC 74F3v (Milan) [x86-64] |
| Memory         | 55 - 880 GiB        |    |
| Local Storage  | 1 Disk         | 180 - 2,880 GiB  |
| Remote Storage | 4 - 32 Disks        | 6,400 - 80,000 IOPS <br>100 - 1,200 MBps |
| Network        | 1 - 8 NICs        | 5,000 - 80,000 Mbps <br>Interfaces: NetVSC, ConnectX  |
| Accelerators   | 1/6 - 2 GPUs            | Nvidia A10 GPU (24GB)    |
