---
title: ND GB300-v6 size series
description: Information on and specifications of the ND GB300-v6-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/13/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to assess the specifications and features of the ND GB300-v6 series, so that I can choose the optimal virtual machine size for our high-performance computing needs.
---

The NDGB300v6 series is Azure’s next-generation GPU VM line purposebuilt for largescale AI, especially highthroughput inference for reasoning and agentic systems. These VMs are powered by NVIDIA GB300 NVL72 rackscale systems (Blackwell Ultra GPUs + Grace CPUs).  Each ND-GB300-v6 VM has two NVIDIA Grace CPUs and four NVIDIA Blackwell B300 GPUs. The GPUs are interconnected via fifth-generation NVLINK, providing a total of 4× 1.8 TB/s NVLINK bandwidth per VM. Each GPU also has 800 Gb/s InfiniBand connectivity, enabling clusterscale performance with low latency. There are 18 VMs per rack, so effectively 72 NVIDIA Blackwell Ultra GPUs with 36 NVIDIA Grace CPUs, exposing ~37 TB of fast memory (~20TB High Bandwidth Memory, ~17TB of CPU Memory), 130 TB/s of intrarack NVLink bandwidth.  

At rack scale, ND GB300 v6 delivers up to 1.44 exaFLOPS FP4 Tensor Core performance per NVL72 domain and has demonstrated ~1.1 million tokens/second LLM inference throughput per rack, which is ~27% higher than ND GB200 v6. See here for more details. Compared to GB200, GB300 provides ~1.5× FP4 compute, 50% higher HBM capacity (288 GB HBM3E per GPU vs. 192 GB), and 2x the back-end network bandwidth per GPU (800Gbps vs 400 Gbps). 

The ND GB300 v6 architecture builds on the ND v6 GB200 to efficiently distribute workloads and memory demands across multiple GPUs, driving markedly higher inference throughput for longcontext and multimodal workloads AI and scientific applications. These instances deliver best-in-class performance for AI, ML, and analytics workloads with out-of-the-box support for frameworks like PyTorch, Tensorflow, JAX, RAPIDS, and more.  
