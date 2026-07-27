---
title: Lsv2 size series
description: Information on and specifications of the Lsv2-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As an IT infrastructure planner, I want to review the specifications and feature support of the Lsv2-series virtual machines, so that I can determine the appropriate VM size for my organization's performance and storage requirements."
---

# Lsv2 sizes series

[!INCLUDE [lsv2-summary](./includes/lsv2-series-summary.md)]

## Host specifications
[!INCLUDE [lsv2-series-specs](./includes/lsv2-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_L8s_v2 | 8 | 64 |
| Standard_L16s_v2 | 16 | 128 |
| Standard_L32s_v2 | 32 | 256 |
| Standard_L48s_v2 | 48 | 384 |
| Standard_L64s_v2 | 64 | 512 |
| Standard_L80s_v26 | 80 | 640 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Max NVMe Disks (Qty.) | NVMe Disk Size (TiB) | NVMe Disk IOPS | NVMe Disk Throughput (MBps) | 
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L8s_v2  | 1 | 80  | 1  | 1.92 | 400,000 | 2,000 |
| Standard_L16s_v2 | 1 | 160 | 2  | 1.92 | 800,000 | 4,000 |
| Standard_L32s_v2 | 1 | 320 | 4  | 1.92 | 1,500,000   | 8,000 |
| Standard_L48s_v2 | 1 | 480 | 6  | 1.92 | 2,200,000   | 14,000 |
| Standard_L64s_v2 | 1 | 640 | 8  | 1.92 | 2,900,000   | 16,000 |
| Standard_L80s_v2 | 1 | 800 | 10 | 1.92 | 3,800,000   | 20,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Additionally, temp disk performance often differs between read and write operations. During steady state operations, write performance is expected to be lower than read performance.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L8s_v2 | 16 | 8,000 | 160 | 8,000 | 1,280 | 400,000 | 2,000 |
| Standard_L16s_v2 | 32 | 16,000 | 320 | 16,000 | 1,280 | 800,000 | 4,000 |
| Standard_L32s_v2 | 32 | 32,000 | 640 | 32,000 | 1,280 | 1,500,000 | 8,000 |
| Standard_L48s_v2 | 32 | 48,000 | 960 | 48,000 | 2,000 | 2,200,000 | 14,000 |
| Standard_L64s_v2 | 32 | 64,000 | 1,280 | 64,000 | 2,000 | 2,900,000 | 16,000 |
| Standard_L80s_v2 | 32 | 80,000 | 1,400 | 80,000 | 2,000 | 3,800,000 | 20,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_L8s_v2 | 2 | 3,200 |
| Standard_L16s_v2 | 4 | 6,400 |
| Standard_L32s_v2 | 8 | 12,800 |
| Standard_L48s_v2 | 8 | 16,000+ |
| Standard_L64s_v2 | 8 | 16,000+ |
| Standard_L80s_v26 | 8 | 16,000+ |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

## Feature support

|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Not Supported |
|[Live Migration](../../maintenance-and-updates.md)| Not Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Not Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Not Supported |


[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]

