---
title: Azure Cobalt processor-based Virtual Machines
description: An overview of Azure Cobalt processor-based Virtual Machines. 
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 11/07/2024
ms.author: archat
---

# Azure Cobalt processor-based Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Cobalt 100-based Virtual Machines (VMs) are powered by Azure's first-generation Cobalt 100 processor, delivering outstanding performance for both general-purpose and memory-optimized workloads. These VMs utilize Microsoft's first 64-bit Arm-based CPU, which was fully designed by Microsoft. The Cobalt 100 processor, based on the Arm's Neoverse N2 design, aims to enhance performance and power efficiency for broad range of workloads. Operating at 3.4 GHz, the Azure Cobalt 100 processor provides an entire physical core for each virtual machine vCPU. 


## List of VM series by family
The following section lists the VM series supported by Cobalt 100 processors. To find detailed information on a series, including specifications and recommended workloads, check out the links in the *Series* column. 

### Cobalt 100-based VMs
| Family | Series | Regions |
|----|---|---|
| General Purpose  | [Dpsv6-series](./general-purpose/dpsv6-series.md), [Dpdsv6-series](./general-purpose/dpdsv6-series.md), [Dplsv6-series](./general-purpose/dplsv6-series.md), [Dpldsv6-series](./general-purpose/dpldsv6-series.md) | Canada Central, Central US, East US 2, East US, Germany West Central, Japan East, Mexico Central, North Europe, Southeast Asia, Sweden Central, Switzerland North, UAE North, West Europe, and West US 2. |
| Memory Optimized | [Epsv6-series](./memory-optimized/epsv6-series.md), [Epdsv6-series](./memory-optimized/epdsv6-series.md) | Canada Central, Central US, East US 2, East US, Germany West Central, Japan East, Mexico Central, North Europe, Southeast Asia, Sweden Central, Switzerland North, UAE North, West Europe, and West US 2.|

To learn how to deploy these VMs for cloud-native applications on AKS, check out [Azure Kubernetes Service (AKS) demo on Cobalt 100 VMs](https://aka.ms/C100-VM-deploy-demo).
