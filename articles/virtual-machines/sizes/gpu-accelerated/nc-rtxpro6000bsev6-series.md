---
title: NC_RTXPRO6000BSE_v6 size series
description: Information on and specifications of the NC_RTXPRO6000BSE_v6-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 10/21/2025
ms.author: v-nmagatala
ms.reviewer: iamwilliew
# Customer intent: As a cloud architect, I want to understand the specifications and feature support of the NC_RTXPRO6000BSE_v6 size series, so that I can select the appropriate virtual machine size for my high-performance computing workloads.
---

# NC_RTXPRO6000BSE_v6 sizes series

The RTXPRO6000BSE series virtual machine (VM) is a powerful addition to the Azure GPU family. You can use this series for real-world Azure Applied AI training, visualization, and batch inference workloads.These VMs are ideal for real-world Applied AI workloads, such as:

	- GPU-accelerated analytics and databases
	- Batch inferencing with heavy pre and post-processing
	- Advanced visualization and rendering for design and engineering
	- Machine learning (ML) development and model training
	- Video processing and transcoding
	- AI/ML web services and virtual workstation environments

## Host specifications
| Part | Quantity <br><sup>Count Units | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
|Processor|16 - 320 vCPUs|4.2 GHz|
|Memory|64 - 1280 GiB|DDR5|
|Network|6 - 8 NICs|25,000 - 200,000 Mbps|
|Accelerators|1 - 4|NVIDIA RTX Pro 6000 GPU|


## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Sizes in series

### [General Purpose](#tab/sizegeneralpurpose)

|  Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_NC32ds_xl_RTXPRO6000BSE_v6|32|128|
| Standard_NC64ds_xl_RTXPRO6000BSE_v6|64|256|
| Standard_NC128ds_xl_RTXPRO6000BSE_v6|128|512|
| Standard_NC256ds_xl_RTXPRO6000BSE_v6|256|1024|
| Standard_NC320ds_xl_RTXPRO6000BSE_v6|320|1280|


### [Compute Optimized](#tab/sizecomputeoptimized)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
|Standard_NC16lds_xl_RTXPRO6000BSE_v6|16|64|
|Standard_NC32lds_xl_RTXPRO6000BSE_v6|32|64|
|Standard_NC64lds_xl_RTXPRO6000BSE_v6|64|128|
|Standard_NC128lds_xl_RTXPRO6000BSE_v6|128|256|
|Standard_NC256lds_xl_RTXPRO6000BSE_v6|256|512|
|Standard_NC320lds_xl_RTXPRO6000BSE_v6|320|640|
#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)


### [Memory Optimized](#tab/sizememoryoptimzed)

|  Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_NC32mds_xl_RTXPRO6000BSE_v6|32|256|
| Standard_NC64mds_xl_RTXPRO6000BSE_v6|64|512|
| Standard_NC128mds_xl_RTXPRO6000BSE_v6|128|1024|

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Bandwidth (Mbps) |
| --- | --- | --- |
|Standard_NC32ds_xl_RTXPRO6000BSE_v6|6|25000|
|Standard_NC64ds_xl_RTXPRO6000BSE_v6|6|50000|
|Standard_NC128ds_xl_RTXPRO6000BSE_v6|8|75000|
|Standard_NC256ds_xl_RTXPRO6000BSE_v6|8|100000|
|Standard_NC320ds_xl_RTXPRO6000BSE_v6|8|200000|
| Standard_NC16lds_xl_RTXPRO6000BSE_v6|4|25000|
| Standard_NC32lds_xl_RTXPRO6000BSE_v6|6|25000|
| Standard_NC64lds_xl_RTXPRO6000BSE_v6|6|50000|
| Standard_NC128lds_xl_RTXPRO6000BSE_v6|8|75000|
| Standard_NC256lds_xl_RTXPRO6000BSE_v6|8|100000|
| Standard_NC320lds_xl_RTXPRO6000BSE_v6|8|200000|
|Standard_NC32mds_xl_RTXPRO6000BSE_v6|6|25000|
|Standard_NC64mds_xl_RTXPRO6000BSE_v6|6|50000|
|Standard_NC128mds_xl_RTXPRO6000BSE_v6|8|75000|



#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).


---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]