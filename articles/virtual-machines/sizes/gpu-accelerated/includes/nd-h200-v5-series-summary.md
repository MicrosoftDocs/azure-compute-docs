---
title: ND-H200-v5-series summary include file
description: Include file for ND-H200-v5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 09/12/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a data scientist working on AI and HPC projects, I want to utilize ND H200 v5 series VMs with NVIDIA H200 GPUs, so that I can efficiently process larger datasets and complex models for generative AI and scientific computing."
---

The ND v5 H200 series virtual machine (VM) is designed to deliver exceptional performance for AI and high-performance computing (HPC) workloads. These VMs leverage the power of the NVIDIA H200 Tensor Core GPU, which offer a 76% increase in High Bandwidth Memory over the H100 GPUs to deliver higher performance on state-of-the-art Generative AI models. With 141 GB of high-speed memory, and 4.8 TB/s of memory bandwidth, the H200 GPU can handle larger datasets and more complex models, making it ideal for generative AI and scientific computing.  

The ND H200 v5 series starts with a single VM and eight NVIDIA H200 Tensor Core GPUs, interconnected with 900 GB/s NVLink. ND H200 v5-based deployments can scale up to thousands of GPUs with 3.2Tb/s of interconnect bandwidth per VM. Each GPU within the VM is provided with its own dedicated, topology-agnostic 400 Gb/s NVIDIA Quantum-2 CX7 InfiniBand connection. These connections are automatically configured between VMs occupying the same virtual machine scale set, and support GPUDirect RDMA.  

These instances provide excellent performance for many AI, ML, and analytics tools that support GPU acceleration "out-of-the-box" such as TensorFlow, Pytorch, Caffe, RAPIDS, and other frameworks. Additionally, the scale-out InfiniBand interconnect is supported by a large set of existing AI and HPC tools that are built on NVIDIA’s NCCL communication libraries for seamless clustering of GPUs. 
