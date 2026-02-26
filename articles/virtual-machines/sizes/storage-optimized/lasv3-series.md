---
title: Lasv3 size series
description: Information on and specifications of the Lasv3-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and capabilities of the Lasv3 size series for VMs, so that I can select the appropriate size that meets the performance needs of my applications."
---

# Lasv3 sizes series

[!INCLUDE [lasv3-summary](./includes/lasv3-series-summary.md)]

## Host specifications
[!INCLUDE [lasv3-series-specs](./includes/lasv3-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Not Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_L8as_v3 | 8 | 64 |
| Standard_L16as_v3 | 16 | 128 |
| Standard_L32as_v3 | 32 | 256 |
| Standard_L48as_v3 | 48 | 384 |
| Standard_L64as_v3 | 64 | 512 |
| Standard_L80as_v3 | 80 | 640 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Max NVMe Disks (Qty.) | NVMe Disk Size (TB) | NVMe Disk IOPS | NVMe Disk Throughput (MB/s) | 
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L8as_v3  | 1 | 80  | 1  | 1.92 | 400000 | 2000 |
| Standard_L16as_v3 | 1 | 160 | 2  | 1.92 | 800000 | 4000 |
| Standard_L32as_v3 | 1 | 320 | 4  | 1.92 | 1.5M   | 8000 |
| Standard_L48as_v3 | 1 | 480 | 6  | 1.92 | 2.2M   | 14000 |
| Standard_L64as_v3 | 1 | 640 | 8  | 1.92 | 2.9M   | 16000 |
| Standard_L80as_v3 | 1 | 800 | 10 | 1.92 | 3.8M   | 20000 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).
- **Temp disk**: Lasv3-series VMs have a standard SCSI-based temp resource disk for use by the OS paging or swap file (`D:` on Windows, `/dev/sdb` on Linux). This disk provides 80 GiB of storage, 4000 IOPS, and 80 MBps transfer rate for every 8 vCPUs. For example, Standard_L80as_v3 provides 800 GiB at 40000 IOPS and 800 MBPS. This configuration ensures that the NVMe drives can be fully dedicated to application use. This disk is ephemeral, and all data is lost on stop or deallocation. 
- **NVMe Disks**: NVMe disk throughput can go higher than the specified numbers. However, higher performance isn't guaranteed. Local NVMe disks are ephemeral. Data is lost on these disks if you stop or deallocate your VM. 
- **NVMe Disk encryption** Lasv3 VMs created or allocated on or after 1/1/2023 have their local NVMe drives encrypted by default using hardware-based encryption with a Platform-managed key, except for the regions listed below. 

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_L8as_v3 | 16 | 12800 | 200 | 20000 | 1280 |
| Standard_L16as_v3 | 32 | 25600 | 384 | 40000 | 1280 |
| Standard_L32as_v3 | 32 | 51200 | 768 | 80000 | 1600 |
| Standard_L48as_v3 | 32 | 76800 | 1152 | 80000 | 2000 |
| Standard_L64as_v3 | 32 | 80000 | 1280 | 80000 | 2000 |
| Standard_L80as_v3 | 32 | 80000 | 1400 | 80000 | 2000 |

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
| Standard_L8as_v3 | 4 | 12500 |
| Standard_L16as_v3 | 8 | 12500 |
| Standard_L32as_v3 | 8 | 16000 |
| Standard_L48as_v3 | 8 | 24000 |
| Standard_L64as_v3 | 8 | 32000 |
| Standard_L80as_v3 | 8 | 32000 |

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

