---
title: Laosv5 size series
description: Information on and specifications of the Laosv5-series sizes
author: zhousarah
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
---

# Laosv5 size series

[!INCLUDE [laosv5-summary](./includes/laosv5-series-summary.md)]

## Host specifications
[!INCLUDE [laosv5-series-specs](./includes/laosv5-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs and memory for each size.

| Size Name | vCPUs | Memory (GiB) |
| --- | --- | --- |
| Standard_L2aos_v5 | 2 | 16 |
| Standard_L4aos_v5 | 4 | 32 |
| Standard_L8aos_v5 | 8 | 64 |
| Standard_L12aos_v5 | 12 | 96 |
| Standard_L16aos_v5 | 16 | 128 |
| Standard_L24aos_v5 | 24 | 192 |
| Standard_L32aos_v5 | 32 | 256 |
| Standard_L48aos_v5 | 48 | 384 |
| Standard_L64aos_v5 | 64 | 512 |
| Standard_L96aos_v5 | 96 | 768 |
| Standard_L128aos_v5 | 128 | 1024 |
| Standard_L160iaos_v5 | 160 | 1040 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage information for each size.

| Size Name | Temp Storage Disks | Temp Disk Size (GB) | Temp Disk Random Read IOPS | Temp Disk Sequential Read Throughput (MBps) | Temp Disk Random Write IOPS | Temp Disk Sequential Write Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L2aos_v5 | 1 | 1,440 | 215,625 | 1,220 | 107,813 | 565 |
| Standard_L4aos_v5 | 1 | 2,880 | 431,250 | 2,440 | 215,625 | 1,125 |
| Standard_L8aos_v5 | 1 | 5,760 | 862,500 | 4,875 | 431,250 | 2,250 |
| Standard_L12aos_v5 | 1 | 8,640 | 1,293,750 | 7,315 | 646,875 | 3,375 |
| Standard_L16aos_v5 | 1 | 11,520 | 1,725,000 | 9,750 | 862,500 | 4,500 |
| Standard_L24aos_v5 | 2 | 8,640 | 2,587,500 | 14,625 | 1,293,750 | 6,750 |
| Standard_L32aos_v5 | 2 | 11,520 | 3,450,000 | 19,500 | 1,725,000 | 9,000 |
| Standard_L48aos_v5 | 3 | 11,520 | 5,175,000 | 29,250 | 2,587,500 | 13,500 |
| Standard_L64aos_v5 | 3 | 15,360 | 6,900,000 | 39,000 | 3,450,000 | 18,000 |
| Standard_L96aos_v5 | 9 | 11,520 | 10,350,000 | 58,500 | 5,175,000 | 27,000 |
| Standard_L128aos_v5 | 6 | 15,360 | 13,800,000 | 78,000 | 6,900,000 | 36,000 |
| Standard_L160iaos_v5 | 9 | 15,360 | 20,700,000 | 117,000 | 10,350,000 | 54,000 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)


#### Table definitions
- Temp disk performance depends on many factors, including block size, workload patterns of read and write operations, queue depth (QD), and others. View temp disk performance specifications as best-case performance numbers, assuming 4 KB block sizes and QD=256 for IOPS, and 256 KB block sizes with QD=64 for throughput. Write performance is heavily impacted by how many blocks are in use on a device. Temp disk write performance specifications assume a device has a clean slate to enable the best performance. During steady-state operations, write performance is lower than the published specifications.
- For Laosv5 and Laosv4, temp disk refers to the NVMe local data disks used by the VM. While Lsv3 and Lasv3 have NVMe local data disks and a SCSI local temp disk, Laosv5 and Laosv4 only have NVMe local temp disks. There's no SCSI local temp disk on Laosv5.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps, where MBps = 10^6 bytes/sec.
- To learn how to get the best local storage performance for your VMs, see the [NVMe Temp Disk FAQ](/azure/virtual-machines/enable-nvme-temp-faqs).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage information for each size.

| Size Name | Max Remote Storage Disks | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst Ultra Disk and Premium SSD v2 IOPS | Uncached Burst Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L2aos_v5 | 4 | 4,400 | 150 | 44,000 | 1,413 | 5,000 | 170 | 50,000 | 1,600 |
| Standard_L4aos_v5 | 8 | 8,800 | 300 | 47,200 | 1,413 | 10,000 | 340 | 53,636 | 1,600 |
| Standard_L8aos_v5 | 16 | 17,600 | 600 | 47,200 | 1,413 | 20,000 | 680 | 53,636 | 1,600 |
| Standard_L12aos_v5 | 32 | 26,400 | 900 | 47,200 | 1,413 | 30,000 | 1,020 | 53,636 | 1,600 |
| Standard_L16aos_v5 | 32 | 35,200 | 1,200 | 72,700 | 1,413 | 40,000 | 1,360 | 82,613 | 1,600 |
| Standard_L24aos_v5 | 32 | 52,800 | 1,800 | 72,700 | 1,916 | 60,000 | 2,040 | 82,613 | 2,170 |
| Standard_L32aos_v5 | 32 | 70,400 | 2,400 | 94,400 | 2,875 | 80,000 | 2,720 | 107,272 | 3,258 |
| Standard_L48aos_v5 | 32 | 105,600 | 3,600 | 132,000 | 5,749 | 120,000 | 4,080 | 150,000 | 6,515 |
| Standard_L64aos_v5 | 32 | 140,800 | 4,800 | 192,500 | 5,749 | 160,000 | 5,440 | 218,750 | 6,515 |
| Standard_L96aos_v5 | 32 | 211,200 | 7,200 | 220,000 | 7,664 | 240,000 | 8,160 | 250,000 | 8,685 |
| Standard_L128aos_v5 | 64 | 260,000 | 9,600 | 260,000 | 10,588 | 320,000 | 10,880 | 320,000 | 12,000 |
| Standard_L160iaos_v5 | 64 | 260,000 | 12,000 | 260,000 | 12,000 | 400,000 | 12,000 | 400,000 | 12,000 |

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
| Standard_L2aos_v5 | 2 | 25,000 |
| Standard_L4aos_v5 | 2 | 25,000 |
| Standard_L8aos_v5 | 4 | 25,000 |
| Standard_L12aos_v5 | 6 | 25,000 |
| Standard_L16aos_v5 | 8 | 25,000 |
| Standard_L24aos_v5 | 8 | 37,500 |
| Standard_L32aos_v5 | 8 | 50,000 |
| Standard_L48aos_v5 | 8 | 75,000 |
| Standard_L64aos_v5 | 8 | 100,000 |
| Standard_L96aos_v5 | 8 | 150,000 |
| Standard_L128aos_v5 | 8 | 150,000 |
| Standard_L160iaos_v5 | 8 | 200,000 |


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
|[Premium Storage caching](../../premium-storage-performance.md)| Supported (except for L32aos_v5) |
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
