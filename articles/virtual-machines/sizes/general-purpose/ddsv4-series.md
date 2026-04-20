---
title: Ddsv4 size series
description: Information on and specifications of the Ddsv4-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and capabilities of the Ddsv4 virtual machine sizes, so that I can select the appropriate size for my application's performance and resource requirements."
---

# Ddsv4 sizes series

[!INCLUDE [ddsv4-summary](./includes/ddsv4-series-summary.md)]

## Host specifications
[!INCLUDE [ddsv4-series-specs](./includes/ddsv4-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2ds_v4 | 2 | 8 |
| Standard_D4ds_v4 | 4 | 16 |
| Standard_D8ds_v4 | 8 | 32 |
| Standard_D16ds_v4 | 16 | 64 |
| Standard_D32ds_v4 | 32 | 128 |
| Standard_D48ds_v4 | 48 | 192 |
| Standard_D64ds_v4 | 64 | 256 |

#### VM Basics resources
- [What are vCPUs](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MBps) |
| --- | --- | --- | --- | --- |
| Standard_D2ds_v4 | 1 | 75 | 9,000 | 125 |
| Standard_D4ds_v4 | 1 | 150 | 19,000 | 250 |
| Standard_D8ds_v4 | 1 | 300 | 38,000 | 500 |
| Standard_D16ds_v4 | 1 | 600 | 85,000 | 1,000 |
| Standard_D32ds_v4 | 1 | 1,200 | 150,000 | 2000 |
| Standard_D48ds_v4 | 1 | 1,800 | 225,000 | 3,000 |
| Standard_D64ds_v4 | 1 | 2,400 | 300,000 | 4,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) |
| --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v4 | 4 | 3,200 | 48 | 4,000 | 200 |
| Standard_D4ds_v4 | 8 | 6,400 | 96 | 8,000 | 200 |
| Standard_D8ds_v4 | 16 | 12,800 | 192 | 16,000 | 400 |
| Standard_D16ds_v4 | 32 | 25,600 | 384 | 32,000 | 800 |
| Standard_D32ds_v4 | 32 | 51,200 | 768 | 64,000 | 1,600 |
| Standard_D48ds_v4 | 32 | 76,800 | 1,152 | 80,000 | 2000 |
| Standard_D64ds_v4 | 32 | 80,000 | 1,200 | 80,000 | 2000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D2ds_v4 | 2 | 5,000 |
| Standard_D4ds_v4 | 2 | 10,000 |
| Standard_D8ds_v4 | 4 | 12,500 |
| Standard_D16ds_v4 | 8 | 12,500 |
| Standard_D32ds_v4 | 8 | 16,000 |
| Standard_D48ds_v4 | 8 | 24,000 |
| Standard_D64ds_v4 | 8 | 30,000 |

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
|[Premium Storage caching](../../premium-storage-performance.md)| Supported |
|[Live Migration](../../maintenance-and-updates.md#live-migration)| Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Supported |
|[VM Generation Support](../../generation-2.md)| Generation 1 and 2 |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disks](../../ephemeral-os-disks.md)| Supported |
|[Local temporary storage](../../overview.md#local-temporary-storage)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]


