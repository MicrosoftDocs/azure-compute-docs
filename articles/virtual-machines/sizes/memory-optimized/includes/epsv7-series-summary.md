---
title: Epsv7-series summary include
description: Include file containing a summary of the Epsv7-series VM sizes.
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: include
ms.date: 11/09/2026
ai-usage: ai-generated
ms.author: noahwood
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to understand the capabilities and workloads of Epsv7-series VMs, so that I can select an appropriate size for my applications."
---

The Epsv7-series virtual machines are powered by Azure's second-generation Cobalt 200 processor, designed for memory-optimized workloads. The Azure Cobalt 200 processor is based on the Arm Neoverse V3 architecture and provides an entire physical core for each virtual machine vCPU. Cobalt 200 supports the [Processor Mode](../../../vm-processor-mode.md) feature, running at up to 3.6 GHz in Opportunistic mode and at 3.2 GHz in Deterministic mode. These virtual machines offer 8 GiB of memory per vCPU and a wide range of vCPU sizes.

These VMs are suited for memory-intensive scale-out and enterprise workloads, such as relational and NoSQL databases, data analytics engines, and in-memory caches. Epsv7-series virtual machines offer up to 128 vCPUs and 1,024 GiB of RAM. These VMs come without local disk, providing a better value proposition for workloads that don't require local temporary storage.

Epsv7-series virtual machines support Standard SSD, Standard HDD, and Premium SSD types. You can also attach Premium SSD v2 and Ultra Disk storage. Disk storage is billed separately from virtual machines. For more information, see [pricing for disks](https://azure.microsoft.com/pricing/details/managed-disks/).
