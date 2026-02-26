---
title: HBv4 size series
description: Information on and specifications of the HBv4-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/24/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and features of the HBv4 virtual machine sizes, so that I can choose the right model for my application's performance and resource requirements."
---

# HBv4 sizes series

[!INCLUDE [hbv4-summary](./includes/hbv4-series-summary.md)]

## Host specifications
[!INCLUDE [hbv4-series-specs](./includes/hbv4-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br> [Backend Network](../../hbv4-series-overview.md#infiniband-networking): InfiniBand NDR

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) | L3 Cache (MB) | Memory Bandwidth (GB/s) | Base CPU Frequency (GHz) |  Single-core Frequency Peak (GHz) | All-core Frequency Peak (GHz) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB176rs_v4 | 176 | 768 | 2304 | 780 | 2.55 | 3.7 | 3.7 | 
| Standard_HB176-144rs_v4 | 144 | 768 | 2304 | 780 | 2.55 |  3.7 | 3.7 |
| Standard_HB176-96rs_v4 | 96 | 768 | 2304 | 780 | 2.55 | 3.7 | 3.7 |
| Standard_HB176-48rs_v4 | 48 | 768 | 2304 | 780 | 2.55 |  3.7 | 3.7 |
| Standard_HB176-24rs_v4 | 24 | 768 | 2304 | 780 | 2.55 |  3.7 | 3.7 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Local Solid State Disks (Qty.) | Local Solid State Disk Size (GiB) |
| --- | --- | --- | --- | --- |
| Standard_HB176rs_v4 | 1 | 480 | 2 | 1800 |
| Standard_HB176-144rs_v4 | 1 | 480 | 2 | 1800 |
| Standard_HB176-96rs_v4 | 1 | 480 | 2 | 1800 |
| Standard_HB176-48rs_v4 | 1 | 480 | 2 | 1800 |
| Standard_HB176-24rs_v4 | 1 | 480 | 2 | 1800 |

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
| Standard_HB176rs_v4 | 32 |
| Standard_HB176-144rs_v4 | 32 |
| Standard_HB176-96rs_v4 | 32 |
| Standard_HB176-48rs_v4 | 32 |
| Standard_HB176-24rs_v4 | 32 |

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
| Standard_HB176rs_v4 | 8 | 80000 |
| Standard_HB176-144rs_v4 | 8 | 80000 |
| Standard_HB176-96rs_v4 | 8 | 80000 |
| Standard_HB176-48rs_v4 | 8 | 80000 |
| Standard_HB176-24rs_v4 | 8 | 80000 |

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
| Standard_HB176rs_v4 | 1 | 400 |
| Standard_HB176-144rs_v4 | 1 | 400 |
| Standard_HB176-96rs_v4 | 1 | 400 |
| Standard_HB176-48rs_v4 | 1 | 400 |
| Standard_HB176-24rs_v4 | 1 | 400 |

#### Backend Networking resources
- [Set up Infiniband on HPC VMs](/azure/virtual-machines/setup-infiniband)


### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]

