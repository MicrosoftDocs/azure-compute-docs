---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: Mdsv3 Very High Memory Series
description: Overview of Mdsv3 Very High Memory Virtual Machines
author:      phgantik # GitHub alias
ms.author:   phgantik # Microsoft alias
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.date:     09/11/2024
---

# Mdsv3 Very High Memory Series

The Mdsv3 Very High Memory (VHM) series, powered by 4<sup>th</sup> generation Intel® Xeon® Platinum 8490H (Sapphire Rapids) processor with an all core base frequency of 1.9 GHz and 3.5 GHz single core turbo frequency, are the next generation of memory-optimized VM sizes delivering faster performance, lower total cost of ownership and improved resilience to failures compared to the previous generation. The Mv3 VHM sizes offer 32TB of memory, up to 8 GBps of throughput for remote storage, and provide up to 185 Gbps of networking performance with the current VHM generation.

## Feature support

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported<br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported<br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[VM Generation Support](/azure/virtual-machines/generation-2): Generation 2<br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>[Ephemeral OS Disks](/azure/virtual-machines/ephemeral-os-disks): Supported<br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported<br>[Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Sizes in series (NVMe)

| **Size**<sup>1</sup> | **vCPU** | **Memory: GiB** | **Max Data Disks** | **Max temp storage throughput: IOPS/MBps** | **Max un-cached Premium** **SSD  throughput: IOPS/MBps** | **Max un-cached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **NIC's (max)** | **Max network bandwidth (Mbps)**  |
|---|---|---|---|---|---|---|---|---|
| **standard_m896ixds_32_v3**<sup>2</sup> | 896 | 30400 | 64 | 4096 | 110000/8000 | 200000/8000 | 8 | 185000 |
| **standard_m1792ixds_32_v3**<sup>3</sup> | 1792 | 30400 | 64 | 4096 | 110000/8000 | 200000/8000 | 8 | 185000 |

<sup>1</sup>VHM VM Sizes are virtual machine sizes that are Isolated to a specific hardware type and dedicated to a single customer.   
<sup>2</sup>The Standard_m896ixds_32_v3 VM comes without Simultaneous Multithreading (SMT), meaning a vCPU is mapped to a full physical core.   
<sup>3</sup>[Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.  

It's also important to note that these VMs are compatible with only certain generation 2 Images. For a list of images that are compatible with the Mdsv3-series, please see below
- Windows Server 2022 Datacenter Edition latest builds
- SUSE Linux enterprise Server 15 SP4 and later
- Red Hat Enterprise Linux 8.8 or later
- Ubuntu 23.10 or later

> [!IMPORTANT]
> Please contact your Azure Account Manager to inquire about accessing these VHM VM sizes.

## Size table definitions

Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.

Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.

Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to **ReadOnly** or **ReadWrite**. For uncached data disk operation, the host cache mode is set to **None**.

To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).

**Expected network bandwidth** is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).

Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

## Other sizes and information

- [General purpose](/azure/virtual-machines/sizes-general)
- [Memory optimized](/azure/virtual-machines/sizes-memory)
- [Storage optimized](/azure/virtual-machines/sizes-storage)
- [GPU optimized](/azure/virtual-machines/sizes-gpu)
- [High performance compute](/azure/virtual-machines/sizes-hpc)
- [Previous generations](/azure/virtual-machines/sizes-previous-gen)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

More information on Disks Types: [Disk Types](/azure/virtual-machines/disks-types)
