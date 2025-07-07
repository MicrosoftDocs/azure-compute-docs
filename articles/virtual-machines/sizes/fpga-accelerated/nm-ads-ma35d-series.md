---
title: NMads MA35d
description: Overview of NMads MA35d virtual machine
author:      iamwilliew
ms.author:   wwilliams
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date:     10/02/2024
ms.subservice: sizes
# Customer intent: As a cloud architect, I want to evaluate the NMads MA35D series virtual machines for video transcoding workloads, so that I can leverage their specialized hardware and performance advantages for efficient and cost-effective video processing in our projects.
---

# NMads MA35D series

The NMads MA35D-Series virtual machines are Azure's first SKU to offer specialized hardware (Xilinx MA35D "Supernova") accelerated VM optimized for batch and real-time video transcoding workloads. The VM series are powered by 4<sup>th</sup> generation AMD EPYC™ Genoa processors. It offers 1 ASIC video processing unit (VPU) with 8GB of memory in addition to 16 vCPUs, 32GB of RAM, 76GB of temporary storage, and 4Gbps of network bandwidth.

Compared with existing general-purpose CPU or GPU based solutions, the NMads MA35D-Series VM SKU provides higher throughout and lower latency while maintaining a lower TCO for customers. The VM series presents a huge opportunity to save on infrastructure costs and improve performance and efficiency on video transcoding. Additionally, the VM provides access to modern codecs such as AV1 to further improve efficiency for video processing. It is the ideal choice for running your video transcoding workloads on the cloud. 

## Host specifications

| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      |  192 vCPUs     | AMD EPYC (Genoa) [x86-64] |
| Memory         |  768 GiB        |    |
| Local Storage  |  3 Disk         | 2TB E1.S NVMe per disk |
| Remote Storage |  8-16 Disks        |  |
| Network        |  5 NICs        |  |
| Accelerators   |  10 MA35D cards/20 VPUs           | AMD Xilinx Alveo MA35D   |

## Feature support 

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported <br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported <br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported <br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported <br>[VM Generation 2 Support](/azure/virtual-machines/generation-2): Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](/azure/virtual-machines/ephemeral-os-disks): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported 

## Sizes in series 

---
### [Basics](#tab/Basics) 

vCPUs (Qty.) and Memory for each size 

| **Size Name** | **vCPUs (Qty.)** | **Memory (GB)** |
|---|---|---|
| **Standard_NM16ads_MA35D** | 16 | 32 |

### VM Basics resources 
- [Check vCPU quotas](/azure/virtual-machines/quotas) 

### [Local Storage](#tab/LocalStorage) 

Local (temp) storage info for each size 

| **Size Name** | **OS Disk Size (GB)** | **Temp Disk Size (GB)** |
|---|---|---|
| **Standard_NM16ads_MA35D** | 128 | 76 |

*Full Windows guest OS image is not supported on ephemeral OS disk.

### Storage resources
- [Introduction to Azure managed disks](/azure/virtual-machines/managed-disks-overview) 

- [Azure managed disk types](/azure/virtual-machines/disks-types) 

- [Share an Azure managed disk](/azure/virtual-machines/disks-shared)   

### [Network](#tab/Network)

Network interface info for each size 

| **Size Name** | **Max NICs (Qty.)** | **Max Bandwidth (Mbps)** |
|---|---|---|
| **Standard_NM16ads_MA35D** | 1 | 4000 |

### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview) 

- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput) 

### [Accelerators](#tab/Accelerators) 

Accelerator (GPUs, FPGAs, etc.) info for each size 

| **Size Name** | **Accelerators (Qty.)** | **Accelerator-Memory (GB)** |
|---|---|---|
| **Standard_NM16ads_MA35D** | 1 | 8 |

---

### Table definitions

Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput) 

Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance depends on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 

To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing). 

## Other size information 

[General purpose](/azure/virtual-machines/sizes-general)

[Memory optimized](/azure/virtual-machines/sizes-memory)

[Storage optimized](/azure/virtual-machines/sizes-storage)

[GPU optimized](/azure/virtual-machines/sizes-gpu)

[High performance compute](/azure/virtual-machines/sizes-hpc)

[Previous generations](/azure/virtual-machines/sizes-previous-gen)

You can [use the pricing calculator](https://azure.microsoft.com/pricing/calculator/) to estimate your Azure VMs costs.

For more information on disk types, see [What disk types are available in Azure?](/azure/virtual-machines/disks-types)

## Next steps 

Learn more about how [Azure compute units (ACU)](/azure/virtual-machines/acu) can help you compare compute performance across Azure SKUs. 

Check out [Azure Dedicated Hosts](/azure/virtual-machines/dedicated-hosts) for physical servers able to host one or more virtual machines assigned to one Azure subscription. 

Learn how to [Monitor Azure virtual machines](/azure/virtual-machines/monitor-vm). 


