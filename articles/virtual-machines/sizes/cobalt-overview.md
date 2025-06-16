---
title: Azure Cobalt processor-based Virtual Machines
description: An overview of Azure Cobalt processor-based Virtual Machines. 
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.workload: infrastructure-services
ms.date: 11/07/2024
ms.author: archat
---

# Azure Cobalt processor-based Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Cobalt 100-based Virtual Machines (VMs) are powered by Azure's first-generation Cobalt 100 processor. These VMs offer an attractive option for a wide range of scale-out and cloud-native Linux-based workloads, including data analytics, web and application servers, open-source databases, caches, and more. These VMs utilize Microsoft's first 64-bit Arm-based CPU, which was fully designed by Microsoft. The Cobalt 100 processor, based on the Arm's Neoverse N2 design, aims to enhance performance and power efficiency for broad range of workloads. Operating at 3.4 GHz, the Azure Cobalt 100 processor provides an entire physical core for each virtual machine vCPU. 


## List of VM series by family
The following section lists the VM series supported by Cobalt 100 processors. To find detailed information on a series, including specifications and recommended workloads, check out the links in the *Series* column. 

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
| Azure Linux | 2 |
| Debian | 11+ |
| Flatcar | N/A |
| Kali Linux | N/A |
| QNX Neutrino RTOS | 7.1 |
| Red Hat Enterprise Linux (RHEL) | 8.6+ |
| SUSE Linux Enterprise Server (SLES) | 15 SP4+ |
| Ubuntu | 20.04+ |
| Windows 11 Client | N/A |

To find the most up-to-date Guest OS support, visit [Azure portal](https://portal.azure.com/#create/Microsoft.VirtualMachine). Under "Instance Details," select "See all images," then click the "Image Type" filter to select "Arm64."



