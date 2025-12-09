---
title: HX size series
description: Information on and specifications of the HX-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to understand the specifications and features of the HX series VM sizes, so that I can effectively select the appropriate size for my workloads based on performance and storage requirements.
---

# HX sizes series

[!INCLUDE [hx-summary](./includes/hx-series-summary.md)]

## Host specifications
[!INCLUDE [hx-series-specs](./includes/hx-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br> [Backend Network](../../hx-series-overview.md#infiniband-networking): InfiniBand NDR

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) | L3 Cache (MB) | Memory Bandwidth (GB/s) | Base CPU Frequency (GHz) |  Single-core Frequency Peak (GHz) | All-core Frequency Peak (GHz) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HX176rs | 176 | 1408 | 2304 | 780 | 2.55 | 3.7 | 3.7 |
| Standard_HX176-144rs | 144 | 1408 | 2304 | 780 | 2.55 | 3.7 | 3.7 |
| Standard_HX176-96rs | 96 | 1408 | 2304 | 780 | 2.55 | 3.7 | 3.7 |
| Standard_HX176-48rs | 48 | 1408 | 2304 | 780 | 2.55 | 3.7 | 3.7 |
| Standard_HX176-24rs | 24 | 1408 | 2304 | 780 | 2.55 | 3.7 | 3.7 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Local Solid State Disks (Qty.) | Local Solid State Disk Size (GiB) |
| --- | --- | --- | --- | --- |
| Standard_HX176rs | 1 | 480 | 2 | 1800 |
| Standard_HX176-144rs | 1 | 480 | 2 | 1800 |
| Standard_HX176-96rs | 1 | 480 | 2 | 1800 |
| Standard_HX176-48rs | 1 | 480 | 2 | 1800 |
| Standard_HX176-24rs | 1 | 480 | 2 | 1800 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) |
| --- | --- |
| Standard_HX176rs | 32 |
| Standard_HX176-144rs | 32 |
| Standard_HX176-96rs | 32 |
| Standard_HX176-48rs | 32 |
| Standard_HX176-24rs | 32 |

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
| Standard_HX176rs | 8 |  80000 |
| Standard_HX176-144rs | 8 |  80000 |
| Standard_HX176-96rs | 8 | 80000 |
| Standard_HX176-48rs | 8 | 80000 |
| Standard_HX176-24rs | 8 | 80000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).


### [Backend Network](#tab/sizebacknetwork)

Network interface info for each size

| Size Name | Backend NICs (Qty.) | RDMA Performance (Gb/s) |
| --- | --- | --- |
| Standard_HX176rs | 1 | 400 |
| Standard_HX176-144rs | 1 | 400 |
| Standard_HX176-96rs | 1 | 400 |
| Standard_HX176-48rs | 1 | 400 |
| Standard_HX176-24rs | 1 | 400 |

#### Backend Networking resources
- [Set up Infiniband on HPC VMs](/azure/virtual-machines/setup-infiniband)


### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]

