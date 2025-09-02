---
title: Dasv4 size series
description: Information on and specifications of the Dasv4-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to review the specifications and features of the Dasv4 size series, so that I can choose the appropriate virtual machine type to optimize performance for my applications.
---

# Dasv4 sizes series

[!INCLUDE [dasv4-summary](./includes/dasv4-series-summary.md)]

## Host specifications
[!INCLUDE [dasv4-series-specs](./includes/dasv4-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2as_v4 | 2 | 8 |
| Standard_D4as_v4 | 4 | 16 |
| Standard_D8as_v4 | 8 | 32 |
| Standard_D16as_v4 | 16 | 64 |
| Standard_D32as_v4 | 32 | 128 |
| Standard_D48as_v4 | 48 | 192 |
| Standard_D64as_v4 | 64 | 256 |
| Standard_D96as_v4 | 96 | 384 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) | Temp Disk Random Write (RW)<sup>1</sup> IOPS | Temp Disk Random Write (RW)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2as_v4 | 4 | 16 | 4000 | 32 | 4000 | 100 |
| Standard_D4as_v4 | 8 | 32 | 8000 | 64 | 8000 | 200 |
| Standard_D8as_v4 | 16 | 64 | 16000 | 128 | 16000 | 400 |
| Standard_D16as_v4 | 32 | 128 | 32000 | 255 | 32000 | 800 |
| Standard_D32as_v4 | 32 | 256 | 64000 | 510 | 64000 | 1600 |
| Standard_D48as_v4 | 32 | 384 | 96000 | 1020 | 96000 | 2000 |
| Standard_D64as_v4 | 32 | 512 | 128000 | 1020 | 128000 | 2000 |
| Standard_D96as_v4 | 32 | 768 | 192000 | 1020 | 192000 | 2000 |

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
| Standard_D2as_v4 | 4 | 3200 | 48 | 4000 | 200 |
| Standard_D4as_v4 | 8 | 6400 | 96 | 8000 | 200 |
| Standard_D8as_v4 | 16 | 12800 | 192 | 16000 | 400 |
| Standard_D16as_v4 | 32 | 25600 | 384 | 32000 | 800 |
| Standard_D32as_v4 | 32 | 51200 | 768 | 64000 | 1600 |
| Standard_D48as_v4 | 32 | 76800 | 1148 | 80000 | 2000 |
| Standard_D64as_v4 | 32 | 80000 | 1200 | 80000 | 2000 |
| Standard_D96as_v4 | 32 | 80000 | 1200 | 80000 | 2000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D2as_v4 | 2 | 2000 |
| Standard_D4as_v4 | 2 | 4000 |
| Standard_D8as_v4 | 4 | 8000 |
| Standard_D16as_v4 | 8 | 10000 |
| Standard_D32as_v4 | 8 | 16000 |
| Standard_D48as_v4 | 8 | 24000 |
| Standard_D64as_v4 | 8 | 32000 |
| Standard_D96as_v4 | 8 | 40000 |

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


