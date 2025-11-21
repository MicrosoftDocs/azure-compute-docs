---
title: NC_RTXPRO6000BSE_v6 series
description: Information on and specifications of the NC_RTXPRO6000BSE_v6-series sizes
author: magatala-MSFT
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/20/2025
ms.author: v-nmagatala
ms.reviewer: iamwilliew
# Customer intent: As a cloud architect, I want to understand the specifications and feature support of the NC_RTXPRO6000BSE_v6 size series, so that I can select the appropriate virtual machine size for my high-performance computing workloads.
---

# NC RTXPRO6000BSE v6 series

These NC RTX PRO 6000 BSE v6 VMs are powered by NVIDIA RTX PRO 6000 Blackwell Server Edition GPUs, each featuring 96 GB of ultra-fast GDDR7 memory and the latest Blackwell architecture to enable breakthrough performance in multimodal AI, physical AI, and high-fidelity graphics. The VM host is equipped with Intel Granite Rapids CPUs, providing all-core turbo frequency of up to 4.2 GHz to handle demanding pre- and post-processing steps efficiently. These VMs are ideal for converged AI and visual computing workloads, such as:
- Real-time digital twin and NVIDIA Omniverse simulation.
- LLM Inference and RAG (Retrieval-Augmented Generation) for models under 70B parameters.
- High-fidelity 3D rendering, product design, and video streaming.
- GPU-accelerated desktop virtualization (VDI) with NVIDIA RTX Virtual Workstation.
- Scientific visualization and High-Performance Computing (HPC) using FP32.
- Agentic AI application development and deployment.

Sign up to Participate in [NCv6 Public Preview](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9s7orOb3OJJnwABCNj_8JdUMzlLSzJFTTdRRE8yU0UxWFFYQlpYV1hDVy4u) !

## Host specifications
| Part | Quantity <br><sup>(Count Units) | Specs <br><sup>(SKU ID, Performance Units, etc.)  |
|---|---|---|
|Processor|upto 320 vCPUs|Intel GNR AP (Granite Rapids-AP) [x86-64], up to 4.2 GHz|
|Memory|	768 – 1536 GiB |High-speed DDR5|
|Disk|8 - 16 Disks|250,000 – 500,000 IOPS (Target) / 8,000 – 15,000 MBps (Target)|
|Network|2 - 4 NICs|200,000 Mbps|
|Accelerators|1 - 2 GPUs|NVIDIA RTX PRO 6000 BSE (96 GB GDDR7 vRAM per GPU)|


## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Available sizes

### [General Purpose](#tab/sizegeneralpurpose)

|  Size Name | vCPUs (Qty.) | Memory (GB) | Max NICs (Qty.)| Max Bandwidth(Mbps)| Temp Disk Storage(GiB)|
| --- | --- | --- |---|---|---|
| Standard_NC32ds_xl_RTXPRO6000BSE_v6|32|128|6|25000|256|
| Standard_NC64ds_xl_RTXPRO6000BSE_v6|64|256|6|50000|512|
| Standard_NC128ds_xl_RTXPRO6000BSE_v6|128|512|8|75000|1024|
| Standard_NC256ds_xl_RTXPRO6000BSE_v6|256|1024|8|100000|2048|
| Standard_NC320ds_xl_RTXPRO6000BSE_v6|320|1280|8|200000|2048|

### [Memory Optimized](#tab/sizememoryoptimized)

| Size Name | vCPUs (Qty.) | Memory (GB) |Max NICs (Qty.)| Max Bandwidth(Mbps)| Temp Disk Storage(GiB)|
| --- | --- | --- | --- | --- | --- |
|Standard_NC32mds_xl_RTXPRO6000BSE_v6|32|128|6|25000|256|
|Standard_NC64mds_xl_RTXPRO6000BSE_v6|64|256|6|50000|512|
|Standard_NC128mds_xl_RTXPRO6000BSE_v6|128|512|8|75000|1024|

### [Compute Optimized](#tab/sizecomputeoptimized)

| Size Name | vCPUs (Qty.) | Memory (GB) |Max NICs (Qty.)| Max Bandwidth(Mbps)| Temp Disk Storage(GiB)|
| --- | --- | --- | --- | --- | --- |
|Standard_NC16lds_xl_RTXPRO6000BSE_v6|16|64|4|25000|256|
|Standard_NC32lds_xl_RTXPRO6000BSE_v6|32|128|6|25000|256|
|Standard_NC64lds_xl_RTXPRO6000BSE_v6|64|256|6|50000|512|
|Standard_NC128lds_xl_RTXPRO6000BSE_v6|128|512|8|75000|1024|
|Standard_NC256lds_xl_RTXPRO6000BSE_v6|256|1024|8|100000|2048|
|Standard_NC320lds_xl_RTXPRO6000BSE_v6|320|1280|8|200000|2048|


#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)


## Storage Notes
- Temp disk speed varies between Random Read (RR) and Random Write (RW). RR is typically faster.
- Capacity is shown in GiB (1024³ bytes). When comparing to GB (1000³ bytes), GiB values may appear smaller. For example, 1023 GiB = 1098.4 GB. 
- Disk throughput is measured in IOPS and MBps (MBps = 10⁶ bytes/sec).
- For optimal storage performance, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

## Networking Notes

- Expected bandwidth is the maximum aggregated bandwidth per VM type across all NICs. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).
- Upper limits are guidelines, not guarantees. Actual performance depends on network congestion, application load, and settings. To optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  For bandwidth testing, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).


[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
