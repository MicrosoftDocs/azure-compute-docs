---
title: HBv2-series virtual machine (VM) overview - Azure Virtual Machines | Microsoft Docs
description: Learn about the HBv2-series VM size in Azure.
services: virtual-machines
ms.custom:
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.topic: concept-article
ms.date: 05/04/2026
ms.reviewer: cynthn
ms.author: padmalathas
author: padmalathas
ai-usage: ai-assisted
# Customer intent: As an HPC application developer, I want to understand the specifications and performance characteristics of the HBv2-series virtual machines, so that I can optimize my workloads and ensure efficient resource allocation for high-performance computing tasks.
---


# HBv2 series virtual machine overview

> [!IMPORTANT]
> **HBv2-series VMs are scheduled for retirement on May 31, 2027.** This applies to all HBv2 sizes: Standard_HB120rs_v2, Standard_HB120-96rs_v2, Standard_HB120-64rs_v2, Standard_HB120-32rs_v2, and Standard_HB120-16rs_v2. After this date, HBv2 VMs will be set to a deallocated state, stop working, stop incurring billing charges, and lose SLA and support. Plan your migration to current-generation HPC alternatives before the retirement date.

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets.

Maximizing high performance compute (HPC) application performance on AMD EPYC requires a thoughtful approach memory locality and process placement. Below we outline the AMD EPYC architecture and our implementation of it on Azure for HPC applications. We use the term **pNUMA** to refer to a physical NUMA domain, and **vNUMA** to refer to a virtualized NUMA domain.

Physically, an [HBv2-series](hbv2-series.md) server is 2 * 64-core EPYC 7V12 CPUs for a total of 128 physical cores. Simultaneous Multithreading (SMT) is disabled on HBv2. These 128 cores are divided into 16 sections (8 per socket), each section containing 8 processor cores. Azure HBv2 servers also run the following AMD BIOS settings:

```output
Nodes per Socket (NPS) = 2
L3 as NUMA = Disabled
NUMA domains within VM OS = 4
C-states = Enabled
```

As a result, the server boots with 4 NUMA domains (2 per socket). Each domain is 32 cores in size. Each NUMA has direct access to 4 channels of physical DRAM operating at 3,200 MT/s.

To provide room for the Azure hypervisor to operate without interfering with the VM, we reserve 8 physical cores per server.

## VM topology

We reserve these 8 hypervisor host cores symmetrically across both CPU sockets, taking the first 2 cores from specific Core Complex Dies (CCDs) on each NUMA domain, with the remaining cores for the HBv2-series VM.
The CCD boundary isn't equivalent to a NUMA boundary. On HBv2, a group of four consecutive (4) CCDs is configured as a NUMA domain, both at the host server level and within a guest VM. Thus, all HBv2 VM sizes expose 4 NUMA domains that appear to an OS and application. 4 uniform NUMA domains, each with different number of cores depending on the specific [HBv2 VM size](hbv2-series.md).

Process pinning works on HBv2-series VMs because we expose the underlying silicon as-is to the guest VM. We strongly recommend process pinning for optimal performance and consistency.


## Hardware specifications

| Hardware Specifications          | HBv2-series VM                   |
|----------------------------------|----------------------------------|
| Cores                            | 120 (SMT disabled)               |
| CPU                              | AMD EPYC 7V12                    |
| CPU Frequency (non-AVX)          | ~3.1 GHz (single + all cores)    |
| Memory                           | 4 GB/core (480 GB total)         |
| Local Disk                       | 960 GiB NVMe (block), 480 GB SSD (page file) |
| Infiniband                       | 200 Gb/s HDR NVIDIA Mellanox ConnectX-6 |
| Network                          | 50 Gb/s Ethernet (40 Gb/s usable) Azure second Gen SmartNIC |


## Software specifications

| Software Specifications     | HBv2-series VM                                            |
|-----------------------------|-----------------------------------------------------------|
| Max MPI Job Size                | 36,000 cores (300 VMs in a single virtual machine scale set with singlePlacementGroup=true) |
| MPI Support                     | HPC-X, OpenMPI, MVAPICH2, MPICH   |
| Additional Frameworks           | UCX, libfabric, PGAS |
| Azure Storage Support           | Standard and Premium Disks (maximum 8 disks), Azure NetApp Files, Azure Files, Azure HPC Cache, Azure Managed Lustre File System |
| Supported and Validated OS      | RHEL 8.3+, AlmaLinux 8.10+, Ubuntu 22.04+ LTS, SLES 15 SP2+, Windows Server 2022+  |
| Recommended OS for Performance | AlmaLinux HPC 9.7, Ubuntu HPC 24.04+, Windows Server 2025 |
| Orchestrator Support        | CycleCloud, Batch, AKS; [cluster configuration options](sizes-hpc.md#cluster-configuration-options)  |


## Availability and purchasing

> [!IMPORTANT]
> **Reserved Instance purchase end date**: 1-year and 3-year Reserved Instance purchases for HBv2-series VMs ended on April 2, 2026. Avoid new long-term RI commitments on HBv2-series. Existing RIs remain valid until their expiration date.

- **Pay-as-you-go**: HBv2-series VMs remain available on a pay-as-you-go basis until retirement on **May 31, 2027**.
- **Plan migration**: Begin planning and testing migrations to current-generation HPC alternatives ahead of the retirement date. If you have active RIs for HBv2, consider exchanging them for supported HPC series RIs or trading them in for an Azure Savings Plan for compute. For more information, see [Exchange and refund Azure reservations](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations).

## Migrate to current-generation HPC VMs

HBv2-series VMs will be retired on May 31, 2027. Microsoft recommends migrating to the following current-generation HPC VM series:

| Recommended series | Key characteristics |
|---|---|
| [HBv5-series](hbv5-series-overview.md) | 4th Gen AMD EPYC (Zen4), up to 368 cores, HBM3 memory, NDR InfiniBand – best for memory bandwidth-intensive workloads |
| [HX-series](hx-series-overview.md) | AMD EPYC 9004, up to 176 cores, large L3 cache – best suited for memory-capacity-intensive HPC workloads |
| [HBv4-series](hbv4-series-overview.md) | 4th Gen AMD EPYC (Genoa), up to 176 cores, NDR InfiniBand – strong all-round HPC replacement |
| [HBv3-series](hbv3-series-overview.md) | 3rd Gen AMD EPYC (Milan), up to 120 cores, HDR InfiniBand – comparable generation step-up from HBv2 |

Before migrating, validate your workloads on the target series:
- **MPI and RDMA**: Confirm compatibility with the target InfiniBand fabric (HDR v. NDR) and your MPI library version.
- **Memory bandwidth**: Run representative benchmarks to confirm performance parity or improvement on the target series.
- **Application testing**: Complete end-to-end testing of your HPC applications on the target series before production migration.

## Next steps

- For migration to current-generation HPC VMs, see the [Migrate to current-generation HPC VMs](#migrate-to-current-generation-hpc-vms) section in this article.
- For more information about [AMD EPYC architecture](https://bit.ly/2Epv3kC) and [multi-chip architectures](https://bit.ly/2GpQIMb), see the [HPC Tuning Guide for AMD EPYC Processors](https://bit.ly/2T3AWZ9).
- For latest announcements on HPC workload examples, and performance results see [Azure Compute Tech Community Blogs](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- For a higher level architectural view of running HPC workloads, see [High Performance Computing (HPC) on Azure](/azure/architecture/topics/high-performance-computing/).
- For more information on the current-generation HBv5 series, see [HBv5-series overview](hbv5-series-overview.md).
