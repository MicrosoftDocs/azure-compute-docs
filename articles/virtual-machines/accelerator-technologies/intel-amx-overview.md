---
title: Intel AMX on Azure overview
description: Learn what Intel AMX is, which Azure VM sizes support it, and how to get started running AI workloads that use Intel AMX on Azure.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/24/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud infrastructure planner, I want to understand Intel AMX and which Azure VM sizes support it, so that I can choose the right virtual machine for my AI inference and training workloads.
---

# What is Intel AMX?

Intel Advanced Matrix Extensions (Intel AMX) is a built-in AI accelerator in Intel Xeon Scalable processors. It's an extension to the x86 instruction set that accelerates the matrix-multiplication operations at the heart of deep learning, so you can run AI inference and training directly on the CPU without a discrete accelerator such as a GPU. Intel introduced Intel AMX with 4th Generation Intel Xeon Scalable processors (Sapphire Rapids).

On Azure, Intel AMX is available on virtual machines (VMs) that use these newer Intel processors. Because it's a CPU instruction-set feature rather than a separate device, there's no driver to install. You benefit from Intel AMX by running an AI framework or library that targets it, and it works on both Linux and Windows.

## Key capabilities of Intel AMX

Intel AMX adds dedicated matrix-processing hardware to each CPU core. Its main elements include:

- **Tiles**: A set of two-dimensional registers that hold large blocks of matrix data close to the core.
- **Tile Matrix Multiplication (TMUL)**: An accelerator engine that operates on the tiles to perform matrix-multiply operations in a single instruction.
- **BF16 and INT8 data types**: Support for bfloat16 (training and inference) and 8-bit integer (inference) precision, which speeds up AI math while reducing memory use.
- **Framework integration**: Optimizations upstreamed into popular frameworks and libraries such as PyTorch, TensorFlow, and Intel oneDNN, so you can use Intel AMX with minimal code changes.

## Azure VM sizes with Intel AMX support

Intel AMX is available on every Azure VM size that uses 4th Generation Intel Xeon Scalable processors (Sapphire Rapids) or a newer Intel generation, such as 5th Generation (Emerald Rapids) and 6th Generation (Granite Rapids). That covers a large and growing number of series, including most current-generation Intel general-purpose and memory-optimized sizes such as the [Dsv6](../sizes/general-purpose/dsv6-series.md), [Esv6](../sizes/memory-optimized/esv6-series.md), [Dsv7](../sizes/general-purpose/dsv7-series.md), and [Esv7](../sizes/memory-optimized/esv7-series.md) series.

Because so many sizes qualify, here's a list of sizes that *don't* have Intel AMX:

| VM sizes | Why Intel AMX isn't available |
| --- | --- |
| Intel sizes on older processor generations, such as the [Dsv5](../sizes/general-purpose/dsv5-series.md) and [Esv5](../sizes/memory-optimized/esv5-series.md) series (Intel Ice Lake) and v4 and earlier series (Intel Cascade Lake) | These processors predate Intel AMX, which was introduced with 4th Generation Intel Xeon Scalable (Sapphire Rapids). |
| AMD-based sizes, such as the [Dasv6](../sizes/general-purpose/dasv6-series.md) and [Easv6](../sizes/memory-optimized/easv6-series.md) series (AMD EPYC) | Intel AMX is an Intel-only instruction-set extension. |
| Arm-based sizes, such as the Azure Cobalt 100 ([Dpsv6](../sizes/general-purpose/dpsv6-series.md), [Epsv6](../sizes/memory-optimized/epsv6-series.md)) and Ampere Altra ([Dpsv5](../sizes/general-purpose/dpsv5-series.md), [Epsv5](../sizes/memory-optimized/epsv5-series.md)) series | Intel AMX is an Intel x86-only feature. |

To confirm whether a specific size includes Intel AMX, check the processor listed on its size page. If the size uses a 4th Generation Intel Xeon (Sapphire Rapids) processor or newer, it supports Intel AMX. For the full list of VM sizes, see the [VM sizes overview](../sizes/overview.md).

## Verify Intel AMX is available

After you deploy a VM, confirm that Intel AMX is present on the CPU. On Linux, check the CPU flags:

```bash
lscpu | grep amx
```

If Intel AMX is available, the output lists the `amx_tile`, `amx_bf16`, and `amx_int8` flags.

## Use Intel AMX

Intel AMX is a CPU feature, so there's no driver to install. To take advantage of it, run AI frameworks and libraries that are optimized for Intel AMX. Intel upstreams these optimizations into popular open-source projects, so in many cases you only need a recent framework version.

For Intel's own documentation and getting-started resources, see:

- [What is Intel AMX?](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/what-is-intel-amx.html)
- [Intel AMX overview for Intel Xeon processors](https://www.intel.com/content/www/us/en/products/details/processors/xeon/features/advanced-matrix-extensions.html)
- [PyTorch optimizations from Intel](https://www.intel.com/content/www/us/en/developer/tools/oneapi/optimization-for-pytorch.html)

## Related content

- [VM sizes overview](../sizes/overview.md)
- [Dsv6 size series](../sizes/general-purpose/dsv6-series.md)
- [Esv6 size series](../sizes/memory-optimized/esv6-series.md)
- [AMD ROCm on Azure overview](rocm-overview.md)
- [NVIDIA CUDA on Azure overview](cuda-overview.md)
- [Intel AMX documentation](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/what-is-intel-amx.html)
