---
title: HX-series summary include file
description: Include file for HX-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 01/31/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to assess the capabilities of HX-series virtual machines, so that I can determine their suitability for running memory-intensive workloads in applications like silicon design and advanced manufacturing processes.
---
HX-series VMs are optimized for workloads that require significant memory capacity with twice the memory capacity as HBv4. For example, workloads such as silicon design can use HX-series VMs to enable EDA customers targeting the most advanced manufacturing processes to run their most memory-intensive workloads.

HX VMs feature up to 176 AMD EPYC™ 9V33X ("Genoa-X") CPU cores with AMD's 3D V-Cache, clock frequencies up to 3.7 GHz, and no simultaneous multithreading. HX-series VMs also provide 1408 GB of RAM, 2.3 GB L3 cache. The 2.3 GB L3 cache per VM can deliver up to 5.7 TB/s of bandwidth to amplify up to 780 GB/s of bandwidth from DRAM, for a blended average of 1.2 TB/s of effective memory bandwidth across a broad range of customer workloads. The VMs also provide up to 12 GB/s (reads) and 7 GB/s (writes) of block device SSD performance.

All HX-series VMs feature 400 Gb/s NDR InfiniBand from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. NDR continues to support features like Adaptive Routing and the Dynamically Connected Transport (DCT). This newest generation of InfiniBand also brings greater support for offload of MPI collectives, optimized real-world latencies due to congestion control intelligence, and enhanced adaptive routing capabilities. These features enhance application performance, scalability, and consistency, and their usage is recommended.
