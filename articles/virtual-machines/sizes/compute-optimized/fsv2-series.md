---
title: Fsv2-series sizes
description: Information on and specifications of the Fsv2-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/30/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to compare the specifications of the Fsv2 series virtual machine sizes, so that I can select the appropriate VM size for my application's performance and resource requirements.
---

# Fsv2-series

[!INCLUDE [fsv2-summary](./includes/fsv2-series-summary.md)]

## Host specifications for the Fsv2-series
[!INCLUDE [fsv2-series-specs](./includes/fsv2-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in the Fsv2-series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size in the Fsv2-series.

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_F2s_v2 | 2 | 4 |
| Standard_F4s_v2 | 4 | 8 |
| Standard_F8s_v2 | 8 | 16 |
| Standard_F16s_v2 | 16 | 32 |
| Standard_F32s_v2 | 32 | 64 |
| Standard_F48s_v2 | 48 | 96 |
| Standard_F64s_v2 | 64 | 128 |
| Standard_F72s_v2 | 72 | 144 |

#### Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size in the Fsv2-series.

| Size Name | Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read IOPS | Temp Disk Sequential Read Throughput (MBps) |
| --- | --- | --- | --- | --- |
| Standard_F2s_v2 | 1 | 16 | 4,000 | 31 |
| Standard_F4s_v2 | 1 | 32 | 8,000 | 63 |
| Standard_F8s_v2 | 1 | 64 | 16,000 | 127 |
| Standard_F16s_v2 | 1 | 128 | 32,000 | 255 |
| Standard_F32s_v2 | 1 | 256 | 64,000 | 512 |
| Standard_F48s_v2 | 1 | 384 | 96,000 | 768 |
| Standard_F64s_v2 | 1 | 512 | 128,000 | 1,024 |
| Standard_F72s_v2 | 1 | 576 | 144,000 | 1,152 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Fsv2-series local storage table definitions
- Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Additionally, temp disk performance often differs between read and write operations. During steady state operations, write performance is expected to be lower than read performance.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size in the Fsv2-series.

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) |
| --- | --- | --- | --- | --- | --- |
| Standard_F2s_v2 | 4 | 3,200 | 47 | 4,000 | 200 |
| Standard_F4s_v2 | 8 | 6,400 | 95 | 8,000 | 200 |
| Standard_F8s_v2 | 16 | 12,800 | 190 | 16,000 | 400 |
| Standard_F16s_v2 | 32 | 25,600 | 380 | 32,000 | 800 |
| Standard_F32s_v2 | 32 | 51,200 | 750 | 64,000 | 1,600 |
| Standard_F48s_v2 | 32 | 76,800 | 1,100 | 80,000 | 2,000 |
| Standard_F64s_v2 | 32 | 80,000 | 1,100 | 80,000 | 2,000 |
| Standard_F72s_v2 | 32 | 80,000 | 1,100 | 80,000 | 2,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Fsv2-series remote storage table definitions
- Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size in the Fsv2-series.

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_F2s_v2 | 2 | 5,000 |
| Standard_F4s_v2 | 2 | 10,000 |
| Standard_F8s_v2 | 4 | 12,500 |
| Standard_F16s_v2 | 4 | 12,500 |
| Standard_F32s_v2 | 8 | 16,000 |
| Standard_F48s_v2 | 8 | 21,000 |
| Standard_F64s_v2 | 8 | 28,000 |
| Standard_F72s_v2 | 8 | 30,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Fsv2-series network table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size in the Fsv2-series.

> [!NOTE]
> No accelerators are present in this series.

---

## Feature support

|Feature name | Support status | 
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md) |  Supported |
|[Premium Storage caching](../../premium-storage-performance.md) |  Supported |
|[Live Migration](../../maintenance-and-updates.md) |  Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md) |  Supported |
|[Generation 2 VMs](../../generation-2.md) |  Supported |
|[Generation 1 VMs](../../generation-2.md) |  Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli) |  Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md) |  Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) |  Supported |


[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]

