---
title: HBv5 series summary include file
description: Include file for HBv5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 10/05/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
---

HBv5-series VMs are optimized for the most memory bandwidth-intensive HPC applications, including:

* Computational fluid dynamics
* Automotive and aerospace simulations
* Weather modeling
* Energy research
* Molecular dynamics simulations
* Computer-aided engineering and other HPC workloads

HBv5 VMs feature 6.7 TB/s of memory bandwidth across 432 GB (403 GiB) of high-bandwidth memory (HBM) and up to 368 4th Generation AMD EPYC™ processor cores with 4 GHz boost frequencies, 3.5 GHz base frequencies, and no simultaneous multithreading. 

Each HBv5-series VM also includes 14.3 TiB of local NVMe SSD storage with up to 50 GB/s (reads) and 30 GB/s (writes) of block device performance.

All HBv5-series VMs feature 800 Gb/s per node (4 x 200 Gb/s CX-7 NIC) of InfiniBand connectivity from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a nonblocking fat tree for optimized and consistent RDMA performance. 

The InfiniBand NICs support features like Adaptive Routing, Dynamically Connected Transport (DCT), hardware acceleration of MPI collectives, and congestion control. These features enhance application performance, scalability, and consistency, and their usage is recommended for optimal performance and cost efficiency.
