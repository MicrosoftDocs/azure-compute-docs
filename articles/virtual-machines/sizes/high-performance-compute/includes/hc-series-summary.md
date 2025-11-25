---
title: HC-series summary include file
description: Include file for HC-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a computational scientist, I want to utilize HC-series virtual machines, so that I can run complex simulations and analyses efficiently with optimized performance and scalability."
---
HC-series Virtual Machine (VM)s are optimized for applications driven by dense computation, such as implicit finite element analysis, molecular dynamics, and computational chemistry. HC VMs feature 44 Intel Xeon Platinum 8168 processor cores, 8 GB of RAM per CPU core, and no hyperthreading. Intel Xeon Platinum processors work with Intel’s software tools, including the Intel Math Kernel Library. They also support advanced vector processing features like AVX-512 for faster calculations.

HC-series VMs feature 100 Gb/sec Mellanox EDR InfiniBand. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. These VMs also supports Adaptive Routing and Dynamic Connected Transport (DCT), along with standard Reliable Connection (RC) and Unreliable Datagram (UD) transports. These features enhance application performance, scalability, and consistency, and their usage is recommended.
