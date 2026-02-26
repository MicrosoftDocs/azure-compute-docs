---
title: Lsv4 size series
description: Information on and specifications of the Lsv4-series sizes
author: rrgomatam1
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/18/2025
ms.author: rishigomatam
ms.reviewer: mattmcinnes
---

# Lsv4 sizes series


[!INCLUDE [lsv4-summary](./includes/lsv4-series-summary.md)]

## Host specifications
[!INCLUDE [lsv4-series-specs](./includes/lsv4-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported (except for L96s_v4) <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size.

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_L2s_v4 | 2 | 16 |
| Standard_L4s_v4 | 4 | 32 |
| Standard_L8s_v4 | 8 | 64 |
| Standard_L16s_v4 | 16 | 128 |
| Standard_L32s_v4 | 32 | 256 |
| Standard_L48s_v4 | 48 | 384 |
| Standard_L64s_v4 | 64 | 512 |
| Standard_L80s_v4 | 80 | 640 |
| Standard_L96s_v4 | 96 | 768 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size.

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GB) | Temp Disk Random Read<sup>1</sup> IOPS | Temp Disk Sequential Read<sup>1</sup> Throughput (MB/s) | Temp Disk Random Write<sup>1</sup> IOPS | Temp Disk Sequential Write<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L2s_v4 | 1 | 480 | 137500 | 750 | 55000 | 375 |
| Standard_L4s_v4 | 2 | 480 | 275000 | 1500 | 110000 | 750 |
| Standard_L8s_v4 | 4 | 480 | 550000 | 3000 | 220000 | 1500 |
| Standard_L16s_v4 | 4 | 960 | 1100000 | 6000 | 440000 | 3000 |
| Standard_L32s_v4 | 8 | 960 | 2200000 | 12000 | 880000 | 6000 |
| Standard_L48s_v4 | 6 | 1920 | 3300000 | 18000 | 1320000 | 9000 |
| Standard_L64s_v4 | 8 | 1920 | 4400000 | 24000 | 1760000 | 12000 |
| Standard_L80s_v4 | 10 | 1920 | 5500000 | 30000 | 2200000 | 15000 |
| Standard_L96s_v4 | 12 | 1920 | 6600000 | 36000 | 2640000 | 18000 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)

#### Table definitions
- <sup>1</sup>Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Additionally, write performance is heavily impacted by how many blocks in use on a device. Temp disk write performance specs assume a device has a clean slate to enable the best performance. During steady state operations, write performance is expected to be lower than the published specs. 
- For Lsv4, temp disk refers to the NVMe local data disks used by the VM. While Lsv3/Lasv3 have NVMe local data disks and a SCSI local temp disk, Lsv4 only has NVMe local temp disks. There is no SCSI local temp disk on Lsv4.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best local storage performance for your VMs, see the [NVMe Temp Disk FAQ](/azure/virtual-machines/enable-nvme-temp-faqs).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size.

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L2s_v4 | 8 | 3,750 | 106 | 40,000 | 1,250 | 4,157 | 124 | 44,444 | 1,463 |
| Standard_L4s_v4 | 12 | 6,400 | 212 | 40,000 | 1,250 | 8,333 | 248 | 52,083 | 1,463 |
| Standard_L8s_v4 | 24 | 12,800 | 424 | 40,000 | 1,250 | 16,667 | 496 | 52,083 | 1,463 |
| Standard_L16s_v4 | 48 | 25,600 | 848 | 40,000 | 1,250 | 33,333 | 992 | 52,083 | 1,463 |
| Standard_L32s_v4 | 64 | 51,200 | 1,696 | 80,000 | 1,696 | 66,667 | 1,984 | 104,167 | 1,984 |
| Standard_L48s_v4 | 64 | 76,800 | 2,544 | 80,000 | 2,544 | 100,000 | 2,976 | 104,167 | 2,976 |
| Standard_L64s_v4 | 64 | 102,400 | 3,392 | 102,400 | 3,392 | 133,333 | 3,969 | 133,333 | 3,969 |
| Standard_L80s_v4 | 64 | 128,000 | 4,240 | 128,000 | 4,240 | 166,640 | 4,960 | 166,640 | 4,960 |
| Standard_L96s_v4 | 64 | 153,600 | 5,088 | 153,600 | 5,088 | 200,000 | 5,953 | 200,000 | 5,953 |

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

Network interface info for each size.

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_L2s_v4 | 2 |  12,500  |
| Standard_L4s_v4 | 2 |  12,500  |
| Standard_L8s_v4 | 4 |  12,500  |
| Standard_L16s_v4 | 8 |  12,500  |
| Standard_L32s_v4 | 8 |  16,000  |
| Standard_L48s_v4 | 8 |  24,000  |
| Standard_L64s_v4 | 8 |  30,000  |
| Standard_L80s_v4 | 8 |  36,000  |
| Standard_L96s_v4 | 8 |  41,000  |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size.

> [!NOTE]
> No accelerators are present in this series.

---
> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you’ll see an error message. [NVMe](/azure/virtual-machines/enable-nvme-interface) support is available on the most popular OS images, and we're continuously improving OS image compatibility.


[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
