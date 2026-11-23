---
title: Lpsv5 size series
description: Information on and specifications of the Lpsv5-series sizes
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/07/2026
ai-usage: ai-generated
ms.author: noahwood
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and features of the Lpsv5 series of virtual machines, so that I can select the appropriate size for my workload requirements."
---

# Lpsv5 sizes series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows Client VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

[!INCLUDE [lpsv5-series-summary](./includes/lpsv5-series-summary.md)]

## Host specifications

[!INCLUDE [lpsv5-series-specs](./includes/lpsv5-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_L1ps_v5 | 1 | 8 |
| Standard_L2ps_v5 | 2 | 16 |
| Standard_L4ps_v5 | 4 | 32 |
| Standard_L8ps_v5 | 8 | 64 |
| Standard_L16ps_v5 | 16 | 128 |
| Standard_L32ps_v5 | 32 | 256 |
| Standard_L48ps_v5 | 48 | 384 |
| Standard_L64ps_v5 | 64 | 512 |
| Standard_L96ps_v5 | 96 | 768 |
| Standard_L128ps_v5 | 128 | 1,024 |

> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you'll see an error message. [NVMe](../../enable-nvme-interface.md) support is available on the most popular OS images, and we're continuously improving OS image compatibility.

#### VM Basics resources

- [Check vCPU quotas](../../quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Temp Storage Disks (Qty.) | Temp Disk Size (GB) | Temp Disk Random Read IOPS | Temp Disk Sequential Read Throughput (MBps) | Temp Disk Random Write IOPS | Temp Disk Sequential Write Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_L1ps_v5 | 1 | 240 | 75,000 | 380 | 37,500 | 190 |
| Standard_L2ps_v5 | 1 | 480 | 150,000 | 750 | 75,000 | 380 |
| Standard_L4ps_v5 | 2 | 480 | 300,000 | 1,500 | 150,000 | 750 |
| Standard_L8ps_v5 | 4 | 480 | 600,000 | 3,000 | 300,000 | 1,500 |
| Standard_L16ps_v5 | 1 | 3,840 | 1,200,000 | 6,000 | 600,000 | 3,000 |
| Standard_L32ps_v5 | 2 | 3,840 | 2,400,000 | 12,000 | 1,200,000 | 6,000 |
| Standard_L48ps_v5 | 3 | 3,840 | 3,600,000 | 18,000 | 1,800,000 | 9,000 |
| Standard_L64ps_v5 | 4 | 3,840 | 4,800,000 | 24,000 | 2,400,000 | 12,000 |
| Standard_L96ps_v5 | 6 | 3,840 | 7,200,000 | 36,000 | 3,600,000 | 18,000 |
| Standard_L128ps_v5 | 6 | 3,840 | 7,200,000 | 36,000 | 3,600,000 | 18,000 |

#### Storage resources
- [NVMe Overview](../../nvme-overview.md)
- [FAQ for temp NVMe disks](../../enable-nvme-temp-faqs.yml)

#### Table definitions
- Total local temporary storage is calculated by multiplying the max number of storage disks with the temp disk size. For example, for the Standard_L128ps_v5, the total local temporary storage capacity is `6 x 3,840 GB = 23,040 GB`.
- Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Read performance specs assume 100% reads, and write performance specs assume 100% writes. Additionally, write performance is heavily impacted by how many blocks in use on a device. Temp disk write performance specs assume a device has a clean slate to enable the best performance. During steady state operations, write performance is expected to be lower than the published specs.
- NVMe temp disks are presented as raw NVMe devices that need to be initialized and formatted before use. For more details on how to format and initialize drives, refer to the [NVMe Temp Disk FAQ](../../enable-nvme-temp-faqs.yml).
- The IOPS and throughput values shown are the combined performance across all temp disks.
- Local storage capacity is shown in units of GB or 1,000^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB might appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly (R-O) or ReadWrite (R-W). For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst Ultra Disk and Premium SSD v2 IOPS | Uncached Burst Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_L1ps_v5 | 4 | 2,000 | 60 | 22,500 | 710 | 2,500 | 70 | 28,130 | 820 |
| Standard_L2ps_v5 | 8 | 4,000 | 120 | 45,000 | 1,410 | 5,000 | 140 | 56,250 | 1,640 |
| Standard_L4ps_v5 | 12 | 8,000 | 240 | 45,000 | 1,410 | 10,000 | 270 | 56,250 | 1,640 |
| Standard_L8ps_v5 | 24 | 16,000 | 470 | 45,000 | 1,410 | 20,000 | 550 | 56,250 | 1,640 |
| Standard_L16ps_v5 | 48 | 32,000 | 940 | 75,000 | 1,410 | 40,000 | 1,100 | 93,750 | 1,640 |
| Standard_L32ps_v5 | 64 | 64,000 | 1,890 | 100,000 | 1,920 | 80,000 | 2,190 | 125,000 | 2,230 |
| Standard_L48ps_v5 | 64 | 96,000 | 2,830 | 160,000 | 2,870 | 120,000 | 3,290 | 200,000 | 3,340 |
| Standard_L64ps_v5 | 64 | 128,000 | 3,780 | 160,000 | 3,830 | 160,000 | 4,380 | 200,000 | 4,450 |
| Standard_L96ps_v5 | 64 | 192,000 | 5,660 | 192,000 | 5,660 | 240,000 | 6,580 | 240,000 | 6,580 |
| Standard_L128ps_v5 | 64 | 204,800 | 7,550 | 204,800 | 7,550 | 320,000 | 8,770 | 320,000 | 8,770 |

#### Storage resources
- [Introduction to Azure managed disks](../../managed-disks-overview.md)
- [Azure managed disk types](../../disks-types.md)
- [Share an Azure managed disk](../../disks-shared.md)

#### Table definitions
- These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB might appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../disks-performance.md).

### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_L1ps_v5 | 2 | 12,500 |
| Standard_L2ps_v5 | 3 | 16,000 |
| Standard_L4ps_v5 | 4 | 25,000 |
| Standard_L8ps_v5 | 4 | 25,000 |
| Standard_L16ps_v5 | 8 | 25,000 |
| Standard_L32ps_v5 | 8 | 25,000 |
| Standard_L48ps_v5 | 8 | 35,000 |
| Standard_L64ps_v5 | 15 | 45,000 |
| Standard_L96ps_v5 | 15 | 70,000 |
| Standard_L128ps_v5 | 15 | 85,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

> [!NOTE]
> Accelerated networking is required and turned on by default on all Lpsv5 machines.

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth).
- To achieve the expected network performance on Linux or Windows, you might need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

## Feature support

| Feature name | Support status |
| --- | --- |
| [Premium Storage](../../premium-storage-performance.md) | Supported |
| [Premium Storage caching](../../premium-storage-performance.md) | Supported |
| [Live Migration](../../maintenance-and-updates.md#live-migration) | Not Supported |
| [Memory Preserving Updates](../../maintenance-and-updates.md) | Supported |
| [Generation 2 VMs](../../generation-2.md) | Supported |
| [Generation 1 VMs](../../generation-2.md) | Not Supported |
| [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli) | Required |
| [Ephemeral OS Disks](../../ephemeral-os-disks.md) | Supported |
| [Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml) | Supported |
| [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) | Not Supported |
| [Azure Disk Encryption for Linux VMs](../../linux/disk-encryption-linux.md#restrictions) | Not Supported |
| [Azure Disk Encryption for Windows VMs](../../windows/disk-encryption-windows.md#restrictions) | Not Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
