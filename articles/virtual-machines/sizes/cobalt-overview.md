---
title: Azure Cobalt processor-based Virtual Machines
description: An overview of Azure Cobalt processor-based Virtual Machines. 
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.workload: infrastructure-services
ms.date: 11/09/2026
ai-usage: ai-assisted
ms.author: noahwood
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to compare Azure Cobalt 200 and Cobalt 100 virtual machine series, so that I can choose a processor generation and VM family for my workload."
---

# Azure Cobalt processor-based Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure Cobalt processor-based virtual machines (VMs) use Microsoft's custom-designed Arm64 processors for cloud-native workloads. The portfolio includes Cobalt 200-based and Cobalt 100-based VM series, with options for general-purpose, memory-optimized, and storage-optimized workloads.

## Azure Cobalt processors

### Cobalt 200

Cobalt 200-based Virtual Machines (VMs) are powered by Azure's second-generation Cobalt 200 processor. They build on Cobalt 100's strengths and deliver up to 50% higher CPU performance for a wide range of scale-out and cloud-native workloads, including data analytics, web and application servers, open-source databases, caches, and microservices. Cobalt 200 VMs also deliver outstanding performance for agentic AI workloads, such as agent runtimes, sandboxes, tool execution, retrieval, and orchestration.

The Cobalt 200 processor is based on Arm's Neoverse V3 core and offers a full physical core per vCPU. Cobalt 200 supports [Processor Mode](../vm-processor-mode.md), with frequencies up to 3.6 GHz in Opportunistic mode and 3.2 GHz in Deterministic mode.

Cobalt 200 also expands the Cobalt portfolio beyond general purpose and memory optimized VMs. It brings Cobalt to storage optimized L-series VMs (Lpsv5) for workloads like NoSQL databases, big data analytics, and search and indexing engines. It also brings Cobalt to high-memory M-series VMs (Mpsv4) for workloads like large in-memory databases and ERP systems.

### Cobalt 100

Cobalt 100-based Virtual Machines (VMs) are powered by Azure's first-generation Cobalt 100 processor. These VMs offer an attractive option for a wide range of scale-out and cloud-native Linux-based workloads, including data analytics, web and application servers, open-source databases, caches, and more. These VMs utilize Microsoft's first 64-bit Arm-based CPU, which was fully designed by Microsoft. The Cobalt 100 processor, based on the Arm's Neoverse N2 design, aims to enhance performance and power efficiency for broad range of workloads. Operating at 3.4 GHz, the Azure Cobalt 100 processor provides an entire physical core for each virtual machine vCPU.

## List of VM series by family
The following section lists the VM series powered by Cobalt 200 and Cobalt 100 processors. To find detailed information on a series, including specifications and recommended workloads, check out the links in the *Series* column.

### Cobalt 200-based VMs

| Family | Memory Ratio | Series |
| --- | --- | --- |
| General purpose - D family | 2 GiB: 1 vCPU | [Dplsv7-series](./general-purpose/dplsv7-series.md), [Dpldsv7-series](./general-purpose/dpldsv7-series.md) |
| General purpose - D family | 4 GiB: 1 vCPU | [Dpsv7-series](./general-purpose/dpsv7-series.md), [Dpdsv7-series](./general-purpose/dpdsv7-series.md) |
| Memory optimized - E family | 8 GiB: 1 vCPU | [Epsv7-series](./memory-optimized/epsv7-series.md), [Epdsv7-series](./memory-optimized/epdsv7-series.md) |
| Memory optimized - M family | 16 GiB: 1 vCPU | [Mpsv4-series](./memory-optimized/mpsv4-series.md), [Mpdsv4-series](./memory-optimized/mpdsv4-series.md) |
| Storage optimized - L family | 8 GiB: 1 vCPU | [Lpsv5-series](./storage-optimized/lpsv5-series.md) |

### Cobalt 100-based VMs
| Family | Memory Ratio | Series |
|---|---|---|
| General Purpose  | 2 GiB: 1 vCPU |[Dplsv6-series](./general-purpose/dplsv6-series.md), [Dpldsv6-series](./general-purpose/dpldsv6-series.md) |
| General Purpose  | 4 GiB: 1 vCPU | [Dpsv6-series](./general-purpose/dpsv6-series.md), [Dpdsv6-series](./general-purpose/dpdsv6-series.md)|
| Memory Optimized | 8 GiB: 1 vCPU | [Epsv6-series](./memory-optimized/epsv6-series.md), [Epdsv6-series](./memory-optimized/epdsv6-series.md) |

To learn how to deploy these VMs for cloud-native applications on AKS, check out [Azure Kubernetes Service (AKS) demo on Cobalt 100 VMs](https://aka.ms/C100-VM-deploy-demo).

To find the most up-to-date regional availability, visit the Virtual Machines section at the bottom of the [Product Availability by Region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

## Guest OS Support for Cobalt 100 VMs
The following section lists the Guest OS support for Cobalt 100 VMs. 

### Guest OS Support
| Guest OS | Version |
|---|---|
| AlmaLinux | 8+ |
| Azure Linux | 3 |
| Debian | 11+ |
| Flatcar | N/A |
| Kali Linux | N/A |
| QNX Neutrino RTOS | 7.1 |
| Red Hat Enterprise Linux (RHEL) | 8.6+ |
| SUSE Linux Enterprise Server (SLES) | 15 SP4+ |
| Ubuntu | 20.04+ |
| Windows 11 Client | N/A |

To find the most up-to-date Guest OS support, visit [Azure portal](https://portal.azure.com/#create/Microsoft.VirtualMachine). Under "Instance Details," select "See all images," then click the "Image Type" filter to select "Arm64."


