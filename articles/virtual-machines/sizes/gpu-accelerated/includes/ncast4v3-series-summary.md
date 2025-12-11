---
title: NCasT4_v3-series summary include file
description: Include file for NCasT4_v3-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 06/30/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to understand the specifications and capabilities of the NCasT4_v3-series virtual machines, so that I can determine their suitability for deploying AI services and GPU-accelerated applications.
---
The NCasT4_v3-series virtual machines(VM) are powered by [NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) GPUs and AMD EPYC 7V12(Rome) CPUs. The VMs feature up to 4 NVIDIA T4 GPUs with 16 GB of memory each, up to 64 non-multithreaded AMD EPYC 7V12 (Rome) processor cores(base frequency of 2.45 GHz, all-cores peak frequency of 3.1 GHz and single-core peak frequency of 3.3 GHz) and 440 GiB of system memory. These VMs are ideal for deploying AI services- such as real-time inferencing of user-generated requests, or for interactive graphics and visualization workloads using NVIDIA's GRID driver and virtual GPU technology. Standard GPU compute workloads based around CUDA, TensorRT, Caffe, ONNX and other frameworks, or GPU-accelerated graphical applications based on OpenGL and DirectX can be deployed economically, with close proximity to users, on the NCasT4_v3 series.

On NCasT4_v3-series VMs, the Azure NVIDIA GPU driver extension installs CUDA drivers. For graphics and visualization workloads, manually install the Azure-supported GRID drivers.
