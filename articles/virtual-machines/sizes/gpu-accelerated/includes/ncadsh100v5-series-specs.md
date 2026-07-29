---
title: NCads_H100_v5 series specs include
description: Include file containing specifications of NCads_H100_v5-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to review the specifications of the NCads_H100_v5 series VMs, so that I can assess their suitability for high-performance computing workloads in my projects."
---
| Part | Quantity <br><sup>Count Units</sup> | Specs <br><sup>SKU ID, Performance Units, etc.</sup>  |
|---|---|---|
| Processor      | 40 - 80 vCPUs       | AMD EPYC (Genoa) [x86-64]                               |
| Memory         | 320 - 640 GiB          |                                  |
| Local Storage  | 1 Disk           | 3,576 - 7,152 GiB <br> IOPS <br> MBps                               |
| Remote Storage | 8 - 16 Disks    | 100,000 - 240,000 IOPS <br>3,000 - 7,000 MBps   |
| Network        | 2 - 4 NICs          | 40,000 - 80,000 Mbps <br>Interfaces: NetVSC, ConnectX  |
| Accelerators   | 1 - 2 GPUs              | Nvidia PCIe H100 GPU (94GB)                     |
