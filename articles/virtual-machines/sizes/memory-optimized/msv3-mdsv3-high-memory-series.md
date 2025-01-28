---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: Msv3 and Mdsv3 High Memory series
description: Overview of Msv3 and Mdsv3 High Memory virtual machines
author: leebanhassen
ms.author: leebanhassen
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.date:  10/15/2024
---

# Msv3 and Mdsv3 High Memory series

The Msv3 and Mdsv3 High Memory (HM) Virtual Machine (VM) series are the next generation of memory-optimized VM sizes delivering faster performance, lower total cost of ownership and improved resilience to failures compared to the previous generation, Mv2-series VMs. Mv3 HM offers VM sizes with memory ranging from 6TB to 16TB, up to 8,000 MBps throughout to remote storage, and up to 25% networking performance improvements over previous generations.

## Msv3 High Memory series

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported<br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported<br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[VM Generation Support](/azure/virtual-machines/generation-2): Generation 2<br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>[Ephemeral OS Disks](/azure/virtual-machines/ephemeral-os-disks): Not Supported<br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported<br>[Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Msv3 High Memory series (NVMe)

| **Size** | **vCPU** | **Memory: GiB** | **Max data disks** | **Max** **uncached Premium** **SSD  throughput: IOPS/MBps** | **Max** **uncached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **Max NICs** | **Max network bandwidth (Mbps)** |
|---|---|---|---|---|---|---|---|
| **Standard_M416s_6_v3** | 416 | 5,696 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M416s_8_v3** | 416 | 7,600 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M624s_12_v3**<sup>1</sup> | 624 | 11,400 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832s_12_v3**<sup>1</sup> | 832 | 11,400 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832is_16_v3**<sup>1</sup> | 832 | 15,200 | 64 | 130,000/8,000 | 260,000/8,000 | 8 | 40,000 |

<sup>1</sup> [Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.  

## Msv3 High Memory series (SCSI)

| **Size** | **vCPU** | **Memory: GiB** | **Max data disks** | **Max** **uncached Premium** **SSD  throughput: IOPS/MBps** | **Max** **uncached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **Max NICs** | **Max network bandwidth (Mbps)** |
|---|---|---|---|---|---|---|---|
| **Standard_M416s_6_v3** | 416 | 5,696 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M416s_8_v3** | 416 | 7,600 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M624s_12_v3**<sup>1</sup> | 624 | 11,400 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832s_12_v3**<sup>1</sup> | 832 | 11,400 | 64 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832is_16_v3**<sup>1</sup> | 832 | 15,200 | 64 | 130,000/8,000 | 130,000/8,000 | 8 | 40,000 |

<sup>1</sup> [Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.  

## Mdsv3 High Memory series

These virtual machines feature local SSD storage (up to 400 GiB).

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported<br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported<br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[VM Generation Support](/azure/virtual-machines/generation-2): Generation 2<br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>[Ephemeral OS Disks](/azure/virtual-machines/ephemeral-os-disks): Supported<br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported<br>[Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Mdsv3 High Memory series (NVMe)

| **Size** | **vCPU** | **Memory: GiB** | **Temp storage (SSD) GiB** | **Max data disks** | **Max temp storage throughput: IOPS/MBps*** | **Max uncached Premium** **SSD  throughput: IOPS/MBps** | **Max uncached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **Max NICs** | **Max network bandwidth (Mbps)** |
|---|---|---|---|---|---|---|---|---|---|
| **Standard_M416ds_6_v3** | 416 | 5,696 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M416ds_8_v3** | 416 | 7,600 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M624ds_12_v3**<sup>1</sup> | 624 | 11,400 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832ds_12_v3**<sup>1</sup>  | 832 | 11,400 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832ids_16_v3**<sup>1</sup> | 832 | 15,200 | 400 | 64 | 250,000/1,600 | 130,000/8,000 | 260,000/8,000 | 8 | 40,000 |

<sup>*</sup> Read iops is optimized for sequential reads   
<sup>1</sup> [Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.  

## Mdsv3 High Memory series (SCSI)

| **Size** | **vCPU** | **Memory: GiB** | **Temp storage (SSD) GiB** | **Max data disks** | **Max temp storage throughput: IOPS/MBps*** | **Max uncached Premium** **SSD  throughput: IOPS/MBps** | **Max uncached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **Max NICs** | **Max network bandwidth (Mbps)** |
|---|---|---|---|---|---|---|---|---|---|
| **Standard_M416ds_6_v3** | 416 | 5,696 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M416ds_8_v3** | 416 | 7,600 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M624ds_12_v3**<sup>1</sup> | 624 | 11,400 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M624ds_12_v3**<sup>1</sup> | 624 | 11,400 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832ds_12_v3**<sup>1</sup> | 832 | 11,400 | 400 | 64 | 250,000/1,600 | 130,000/4,000 | 130,000/4,000 | 8 | 40,000 |
| **Standard_M832ids_16_v3**<sup>1</sup> | 832 | 15,200 | 400 | 64 | 250,000/1,600 | 130,000/8,000 | 130,000/8,000 | 8 | 40,000 |

<sup>*</sup> Read iops is optimized for sequential reads   
<sup>1</sup> [Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.  

## Size table definitions

Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.

Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.

Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to **ReadOnly** or **ReadWrite**. For uncached data disk operation, the host cache mode is set to **None**.

To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).

**Expected network bandwidth** is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).

Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

## Other sizes and information

[General purpose](/azure/virtual-machines/sizes-general)

[Memory optimized](/azure/virtual-machines/sizes-memory)

[Storage optimized](/azure/virtual-machines/sizes-storage)

[GPU optimized](/azure/virtual-machines/sizes-gpu)

[High performance compute](/azure/virtual-machines/sizes-hpc)

[Previous generations](/azure/virtual-machines/sizes-previous-gen)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

More information on Disks Types: [Disk Types](/azure/virtual-machines/disks-types)

