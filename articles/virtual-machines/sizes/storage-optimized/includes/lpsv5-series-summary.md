---
title: Lpsv5-series summary include
description: Include file containing a summary of the Lpsv5-series VM sizes.
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: include
ms.date: 11/09/2026
ai-usage: ai-generated
ms.author: noahwood
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to understand the capabilities and workloads of Lpsv5-series VMs, so that I can select an appropriate size for my applications."
---

The Lpsv5-series virtual machines feature high-throughput, low-latency local NVMe storage. These VMs are powered by Azure's second-generation Cobalt 200 processor, which is based on the Arm Neoverse V3 architecture and provides an entire physical core for each virtual machine vCPU. Cobalt 200 supports the [Processor Mode](../../../vm-processor-mode.md) feature, running at up to 3.6 GHz in Opportunistic mode and at 3.2 GHz in Deterministic mode. The Lpsv5-series VMs are available in sizes from 1 to 128 vCPUs, with 8 GiB of memory per vCPU and up to 23,040 GB (6 x 3,840 GB) of local NVMe temporary storage.

Lpsv5-series VMs are well suited for scale-up or scale-out storage workloads that need a balance of SSD capacity, compute, and memory. These workloads include data preprocessing and staging, relational and NoSQL databases with local storage requirements, big data analytics, and search and indexing engines.

Lpsv5-series virtual machines support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Premium SSD v2 and Ultra Disk storage. Remote disk storage is billed separately from virtual machines. For more information, see [pricing for disks](https://azure.microsoft.com/pricing/details/managed-disks/).