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
HC-series VMs are optimized for applications driven by dense computation, such as implicit finite element analysis, molecular dynamics, and computational chemistry. HC VMs feature 44 Intel Xeon Platinum 8168 processor cores, 8 GB of RAM per CPU core, and no hyperthreading. The Intel Xeon Platinum platform supports Intel’s rich ecosystem of software tools such as the Intel Math Kernel Library and advanced vector processing capabilities such as AVX-512.

HC-series VMs feature 100 Gb/sec Mellanox EDR InfiniBand. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. These VMs support Adaptive Routing and the Dynamic Connected Transport (DCT, in addition to standard RC and UD transports). These features enhance application performance, scalability, and consistency, and their usage is recommended.
