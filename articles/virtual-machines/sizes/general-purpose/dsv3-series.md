---
title: Dsv3 size series
description: Information on and specifications of the Dsv3-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 09/24/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
---

# Dsv3 sizes series
[!INCLUDE [previous-gen-header](../includes/sizes-previous-gen-header.md)]

[!INCLUDE [dsv3-summary](./includes/dsv3-series-summary.md)]

## Host specifications
[!INCLUDE [dsv3-series-specs](./includes/dsv3-series-specs.md)]

## Feature support

Premium Storage: Supported<br>
Premium Storage caching: Supported<br>
Live Migration: Supported<br>
Memory Preserving Updates: Supported<br>
VM Generation Support: Generation 1 and 2<br>
Accelerated Networking: Supported<br>
Ephemeral OS Disks: Supported<br>
Nested Virtualization: Supported<br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2s_v3 | 2 | 8 |
| Standard_D4s_v3 | 4 | 16 |
| Standard_D8s_v3 | 8 | 32 |
| Standard_D16s_v3 | 16 | 64 |
| Standard_D32s_v3 | 32 | 128 |
| Standard_D48s_v3 | 48 | 192 |
| Standard_D64s_v3 | 64 | 256 |

#### VM Basics resources
- [What are vCPUs](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (cached & temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Disk Cache Size (GiB) | Cached Disk Random Read (RR)<sup>1</sup> IOPS | Cached Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_D2s_v3 | 1 | 16   | 50   | 4000 | 32 |
| Standard_D4s_v3 | 1 | 32   | 100  | 8000 | 64 |
| Standard_D8s_v3 | 1 | 64   | 200  | 16000 | 128 |
| Standard_D16s_v3 | 1 | 128 | 400  | 32000 | 256 |
| Standard_D32s_v3 | 1 | 256 | 800  | 64000 | 512 |
| Standard_D48s_v3 | 1 | 384 | 1200 | 96000 | 768 |
| Standard_D64s_v3 | 1 | 512 | 1600 |128000 | 1024 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_D2s_v3 | 4 | 3200 | 48 | 4000 | 200 |
| Standard_D4s_v3 | 8 | 6400 | 96 | 8000 | 200 |
| Standard_D8s_v3 | 16 | 12800 | 192 | 16000 | 400 |
| Standard_D16s_v3 | 32 | 25600 | 384 | 32000 | 800 |
| Standard_D32s_v3 | 32 | 51200 | 768 | 64000 | 1600 |
| Standard_D48s_v3 | 32 | 76800 | 1152 | 80000 | 2000 |
| Standard_D64s_v3 | 32 | 80000 | 1200 | 80000 | 2000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D2s_v3 | 2 | 1000 |
| Standard_D4s_v3 | 2 | 2000 |
| Standard_D8s_v3 | 4 | 2000 |
| Standard_D16s_v3 | 8 | 2000 |
| Standard_D32s_v3 | 8 | 16000 |
| Standard_D48s_v3 | 8 | 24000 |
| Standard_D64s_v3 | 8 | 30000 |

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

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]


