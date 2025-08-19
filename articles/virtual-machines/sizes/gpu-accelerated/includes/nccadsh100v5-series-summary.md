---
title: NCCads_H100_v5-series summary include file
description: Include file for NCCads_H100_v5-series summary
author: kphande
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/06/2024
ms.author: khande
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a data scientist, I want to deploy virtual machines with advanced GPU capabilities, so that I can efficiently run applied AI workloads and enhance machine learning development."
---
The NCCads H100 v5 series of virtual machines are a new addition to the Azure GPU family. In this VM SKU, Trusted Execution Environment (TEE) spans confidential VM on the CPU and attached GPU, enabling secure offload of data, models, and computation to the GPU.
The NCCads H100 v5 series is powered by 4th-generation AMD EPYC™ Genoa processors and NVIDIA H100 Tensor Core GPU. These VMs feature 1 NVIDIA H100 NVL GPUs with 94 GB memory, 40 non-multithreaded AMD EPYC Genoa processor cores, and 320 GiB of system memory. These VMs are ideal for real-world Applied AI workloads, such as:

- GPU-accelerated analytics and databases
- Batch inferencing with heavy pre- and post-processing
- Machine Learning (ML) development
- Video processing
- AI/ML web services
