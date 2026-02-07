---
title: Laosv4 size series
description: Information on and specifications of the Laosv4-series sizes
author: rrgomatam1
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/18/2025
ms.author: rishigomatam
ms.reviewer: mattmcinnes
---

# Laosv4 sizes series

[!INCLUDE [laosv4-summary](./includes/laosv4-series-summary.md)]

## Host specifications
[!INCLUDE [laosv4-series-specs](./includes/laosv4-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported (except for L32aos_v4) <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size.

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_L2aos_v4 | 2 | 16 |
| Standard_L4aos_v4 | 4 | 32 |
| Standard_L8aos_v4 | 8 | 64 |
| Standard_L12aos_v4 | 12 | 96 |
| Standard_L16aos_v4 | 16 | 128 |
| Standard_L24aos_v4 | 24 | 192 |
| Standard_L32aos_v4 | 32 | 256 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size.

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GB) | Temp Disk Random Read<sup>1</sup> IOPS | Temp Disk Sequential Read<sup>1</sup> Throughput (MB/s) | Temp Disk Random Write<sup>1</sup> IOPS | Temp Disk Sequential Write<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L2aos_v4 | 3 | 480 | 180,000 | 1,100 | 72,000 | 550 |
| Standard_L4aos_v4 | 3 | 960 | 360,000 | 2,200 | 144,000 | 1,100 |
| Standard_L8aos_v4 | 6 | 960 | 720,000 | 4,400 | 288,000 | 2,200 |
| Standard_L12aos_v4 | 9 | 960 | 1,080,000 | 6,600 | 432,000 | 3,300 |
| Standard_L16aos_v4 | 6 | 1,920 | 1,440,000 | 8,800 | 576,000 | 4,400 |
| Standard_L24aos_v4 | 9 | 1,920 | 2,160,000 | 13,200 | 864,000 | 6,600 |
| Standard_L32aos_v4 | 12 | 1,920 | 2,880,000 | 17,600 | 1,152,000 | 8,800 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)


#### Table definitions
- <sup>1</sup>Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Additionally, write performance is heavily impacted by how many blocks in use on a device. Temp disk write performance specs assume a device has a clean slate to enable the best performance. During steady state operations, write performance is expected to be lower than the published specs. 
- For Laosv4, temp disk refers to the NVMe local data disks used by the VM. While Lsv3/Lasv3 have NVMe local data disks and a SCSI local temp disk, Laosv4 only has NVMe local temp disks. There is no SCSI local temp disk on Laosv4.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best local storage performance for your VMs, see the [NVMe Temp Disk FAQ](/azure/virtual-machines/enable-nvme-temp-faqs).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size.

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L2aos_v4 | 4 | 4,400 | 150 | 20,000 | 1,250 | 4,400 | 150 | 20,000 | 1,250 |
| Standard_L4aos_v4 | 8 | 8,800 | 300 | 20,000 | 1,250 | 8,800 | 300 | 20,000 | 1,250 |
| Standard_L8aos_v4 | 16 | 17,600 | 600 | 20,000 | 1,250 | 17,600 | 600 | 20,000 | 1,250 |
| Standard_L12aos_v4 | 32 | 26,400 | 900 | 30,000 | 1,250 | 26,400 | 900 | 30,000 | 1,250 |
| Standard_L16aos_v4 | 32 | 35,200 | 1,200 | 40,000 | 1,250 | 35,200 | 1,200 | 40,000 | 1,250 |
| Standard_L24aos_v4 | 32 | 52,800 | 1,800 | 60,000 | 1,800 | 52,800 | 1,800 | 60,000 | 1,800 |
| Standard_L32aos_v4 | 32 | 70,400 | 2,400 | 80,000 | 2,400 | 70,400 | 2,400 | 80,000 | 2,400 |

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

Network interface info for each size.

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_L2aos_v4 | 2 | 7,500 |
| Standard_L4aos_v4 | 2 | 10,000 |
| Standard_L8aos_v4 | 4 | 12,500 |
| Standard_L12aos_v4 | 6 | 18,750 |
| Standard_L16aos_v4 | 8 | 25,000 |
| Standard_L24aos_v4 | 8 | 37,500 |
| Standard_L32aos_v4 | 8 | 50,000 |

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

