---
title: Famdsv7 size series
description: Information on and specifications of the Famdsv7-series sizes
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 01/21/2026
ms.author: archat
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and features of the Famdsv7 series virtual machine sizes, so that I can select the appropriate resources for my applications based on performance and storage requirements."
---

# Famdsv7 sizes series

[!INCLUDE [famdsv7-summary](./includes/famdsv7-series-summary.md)]

## Host specifications
[!INCLUDE [famdsv7-series-specs](./includes/famdsv7-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_F1amds_v7 | 1 | 8 |
| Standard_F2amds_v7 | 2 | 16 |
| Standard_F4amds_v7 | 4 | 32 |
| Standard_F8amds_v7 | 8 | 64 |
| Standard_F16amds_v7 | 16 | 128 |
| Standard_F32amds_v7 | 32 | 256 |
| Standard_F48amds_v7 | 48 | 384 |
| Standard_F64amds_v7 | 64 | 512 |
| Standard_F80amds_v7 | 80 | 640 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read IOPS | Temp Disk Sequential Read Throughput (MBps) | Temp Disk Random Write IOPS | Temp Disk Sequential Write Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_F1amds_v7 | 1 | 110 | 37,500 | 280 | 15,000 | 140 |
| Standard_F2amds_v7 | 1 | 220 | 75,000 | 560 | 30,000 | 280 |
| Standard_F4amds_v7 | 1 | 440 | 150,000 | 1,120 | 60,000 | 560 |
| Standard_F8amds_v7 | 2 | 440 | 300,000 | 2,240 | 120,000 | 1,120 |
| Standard_F16amds_v7 | 4 | 440 | 600,000 | 4,480 | 240,000 | 2,240 |
| Standard_F32amds_v7 | 4 | 880 | 1,200,000 | 8,960 | 480,000 | 4,480 |
| Standard_F48amds_v7 | 6 | 880 | 1,800,000 | 13,440 | 720,000 | 6,720 |
| Standard_F64amds_v7 | 4 | 1,760 | 2,400,000 | 17,920 | 960,000 | 8,960 |
| Standard_F80amds_v7 | 4 | 2,200 | 3,000,000 | 22,400 | 1,200,000 | 11,200 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)

#### Table definitions
- Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Read performance specs assume 100% reads, and write performance specs assume 100% writes. Additionally, write performance is heavily impacted by how many blocks in use on a device. Temp disk write performance specs assume a device has a clean slate to enable the best performance. During steady state operations, write performance is expected to be lower than the published specs.
- NVMe temp disks are presented as raw NVMe devices that need to be initialized and formatted before use. For more details on how to format and initialize drives, refer to the [NVMe Temp Disk FAQ](/azure/virtual-machines/enable-nvme-temp-faqs).
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst Ultra Disk and Premium SSD v2 IOPS | Uncached Burst Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_F1amds_v7 | 10 | 4,000 | 118 | 44,000 | 1,412 | 4,400 | 136 | 48,400 | 1,653 |
| Standard_F2amds_v7 | 12 | 8,000 | 234 | 47,200 | 1,412 | 8,800 | 273 | 52,083 | 1,653 |
| Standard_F4amds_v7 | 26 | 16,000 | 468 | 47,200 | 1,412 | 17,600 | 547 | 52,083 | 1,653 |
| Standard_F8amds_v7 | 48 | 32,000 | 936 | 72,700 | 1,412 | 35,200 | 1,095 | 80,000 | 1,653 |
| Standard_F16amds_v7 | 64 | 64,000 | 1,872 | 94,400 | 1,916 | 70,400 | 2,191 | 104,167 | 2,241 |
| Standard_F32amds_v7 | 64 | 128,000 | 3,744 | 132,000 | 3,832 | 140,800 | 4,382 | 145,200 | 4,484 |
| Standard_F48amds_v7 | 64 | 192,000 | 5,663 | 192,500 | 5,749 | 211,200 | 6,573 | 211,750 | 6,669 |
| Standard_F64amds_v7 | 64 | 204,800 | 7,488 | 225,280 | 7,663 | 281,600 | 8,764 | 310,886 | 8,966 |
| Standard_F80amds_v7 | 64 | 212,000 | 10,344 | 242,640 | 11,410 | 310,000 | 10,356 | 355,443 | 11,450 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_F1amds_v7 | 2 | 16,000 |
| Standard_F2amds_v7 | 3 | 16,000 |
| Standard_F4amds_v7 | 4 | 25,000 |
| Standard_F8amds_v7 | 8 | 25,000 |
| Standard_F16amds_v7 | 8 | 25,000 |
| Standard_F32amds_v7 | 8 | 45,000 |
| Standard_F48amds_v7 | 8 | 70,000 |
| Standard_F64amds_v7 | 15 | 75,000 |
| Standard_F80amds_v7 | 15 | 80,000 |

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
|[Live Migration](../../maintenance-and-updates.md)| Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Supported |
|[Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Supported |

> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you’ll see an error message. [NVMe](../../../virtual-machines/enable-nvme-interface.md) support is available on the most popular OS images, and we're continuously improving OS image compatibility.

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
