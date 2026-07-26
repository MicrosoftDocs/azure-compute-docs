---
title: AMD ROCm on Azure overview
description: Learn what AMD ROCm is, which Azure GPU VM sizes support it, and how to get started running ROCm workloads on AMD Instinct and Radeon PRO GPUs.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/24/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud infrastructure planner, I want to understand AMD ROCm and which Azure GPU VM sizes support it, so that I can choose the right virtual machine for my AI, HPC, or graphics workloads.
---

# What is AMD ROCm?

AMD ROCm is AMD's open-source software platform for GPU computing. It provides an end-to-end stack of drivers, compilers, runtimes, and libraries that let you run AI, high-performance computing (HPC), and other accelerated workloads on AMD GPUs. ROCm is the AMD counterpart to NVIDIA CUDA and gives you a programming model, math and communication libraries, and tooling that popular frameworks such as PyTorch, TensorFlow, JAX, and vLLM build on. For NVIDIA GPUs, the comparable platform is [NVIDIA CUDA](cuda-overview.md).

On Azure, ROCm is the software layer you install and use to unlock the compute capabilities of AMD GPU-powered virtual machines (VMs). ROCm runs on Linux, and AMD also provides a HIP SDK for Windows for a subset of hardware.

## Key components of the ROCm platform

ROCm is a full stack rather than a single driver. Its main building blocks include:

- **HIP (Heterogeneous-computing Interface for Portability)**: A C++ runtime and kernel language that lets you write portable GPU code. HIP source can target AMD GPUs and, with the same code base, NVIDIA GPUs, which simplifies porting CUDA applications.
- **Math and compute libraries**: GPU-accelerated libraries such as rocBLAS, rocFFT, rocSPARSE, MIOpen (deep learning primitives), and hipBLASLt that frameworks call for performance-critical operations.
- **RCCL (ROCm Communication Collectives Library)**: A collective-communications library (the AMD equivalent of NCCL) used to scale training and HPC workloads across many GPUs.
- **Compilers and tools**: An LLVM-based compiler, plus profiling and debugging tools such as rocprofiler and rocgdb.

### A compute-only stack

ROCm targets the general-purpose compute capabilities of a GPU (the parallel math that drives AI, machine learning, and HPC workloads) and doesn't use the graphics rasterization pipeline that turns 3D geometry into rendered pixels. In other words, ROCm treats the GPU as a massively parallel math engine, not as a display or rendering device.

This distinction matters when you choose a VM size. Azure offers both compute-only AMD GPU sizes, such as the [ND-MI300X-v5 series](../sizes/gpu-accelerated/ndmi300xv5-series.md), and graphics-capable AMD GPU sizes, such as the [NGads V620 series](../sizes/gpu-accelerated/ngadsv620-series.md) and the [NVads V710 v5 series](../sizes/gpu-accelerated/nvadsv710-v5-series.md), that include a rasterization pipeline for gaming and virtual desktop scenarios. ROCm runs on both categories, because it uses the GPU's compute units regardless of whether the hardware also supports rasterization.

## Azure GPU VM sizes with ROCm support

The following Azure GPU VM sizes use AMD GPUs and have explicit ROCm support. Use the links to review full specifications for each series.

| VM size series | AMD GPU | Typical workloads |
| --- | --- | --- |
| [ND-MI300X-v5 series](../sizes/gpu-accelerated/ndmi300xv5-series.md) | [AMD Instinct MI300X](https://www.amd.com/en/products/accelerators/instinct/mi300/mi300x.html) | Large-scale deep learning training and inference, generative AI, and tightly coupled HPC. |
| [NGads V620 series](../sizes/gpu-accelerated/ngadsv620-series.md) | [AMD Radeon PRO V620](https://www.amd.com/en/products/accelerators/radeon-pro/amd-radeon-pro-v620.html) | Cloud gaming, graphics streaming, and GPU-accelerated virtual desktops, with ROCm compute support. |
| [NVads V710 v5 series](../sizes/gpu-accelerated/nvadsv710-v5-series.md) | [AMD Radeon PRO V710](https://www.amd.com/en/products/accelerators/radeon-pro/amd-radeon-pro-v710.html) | GPU-accelerated graphics, virtual desktops, and small-to-medium AI/ML inference. |

For the full list of GPU-accelerated sizes, see the [GPU-accelerated VM sizes overview](../sizes/overview.md#gpu-accelerated).

## Supported operating systems

ROCm supports Linux. The specific distributions and kernel versions that AMD validates depend on the GPU:

- **AMD Instinct MI300X** (ND-MI300X-v5) supports the broadest set of enterprise Linux distributions, including Ubuntu, Red Hat Enterprise Linux (RHEL), and SUSE Linux Enterprise Server (SLES).
- **AMD Radeon PRO V620 and V710** (NGads V620 and NVads V710 v5) support a narrower set of distributions, primarily recent Ubuntu Long Term Support (LTS) releases and, for the V710, recent RHEL releases.

Because AMD updates the validated OS, kernel, and ROCm version combinations with each release, always confirm the current requirements in the [AMD ROCm compatibility matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html) before you deploy.

## Install ROCm on Azure AMD GPU VMs

The recommended way to get a preconfigured environment on Instinct-based VMs is to deploy an Azure Marketplace image that already includes the AMD GPU and InfiniBand drivers along with ROCm. You can also install ROCm manually on a supported distribution.

For Azure-specific, step-by-step guidance, see:

- [Install AMD GPU drivers on N-series Linux VMs](../linux/azure-n-series-amd-gpu-driver-linux-installation-guide.md)
- [Install AMD GPU drivers on Azure ND MI300X v5 Linux VMs](../linux/azure-nd-mi300-series-amd-gpu-driver-linux-installation-guide.md)
- [AMD GPU Driver Extension for Linux](../extensions/hpccompute-amd-gpu-linux.md)

For AMD's own installation and reference documentation, see:

- [ROCm quick start install guide](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/quick-start.html)
- [Install ROCm by using the amdgpu-install tool](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/amdgpu-install.html)
- [ROCm release history](https://rocm.docs.amd.com/en/latest/release/versions.html)

## Related content

- [GPU-accelerated VM sizes overview](../sizes/overview.md#gpu-accelerated)
- [ND-MI300X-v5 size series](../sizes/gpu-accelerated/ndmi300xv5-series.md)
- [NGads V620 size series](../sizes/gpu-accelerated/ngadsv620-series.md)
- [NVads V710 v5 size series](../sizes/gpu-accelerated/nvadsv710-v5-series.md)
- [NVIDIA CUDA on Azure overview](cuda-overview.md)
- [AMD ROCm documentation](https://rocm.docs.amd.com/en/latest/)
