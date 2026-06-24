---
title: HBv4-series VM overview, architecture, topology - Azure Virtual Machines | Microsoft Docs
description: Learn about the HBv4-series VM size in Azure.
services: virtual-machines
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.topic: concept-article
ms.date: 05/05/2026
ms.reviewer: wwilliams
ms.author: padmalathas
author: padmalathas
# Customer intent: As a cloud architect, I want to understand the architecture and specifications of the HBv4-series virtual machines, so that I can select the optimal VM size for my high-performance computing workloads.
---

# HBv4-series virtual machine overview 

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

An [HBv4-series](hbv4-series.md) server features 2 * 96-core EPYC 9V33X CPUs for a total of 192 physical "Zen4" cores with AMD 3D-V Cache. Simultaneous Multithreading (SMT) is disabled on HBv4. These 192 cores are divided into 24 sections (12 per socket), each section containing eight processor cores with uniform access to a 96 MB L3 cache. Azure HBv4 servers also use the following BIOS settings: 

```bash
Nodes per Socket (NPS) = 2
L3 as NUMA = Disabled
NUMA domains within VM OS = 4
C-states = Enabled
```

As a result, the server boots with four NUMA domains (2 per socket) each 48-cores in size. Each NUMA has direct access to six channels of physical DRAM. 

To provide room for the Azure hypervisor to operate without interfering with the VM, we reserve 16 physical cores per server. 


## VM topology
The following diagram shows the topology of the server. We reserve these 16 hypervisor host cores (yellow) symmetrically across both CPU sockets, taking the first two cores from specific Core Complex Dies (CCDs) in each NUMA domain, with the remaining cores for the HBv4-series VM (green).

![Screenshot of HBv4-series server Topology](./media/hpc/architecture/hbv4/hbv4-topology-server.png)

The CCD boundary is different from a NUMA boundary. On HBv4, a group of six (6) consecutive CCDs is configured as a NUMA domain, both at the host server level and within a guest VM. Thus, all HBv4 VM sizes expose four uniform NUMA domains that appear to an OS and application as shown below, each with different number of cores depending on the specific [HBv4 VM size](hbv4-series.md).

![Screenshot of HBv4-series VM Topology](./media/hpc/architecture/hbv4/hbv4-topology-vm.jpg)

Each HBv4 VM size is similar in physical layout, features, and performance of a different CPU from the AMD EPYC 9V33X, as follows:

| HBv4-series VM size             | NUMA domains | Cores per NUMA domain  | Similarity with AMD EPYC         |
|---------------------------------|--------------|------------------------|----------------------------------|
Standard_HB176rs_v4               | 4            | 44                     | Dual-socket EPYC 9684X           |
Standard_HB176-144rs_v4           | 4            | 36                     | Dual-socket EPYC 9684X           |
Standard_HB176-96rs_v4            | 4            | 24                     | Dual-socket EPYC 9684X           |
Standard_HB176-48rs_v4            | 4            | 12                     | Dual-socket EPYC 9384X           |
Standard_HB176-24rs_v4            | 4            | 6                      | Dual-socket EPYC 9184X           |

> [!NOTE]
> * The constrained cores VM sizes only reduce the number of physical cores exposed to the VM. All global shared assets (RAM, memory bandwidth, caches, GMI/xGMI connectivity, InfiniBand, Azure Ethernet network, local SSD) stay constant. This allows a customer to pick a VM size best tailored to workload or software licensing needs.


The virtual NUMA mapping of each HBv4 VM size is mapped to the underlying physical NUMA topology. There's no potential misleading abstraction of the hardware topology. 

The exact topology for the various [HBv4 VM size](hbv4-series.md) appears as follows using the output of [lstopo](https://linux.die.net/man/1/lstopo):

```bash
lstopo-no-graphics --no-io --no-legend --of txt
```
<br>
<details>
<summary>Select to view lstopo output for Standard_HB176rs_v4</summary>

![lstopo output for HBv4-176 VM](./media/hpc/architecture/hbv4/hbv4-176-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB176-144rs_v4</summary>

![lstopo output for HBv4-144 VM](./media/hpc/architecture/hbv4/hbv4-144-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB176-96rs_v4</summary>

![lstopo output for HBv4-64 VM](./media/hpc/architecture/hbv4/hbv4-96-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB176-48rs_v4</summary>

![lstopo output for HBv4-32 VM](./media/hpc/architecture/hbv4/hbv4-48-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB176-24rs_v4</summary>

![lstopo output for HBv4-24 VM](./media/hpc/architecture/hbv4/hbv4-24-lstopo.png)
</details>


## InfiniBand networking
HBv4 VMs also feature NVIDIA NDR InfiniBand network adapters (ConnectX-7) operating at up to 400 Gigabits/sec. The NIC is passed through to the VM via SRIOV, enabling network traffic to bypass the hypervisor. As a result, customers load standard Mellanox OFED drivers on HBv4 VMs as they would a bare metal environment.

HBv4 VMs support Adaptive Routing, Dynamic Connected Transport (DCT, in addition to the standard RC and UD transports), and hardware-based offload of MPI collectives to the onboard processor of the ConnectX-7 adapter. These features enhance application performance, scalability, and consistency, and usage of them is recommended.

## Temporary storage
HBv4 VMs feature three physically local SSD devices. One device is preformatted to serve as a page file and appears within your VM as a generic "SSD" device.

Two other, larger SSDs are provided as unformatted block NVMe devices. As the block NVMe device bypasses the hypervisor, it has higher bandwidth and IOPS.

When paired in a striped array, the NVMe SSDs provide bandwidths of up to 12 GB/s (reads) and 7 GB/s (writes) of bandwidth, and IOPS of up to 186,000 (reads) and 201,000 (writes).


## Hardware specifications 
| Hardware specifications          | HBv4-series VMs              |
|----------------------------------|----------------------------------|
| Cores                            | 176, 144, 96, 48, or 24 (SMT disabled)           | 
| CPU                              | AMD EPYC 9V33X                   | 
| CPU Frequency (non-AVX)          | 2.55 GHz (base), 3.7 GHz (boost)    | 
| Memory                           | 768 GB (RAM per core depends on VM size)         | 
| Local Disk                       | 2 * 1.8 TB NVMe (block), 480 GB SSD (page file) | 
| InfiniBand                       | 400 Gb/s NVIDIA ConnectX-7 NDR InfiniBand | 
| Azure Network                    | 100 Gb/s (80 Gb/s Azure Accelerated Networking) | 


## Software specifications 
| Software specifications        | HBv4-series VMs                                            | 
|--------------------------------|-----------------------------------------------------------|
| Max MPI Job Size               | 52,800 cores (300 VMs in a single virtual machine scale set with singlePlacementGroup=true)  |
| MPI Support                    | HPC-X, OpenMPI, MVAPICH2, MPICH   |
| Additional Frameworks          | UCX, libfabric, PGAS                  |
| Azure Storage Support          | Standard and Premium Disks (maximum 32 disks), Azure NetApp Files, Azure Files, Azure Managed Lustre File System             |
| Supported and Validated OS     | RHEL 8.6+, AlmaLinux 8.10+, Ubuntu 22.04+ LTS, SLES 15 SP7+, Windows Server 2022+  |
| Recommended OS for Performance | AlmaLinux HPC 9.7, Ubuntu HPC 24.04, Windows Server 2025    |
| Orchestrator Support           | Azure CycleCloud, Azure Batch, Azure Kubernetes Service                      | 



## Next steps

- Read about the latest announcements, HPC workload examples, and performance results at the [Azure Compute Tech Community Blogs](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- For a higher level architectural view of running HPC workloads, see [High Performance Computing (HPC) on Azure](/azure/architecture/topics/high-performance-computing/).
