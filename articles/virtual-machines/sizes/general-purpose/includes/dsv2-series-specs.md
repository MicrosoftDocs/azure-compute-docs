---
title: Dsv2 series specs include
description: Include file containing specifications of Dsv2-series VM sizes.
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/29/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As an IT administrator managing cloud resources, I want to access the detailed specifications of Dsv2-series VMs, so that I can select the appropriate VM size for my application's performance and scalability requirements."
---
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 1 - 16 vCPUs       | Intel Xeon Platinum 8370C (Ice Lake) [x86-64] <br>Intel Xeon Platinum 8272CL (Cascade Lake) [x86-64] <br>Intel Xeon 8171M (Skylake) [x86-64] <br>Intel Xeon E5-2673 v4 (Broadwell) [x86-64] <br>Intel Xeon E5-2673 v3 (Haswell) [x86-64] <br>Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]  |
| Memory         | 3.5 - 56 GiB          |                                                    |
| Local Storage  | 1 Disk     | 7 - 112 GiB <br>4000 - 64000 IOPS (RR) <br>32 - 512 MBps (RR)|
| Remote Storage | 4 - 64 Disks    |  3200 - 51200 IOPS <br>48 - 768 MBps                     |
| Network        | 2 - 8 NICs          | 750 - 12000 Mbps       <br>Interfaces: NetVSC, ConnectX, [MANA](https://aka.ms/ManaFAQ1)    |
| Accelerators   | None              |                                                     |
