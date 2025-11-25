---
title: HBv3-series summary include file
description: Include file for HBv3-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to understand the capabilities of HBv3-series VMs, so that I can assess their suitability for high-performance computing applications in my organization.
---
HBv3-series Virtual Machines (VMs) are designed for very demanding computing tasks. They help with things like studying how liquids move, analyzing structures, predicting weather, and processing earthquake data. They also support oil reservoir modeling and testing computer chip designs. HBv3 VMs feature up to 120 AMD EPYC™ 7V73X (Milan-X) CPU cores, 448 GB of RAM, and no simultaneous multithreading. HBv3-series VMs also provide 350 GB/sec of memory bandwidth (amplified up to 630 GB/s), up to 96 MB of L3 cache per core (1536 MB total per VM), up to 7 GB/s of block device SSD performance, and clock frequencies up to 3.5 GHz. All HBv3-series VMs feature 200 Gb/sec High Data Rate (HDR) InfiniBand from NVIDIA Networking to enable supercomputer-scale Message Pass Interface (MPI) workloads. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. The HDR InfiniBand fabric also supports Adaptive Routing and Dynamic Connected Transport (DCT), along with standard RC and UD transports. These features improve performance, make scaling easier, and keep results consistent.
