---
title: ND_GB00_v6-series summary include file
description: Include file for ND_GB200_v6-series summary
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/19/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a data scientist, I want to utilize ND-GB200-v6 series virtual machines for my deep learning projects, so that I can leverage their high-performance GPUs and advanced interconnects for efficient model training and large-scale computations."
---
The ND-GB200-v6 series virtual machine (VM) is a flagship addition to the Azure GPU family, delivering unmatched performance for Deep Learning training, Generative AI, and HPC workloads. These VMs leverage the NVIDIA GB200 Tensor Core GPUs, built on the Blackwell architecture, which offer significant advancements in computational power, memory bandwidth, and scalability over previous generations.
Each ND-GB200-v6 VM is powered by two NVIDIA Grace CPUs and four NVIDIA Blackwell GPUs. The GPUs are interconnected via fifth-generation NVLink, providing a total of 4× 1.8 TB/s NVLink bandwidth per VM. This robust scale-up interconnect enables seamless, high-speed communication between GPUs within the VM. In addition, the VM offers a scale-out backend network with 4× 400 GB/s NVIDIA Quantum-2 CX7 InfiniBand connections per VM, ensuring high-throughput and low-latency communication when interconnecting multiple VMs.
NVIDIA GB200 NVL72 connects up to 72 GPUs per rack, enabling system to operate as a single computer. This 72 GPU rack scale system comprised of groups of 18 ND GB200 v6 VMs delivering up to 1.4 Exa-FLOPS of FP4 Tensor Core throughput, 13.5 TB of shared high bandwidth memory, 130TB/s of cross sectional NVLINK bandwidth, and 28.8Tb/s scale-out networking.

With 128 vCPUs per VM supporting the overall system, the architecture is optimized to efficiently distribute workloads and memory demands for AI and scientific applications. This design enables seamless multi-GPU scaling and robust handling of large-scale models.
These instances deliver best-in-class performance for AI, ML, and analytics workloads with out-of-the-box support for frameworks like TensorFlow, PyTorch, JAX, RAPIDS, and more. The scale-out InfiniBand interconnect is optimized for existing AI and HPC tools built on NVIDIA’s NCCL communication libraries, ensuring efficient distributed computing across large clusters.