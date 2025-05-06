---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: Mdsv3 Very High Memory Series
description: Overview of Mdsv3 Very High Memory Virtual Machines
author:      phgantik # GitHub alias
ms.author:   phgantik # Microsoft alias
ms.service: virtual-machines-windows
ms.topic: concept-article
ms.date:     03/18/2025
---

# Mdsv3 Very High Memory Series

The Mdsv3 Very High Memory (VHM) series, powered by 4<sup>th</sup> generation Intel® Xeon® Platinum 8490H (Sapphire Rapids) processor with an all core base frequency of 1.9 GHz and 3.5 GHz single core turbo frequency, are the next generation of memory-optimized VM sizes delivering faster performance, lower total cost of ownership and improved resilience to failures compared to the previous generation. The Mv3 VHM sizes offer up to 32TB of memory, up to 8 GBps of throughput for remote storage, and provide up to 185 Gbps of networking performance with the current VHM generation.

## Feature support

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported<br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported<br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[VM Generation Support](/azure/virtual-machines/generation-2): Generation 2<br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>[Ephemeral OS Disks](/azure/virtual-machines/ephemeral-os-disks): Supported<br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported<br>[Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Sizes in series (NVMe)

| **Size**<sup>1</sup> | **vCPU** | **Memory: GiB** | **Max Data Disks** | **Max temp storage throughput: IOPS/MBps** | **Max un-cached Premium** **SSD  throughput: IOPS/MBps** | **Max un-cached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **NIC's (max)** | **Max network bandwidth (Mbps)**  |
|---|---|---|---|---|---|---|---|---|
| Standard_M896ixds_32_v3<sup>2</sup> | 896 | 30400 | 64 | 4096 | 110000/8000 | 200000/8000 | 8 | 185000 |
| Standard_M896ixds_24_v3<sup> | 896| 23088| 64 | 4096 | 110000/8000 | 200000/8000 | 8 | 185000 |
| Standard_M1792ixds_32_v3<sup>3</sup> | 1792 | 30400 | 64 | 4096 | 110000/8000 | 200000/8000 | 8 | 185000 |

<sup>1</sup> VHM VM Sizes are virtual machine sizes that are Isolated to a specific hardware type and dedicated to a single customer.<br>   
<sup>2</sup> The Standard_M896ixds_32_v3 VM is the Microsoft recommended VM type with 32TB to host S/4HANA workload. This VM type has Simultaneous Multithreading (SMT) disabled. With that complies with the SAP recommendations stated in SAP note #2711650 for the specific underlying hardware used in Azure to host this Virtual Machine (VM) type. With typical S/4HANA workload, tests this VM realized the best performance.<br>   
<sup>3</sup> The Standard_M1792ixds_32_v3 VM has Simultaneous Multithreading (SMT) enabled and is ideal for analytical workloads as documented in SAP note #2711650 for the specific underlying hardware used in Azure to host this VM type. Typical S/4HANA workloads may show performance regressions compared to the Standard_M896ixds_32_v3 VM type. It is acknowledged that S/4HANA customer workloads are varying and can be different in nature. As a result, there might be cases where S/4HANA customer workloads could eventually benefit. And the Standard_M1792ixds_32_v3 VM type could provide performance improvements compared to the Standard_M896ixds_32_v3 VM type for a customer specific S/4HANA workload. Evaluating and hosting S/4HANA workload on Standard_M1792ixds_32_v3 is in the customer’s own responsibilities.  

It's also important to note that these VMs are compatible with only certain generation 2 Images. For a list of images that are compatible with the Mdsv3-series, please see below
- Windows Server 2022 Datacenter Edition latest builds
- SUSE Linux enterprise Server 15 SP4 and later
- Red Hat Enterprise Linux 8.8 or later
- Ubuntu 23.10 or later

> [!IMPORTANT]
> Contact your Azure Account Manager to inquire about accessing these VHM VM sizes.

## Size table definitions

Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.

Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.

Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to **ReadOnly** or **ReadWrite**. For uncached data disk operation, the host cache mode is set to **None**.

To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).

**Expected network bandwidth** is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).

Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance depends on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

## Other sizes and information

- [General purpose](/azure/virtual-machines/sizes-general)
- [Memory optimized](/azure/virtual-machines/sizes-memory)
- [Storage optimized](/azure/virtual-machines/sizes-storage)
- [GPU optimized](/azure/virtual-machines/sizes-gpu)
- [High performance compute](/azure/virtual-machines/sizes-hpc)
- [Previous generations](/azure/virtual-machines/sizes-previous-gen)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

More information on Disks Types: [Disk Types](/azure/virtual-machines/disks-types)
