---
title: NVIDIA CUDA on Azure overview
description: Learn what NVIDIA CUDA is, which Azure GPU VM sizes support it, and how to get started running CUDA workloads on NVIDIA GPUs.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/24/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud infrastructure planner, I want to understand NVIDIA CUDA and which Azure GPU VM sizes support it, so that I can choose the right virtual machine for my AI, HPC, or graphics workloads.
---

# What is NVIDIA CUDA?

NVIDIA CUDA is NVIDIA's parallel computing platform and programming model for GPU computing. It provides an end-to-end stack of drivers, compilers, runtimes, and libraries that let you run AI, high-performance computing (HPC), and other accelerated workloads on NVIDIA GPUs. CUDA gives you a programming model, math and communication libraries, and tooling that popular frameworks such as PyTorch, TensorFlow, JAX, and vLLM build on. For AMD GPUs, the comparable platform is [AMD ROCm](rocm-overview.md).

On Azure, CUDA is the software layer you install and use to unlock the compute capabilities of NVIDIA GPU-powered virtual machines (VMs). CUDA runs on both Linux and Windows.

## Key components of the CUDA platform

CUDA is a full stack rather than a single driver. Its main building blocks include:

- **CUDA Toolkit (compiler and runtime)**: The `nvcc` compiler, the CUDA runtime, and the CUDA C++ programming model that let you write and build GPU-accelerated code.
- **Math and compute libraries**: GPU-accelerated libraries such as cuBLAS, cuFFT, cuSPARSE, cuDNN (deep learning primitives), and CUTLASS that frameworks call for performance-critical operations.
- **NCCL (NVIDIA Collective Communications Library)**: A collective-communications library used to scale training and HPC workloads across many GPUs.
- **Compilers and tools**: An LLVM-based compiler, plus profiling and debugging tools such as Nsight Systems, Nsight Compute, and cuda-gdb.

### A compute-only stack

CUDA targets the general-purpose compute capabilities of a GPU (the parallel math that drives AI, machine learning, and HPC workloads) and doesn't use the graphics rasterization pipeline that turns 3D geometry into rendered pixels. In other words, CUDA treats the GPU as a massively parallel math engine, not as a display or rendering device.

This distinction matters when you choose a VM size. Azure offers both compute-only NVIDIA GPU sizes, such as the [ND-H100-v5 series](../sizes/gpu-accelerated/ndh100v5-series.md), and graphics-capable NVIDIA GPU sizes, such as the [NVadsA10 v5 series](../sizes/gpu-accelerated/nvadsa10v5-series.md), that include a rasterization pipeline for professional visualization and virtual desktop scenarios. CUDA runs on both categories, because it uses the GPU's compute units regardless of whether the hardware also supports rasterization.

## Azure GPU VM sizes with CUDA support

The following Azure GPU VM sizes use NVIDIA GPUs and support CUDA. Use the links to review full specifications for each series.

| VM size series | NVIDIA GPU | Typical workloads |
| --- | --- | --- |
| [ND-GB200-v6 series](../sizes/gpu-accelerated/nd-gb200-v6-series.md) | [NVIDIA GB200 NVL (Grace Blackwell)](https://www.nvidia.com/en-us/data-center/gb200-nvl72/) | Frontier-scale generative AI training and inference. |
| [ND-H200-v5 series](../sizes/gpu-accelerated/nd-h200-v5-series.md) | [NVIDIA H200](https://www.nvidia.com/en-us/data-center/h200/) | Large-scale deep learning training and inference, generative AI, and HPC. |
| [ND-H100-v5 series](../sizes/gpu-accelerated/ndh100v5-series.md) | [NVIDIA H100](https://www.nvidia.com/en-us/data-center/h100/) | Large-scale deep learning training and inference, and tightly coupled HPC. |
| [NCads H100 v5 series](../sizes/gpu-accelerated/ncadsh100v5-series.md) | [NVIDIA H100 NVL](https://www.nvidia.com/en-us/data-center/h100/) | Mid-scale training, inference, and HPC. |
| [NC A100 v4 series](../sizes/gpu-accelerated/nca100v4-series.md) | [NVIDIA A100](https://www.nvidia.com/en-us/data-center/a100/) | Training, inference, and HPC. |
| [NCasT4 v3 series](../sizes/gpu-accelerated/ncast4v3-series.md) | [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) | Inference, small-scale training, and visualization. |
| [NVadsA10 v5 series](../sizes/gpu-accelerated/nvadsa10v5-series.md) | [NVIDIA A10](https://www.nvidia.com/en-us/data-center/products/a10-gpu/) | GPU-accelerated graphics, virtual desktops, and light inference. |

For the full list of GPU-accelerated sizes, see the [GPU-accelerated VM sizes overview](../sizes/overview.md#gpu-accelerated).

## Supported operating systems

CUDA supports both Linux and Windows. The specific distributions, kernel versions, and driver versions that NVIDIA validates depend on the CUDA Toolkit release and the GPU:

- **Linux**: CUDA supports a broad set of enterprise distributions, including Ubuntu, Red Hat Enterprise Linux (RHEL), Rocky Linux, SUSE Linux Enterprise Server (SLES), and Debian.
- **Windows**: CUDA supports Windows Server and Windows 11. You can also run CUDA workloads in Windows Subsystem for Linux (WSL 2).

Because NVIDIA updates the validated OS, driver, and CUDA Toolkit version combinations with each release, always confirm the current requirements in the [NVIDIA CUDA Toolkit documentation](https://docs.nvidia.com/cuda/) before you deploy.

## Install CUDA on Azure NVIDIA GPU VMs

The recommended way to get a preconfigured environment on NVIDIA GPU VMs is to deploy an Azure Marketplace image that already includes the NVIDIA GPU driver and CUDA. You can also install the NVIDIA GPU driver and CUDA Toolkit manually on a supported distribution.

For Azure-specific, step-by-step guidance, see:

- [Install NVIDIA GPU drivers on N-series VMs running Linux](../linux/n-series-driver-setup.md)
- [Install NVIDIA GPU drivers on N-series VMs running Windows](../windows/n-series-driver-setup.md)
- [NVIDIA GPU Driver Extension for Linux](../extensions/hpccompute-gpu-linux.md)
- [NVIDIA GPU Driver Extension for Windows](../extensions/hpccompute-gpu-windows.md)

For NVIDIA's own installation and reference documentation, see:

- [CUDA Toolkit downloads](https://developer.nvidia.com/cuda-downloads)
- [CUDA installation guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
- [CUDA Toolkit documentation](https://docs.nvidia.com/cuda/)

## Related content

- [GPU-accelerated VM sizes overview](../sizes/overview.md#gpu-accelerated)
- [ND-H100-v5 size series](../sizes/gpu-accelerated/ndh100v5-series.md)
- [ND-H200-v5 size series](../sizes/gpu-accelerated/nd-h200-v5-series.md)
- [NCads H100 v5 size series](../sizes/gpu-accelerated/ncadsh100v5-series.md)
- [AMD ROCm on Azure overview](rocm-overview.md)
- [NVIDIA CUDA documentation](https://docs.nvidia.com/cuda/)
