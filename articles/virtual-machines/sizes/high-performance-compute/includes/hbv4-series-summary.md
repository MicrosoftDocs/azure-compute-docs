---
title: HBv4-series summary include file
description: Include file for HBv4-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to evaluate the HBv4-series VMs for HPC workloads, so that I can ensure they meet the performance and scalability requirements for my organization's computational tasks.
---
HBv4-series VMs are optimized for many high-performance computing (HPC) workloads. These include:
- Computational fluid dynamics
- Finite element analysis
- Frontend and backend Electronic Design Automation (EDA)
- Rendering and graphics work
- Molecular dynamics
- Computational geoscience
- Weather simulation
- Financial risk analysis

HBv4 VMs feature up to 176 AMD EPYC™ 9V33X ("Genoa-X") CPU cores with AMD's 3D V-Cache, clock frequencies up to 3.7 GHz, and no simultaneous multithreading. HBv4-series VMs also provide 768 GB of RAM, 2304 MB L3 cache. Each VM has 2304 MB of L3 cache that delivers up to 5.7 TB/s of bandwidth. This boosts DRAM bandwidth of 780 GB/s, giving an average of 1.2 TB/s of effective memory speed for many workloads. The VMs also provide up to 12 GB/s (reads) and 7 GB/s (writes) of block device SSD performance.

All HBv4-series VMs feature 400 Gb/s NDR InfiniBand from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. NDR continues to support features like Adaptive Routing and the Dynamically Connected Transport (DCT). This newest generation of InfiniBand also brings greater support for offload of MPI collectives, optimized real-world latencies due to congestion control intelligence, and enhanced adaptive routing capabilities. These features enhance application performance, scalability, and consistency, and their usage is recommended.
