---
title: HBv2 size series
description: Information on and specifications of the HBv2-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/24/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and feature support of the HBv2 virtual machine sizes, so that I can select the appropriate VM type for my high-performance computing projects."
---

# HBv2 sizes series

[!INCLUDE [hbv2-summary](./includes/hbv2-series-summary.md)]

## Host specifications
[!INCLUDE [hbv2-series-specs](./includes/hbv2-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br> [Backend Network](../../hbv2-series-overview.md): InfiniBand HDR

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) | L3 Cache (MB) | Memory Bandwidth (GB/s) | Base CPU Frequency (GHz) | Single-core Frequency Peak (GHz) | All-core Frequency Peak (GHz) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB120rs_v2 | 120 | 456 | 512 | 350 | 2.45 | 3.3 | 3.1 |
| Standard_HB120-96rs_v2 | 96 | 456 | 512 | 350 | 2.45 | 3.3 | 3.1 |
| Standard_HB120-64rs_v2 | 64 | 456 | 512 | 350 | 2.45 | 3.3 | 3.1 |
| Standard_HB120-32rs_v2 | 32 | 456 | 512 | 350 | 2.45 | 3.3 | 3.1 |
| Standard_HB120-16rs_v2 | 16 | 456 | 512 | 350 | 2.45 | 3.3 | 3.1 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Local Solid State Disks (Qty.) | Local Solid State Disk Size (GiB) |
| --- | --- | --- | --- | --- |
| Standard_HB120rs_v2 | 1 | 480 | 1 | 960 |
| Standard_HB120-96rs_v2 | 1 | 480 | 1 | 960 |
| Standard_HB120-64rs_v2 | 1 | 480 | 1 | 960 |
| Standard_HB120-32rs_v2 | 1 | 480 | 1 | 960 |
| Standard_HB120-16rs_v2 | 1 | 480 | 1 | 960 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) |
| --- | --- |
| Standard_HB120rs_v2 | 8 |
| Standard_HB120-96rs_v2 | 8 |
| Standard_HB120-64rs_v2 | 8 |
| Standard_HB120-32rs_v2 | 8 |
| Standard_HB120-16rs_v2 | 8 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

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
| Standard_HB120rs_v2 | 8 |  40000 |
| Standard_HB120-96rs_v2 | 8  | 40000 |
| Standard_HB120-64rs_v2 | 8  | 40000 |
| Standard_HB120-32rs_v2 | 8  | 40000 |
| Standard_HB120-16rs_v2 | 8  | 40000 |

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
| Standard_HB120rs_v2 | 1 | 200 |
| Standard_HB120-96rs_v2 | 1 | 200 |
| Standard_HB120-64rs_v2 | 1 | 200 |
| Standard_HB120-32rs_v2 | 1 | 200 |
| Standard_HB120-16rs_v2 | 1 | 200 |

#### Backend Networking resources
- [Set up Infiniband on HPC VMs](/azure/virtual-machines/setup-infiniband)


### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer-hpc](../includes/sizes-footer-hpc.md)]

