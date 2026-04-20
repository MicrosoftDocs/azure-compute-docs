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
HC-series Virtual Machine (VM)s are built for heavy computing tasks. They handle jobs like analyzing complex structures, studying how molecules behave, and running advanced chemistry simulations. HC VMs feature 44 Intel Xeon Platinum 8168 processor cores, 8 GB of RAM per CPU core, and no hyperthreading. Intel Xeon Platinum processors work with Intel’s software tools, including the Intel Math Kernel Library. They also support advanced vector processing features like AVX-512 for faster calculations.

HC-series VMs feature 100 Gb/sec Mellanox EDR InfiniBand. These VMs are connected in a nonblocking fat tree for optimized and consistent RDMA performance. These VMs also supports Adaptive Routing and Dynamic Connected Transport (DCT), along with standard Reliable Connection (RC) and Unreliable Datagram (UD) transports. These features enhance application performance, scalability, and consistency, and their usage is recommended.
