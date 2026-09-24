---
title: Lasv5 size series
description: Information on and specifications of the Lasv5-series sizes
author: zhousarah
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
---

# Lasv5 size series

[!INCLUDE [lasv5-summary](./includes/lasv5-series-summary.md)]

## Host specifications
[!INCLUDE [lasv5-series-specs](./includes/lasv5-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs and memory for each size.

| Size Name | vCPUs | Memory (GiB) |
| --- | --- | --- |
| Standard_L2as_v5 | 2 | 16 |
| Standard_L4as_v5 | 4 | 32 |
| Standard_L8as_v5 | 8 | 64 |
| Standard_L16as_v5 | 16 | 128 |
| Standard_L32as_v5 | 32 | 256 |
| Standard_L48as_v5 | 48 | 384 |
| Standard_L64as_v5 | 64 | 512 |
| Standard_L80as_v5 | 80 | 640 |
| Standard_L96as_v5 | 96 | 768 |
| Standard_L128as_v5 | 128 | 1,024 |
| Standard_L160ias_v5 | 160 | 1,280 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage information for each size.

| Size Name | Temp Storage Disks | Temp Disk Size (GB) | Temp Disk Random Read IOPS | Temp Disk Sequential Read Throughput (MBps) | Temp Disk Random Write IOPS | Temp Disk Sequential Write Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L2as_v5 | 1 | 480 | 150,000 | 750 | 75,000 | 375 |
| Standard_L4as_v5 | 1 | 960 | 300,000 | 1,500 | 150,000 | 750 |
| Standard_L8as_v5 | 1 | 1,920 | 600,000 | 3,000 | 300,000 | 1,500 |
| Standard_L16as_v5 | 1 | 3,840 | 1,200,000 | 6,000 | 600,000 | 3,000 |
| Standard_L32as_v5 | 2 | 3,840 | 2,400,000 | 12,000 | 1,200,000 | 6,000 |
| Standard_L48as_v5 | 3 | 3,840 | 3,600,000 | 18,000 | 1,800,000 | 9,000 |
| Standard_L64as_v5 | 4 | 3,840 | 4,800,000 | 24,000 | 2,400,000 | 12,000 |
| Standard_L80as_v5 | 5 | 3,840 | 6,000,000 | 30,000 | 3,000,000 | 15,000 |
| Standard_L96as_v5 | 6 | 3,840 | 7,200,000 | 36,000 | 3,600,000 | 18,000 |
| Standard_L128as_v5 | 8 | 3,840 | 9,600,000 | 48,000 | 4,800,000 | 24,000 |
| Standard_L160ias_v5 | 8 | 3,840 | 9,600,000 | 48,000 | 4,800,000 | 24,000 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)

#### Table definitions
- Temp disk performance depends on many factors, including block size, workload patterns of read and write operations, queue depth (QD), and others. View temp disk performance specifications as best-case performance numbers, assuming 4 KB block sizes and QD=256 for IOPS, and 256 KB block sizes with QD=64 for throughput. Write performance is heavily impacted by how many blocks are in use on a device. Temp disk write performance specifications assume a device has a clean slate to enable the best performance. During steady-state operations, write performance is lower than the published specifications.
- For Lasv5 and Lasv4, temp disk refers to the NVMe local data disks used by the VM. While Lsv3 and Lasv3 have NVMe local data disks and a SCSI local temp disk, Lasv5 and Lasv4 only have NVMe local temp disks. There's no SCSI local temp disk on Lasv5.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps, where MBps = 10^6 bytes/sec.
- To learn how to get the best local storage performance for your VMs, see the [NVMe Temp Disk FAQ](/azure/virtual-machines/enable-nvme-temp-faqs).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage information for each size.
| Size Name | Max Remote Storage Disks | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst Ultra Disk and Premium SSD v2 IOPS | Uncached Burst Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L2as_v5 | 4 | 4,000 | 118 | 44,000 | 1,413 | 4,400 | 137 | 48,400 | 1,653 |
| Standard_L4as_v5 | 8 | 8,000 | 234 | 47,200 | 1,413 | 8,800 | 274 | 52,083 | 1,653 |
| Standard_L8as_v5 | 16 | 16,000 | 468 | 47,200 | 1,413 | 17,600 | 548 | 52,083 | 1,653 |
| Standard_L16as_v5 | 32 | 32,000 | 936 | 72,700 | 1,413 | 35,200 | 1,096 | 80,000 | 1,653 |
| Standard_L32as_v5 | 32 | 64,000 | 1,872 | 94,400 | 1,916 | 70,400 | 2,191 | 104,167 | 2,242 |
| Standard_L48as_v5 | 32 | 96,000 | 2,808 | 99,000 | 2,875 | 105,600 | 3,291 | 108,900 | 3,363 |
| Standard_L64as_v5 | 32 | 128,000 | 3,744 | 132,000 | 3,833 | 140,800 | 4,382 | 145,200 | 4,485 |
| Standard_L80as_v5 | 32 | 160,000 | 4,704 | 162,500 | 4,791 | 176,000 | 5,478 | 178,475 | 5,577 |
| Standard_L96as_v5 | 32 | 192,000 | 5,664 | 192,500 | 5,749 | 211,200 | 6,574 | 211,750 | 6,669 |
| Standard_L128as_v5 | 32 | 204,800 | 7,488 | 225,280 | 7,664 | 281,600 | 8,765 | 310,886 | 8,967 |
| Standard_L160ias_v5 | 32 | 260,000 | 12,000 | 260,000 | 12,000 | 400,000 | 12,000 | 400,000 | 12,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB might appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps, where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface information for each size.

| Size Name | Max NICs | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_L2as_v5 | 2 | 16,000 |
| Standard_L4as_v5 | 2 | 16,000 |
| Standard_L8as_v5 | 4 | 25,000 |
| Standard_L16as_v5 | 8 | 25,000 |
| Standard_L32as_v5 | 8 | 25,000 |
| Standard_L48as_v5 | 8 | 35,000 |
| Standard_L64as_v5 | 8 | 45,000 |
| Standard_L80as_v5 | 8 | 57,500 |
| Standard_L96as_v5 | 8 | 70,000 |
| Standard_L128as_v5 | 15 | 75,000 |
| Standard_L160ias_v5 | 15 | 200,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance depends on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth).
- To achieve the expected network performance on Linux or Windows, you might need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, and other accelerators) information for each size.

> [!NOTE]
> This series doesn't include any accelerators.

---

## Feature support

|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Supported (except for L96as_v5) |
|[Live Migration](../../maintenance-and-updates.md)| Not Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Supported |

> [!NOTE]
> This VM series works only on OS images that support NVMe. If your current OS image doesn't support NVMe, you see an error message. [NVMe](/azure/virtual-machines/enable-nvme-interface) support is available on the most popular OS images, and Microsoft is continuously improving OS image compatibility.

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
