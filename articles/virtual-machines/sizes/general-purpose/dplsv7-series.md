---
title: Dplsv7 size series
description: Information on and specifications of the Dplsv7-series sizes
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/07/2026
ai-usage: ai-generated
ms.author: noahwood
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and features of the Dplsv7 series of virtual machines, so that I can select the appropriate size for my workload requirements."
---

# Dplsv7 sizes series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows Client VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

[!INCLUDE [dplsv7-series-summary](./includes/dplsv7-series-summary.md)]

**Agentic workloads:** Use these VMs for CPU-based agent sandboxes, tool execution, and orchestration services.

## Host specifications

[!INCLUDE [dplsv7-series-specs](./includes/dplsv7-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_D1pls_v7 | 1 | 2 |
| Standard_D2pls_v7 | 2 | 4 |
| Standard_D4pls_v7 | 4 | 8 |
| Standard_D8pls_v7 | 8 | 16 |
| Standard_D16pls_v7 | 16 | 32 |
| Standard_D32pls_v7 | 32 | 64 |
| Standard_D48pls_v7 | 48 | 96 |
| Standard_D64pls_v7 | 64 | 128 |
| Standard_D96pls_v7 | 96 | 192 |
| Standard_D128pls_v7 | 128 | 256 |

> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you'll see an error message. [NVMe](../../enable-nvme-interface.md) support is available on the most popular OS images, and we're continuously improving OS image compatibility.

#### VM Basics resources

- [Check vCPU quotas](../../quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series. For similar sizes with local storage, see the [Dpldsv7-series](./dpldsv7-series.md).
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst IOPS | Uncached Premium SSD Burst Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst Ultra Disk and Premium SSD v2 IOPS | Uncached Burst Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D1pls_v7 | 4 | 2,000 | 60 | 22,500 | 710 | 2,500 | 70 | 28,130 | 820 |
| Standard_D2pls_v7 | 8 | 4,000 | 120 | 45,000 | 1,410 | 5,000 | 140 | 56,250 | 1,640 |
| Standard_D4pls_v7 | 12 | 8,000 | 240 | 45,000 | 1,410 | 10,000 | 270 | 56,250 | 1,640 |
| Standard_D8pls_v7 | 24 | 16,000 | 470 | 45,000 | 1,410 | 20,000 | 550 | 56,250 | 1,640 |
| Standard_D16pls_v7 | 48 | 32,000 | 940 | 75,000 | 1,410 | 40,000 | 1,100 | 93,750 | 1,640 |
| Standard_D32pls_v7 | 64 | 64,000 | 1,890 | 100,000 | 1,920 | 80,000 | 2,190 | 125,000 | 2,230 |
| Standard_D48pls_v7 | 64 | 96,000 | 2,830 | 160,000 | 2,870 | 120,000 | 3,290 | 200,000 | 3,340 |
| Standard_D64pls_v7 | 64 | 128,000 | 3,780 | 160,000 | 3,830 | 160,000 | 4,380 | 200,000 | 4,450 |
| Standard_D96pls_v7 | 64 | 192,000 | 5,660 | 192,000 | 5,660 | 240,000 | 6,580 | 240,000 | 6,580 |
| Standard_D128pls_v7 | 64 | 204,800 | 7,550 | 204,800 | 7,550 | 320,000 | 8,770 | 320,000 | 8,770 |

#### Storage resources
- [Introduction to Azure managed disks](../../managed-disks-overview.md)
- [Azure managed disk types](../../disks-types.md)
- [Share an Azure managed disk](../../disks-shared.md)

#### Table definitions
- These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../disks-performance.md).

### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D1pls_v7 | 2 | 12,500 |
| Standard_D2pls_v7 | 3 | 16,000 |
| Standard_D4pls_v7 | 4 | 25,000 |
| Standard_D8pls_v7 | 4 | 25,000 |
| Standard_D16pls_v7 | 8 | 25,000 |
| Standard_D32pls_v7 | 8 | 25,000 |
| Standard_D48pls_v7 | 8 | 35,000 |
| Standard_D64pls_v7 | 15 | 45,000 |
| Standard_D96pls_v7 | 15 | 70,000 |
| Standard_D128pls_v7 | 15 | 85,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

> [!NOTE]
> Accelerated networking is required and turned on by default on all Dplsv7 machines.

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth).
- To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

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
| [Ephemeral OS Disks](../../ephemeral-os-disks.md) | Not Supported |
| [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) | Not Supported |
| [Azure Disk Encryption for Linux VMs](../../linux/disk-encryption-linux.md#restrictions) | Not Supported |
| [Azure Disk Encryption for Windows VMs](../../windows/disk-encryption-windows.md#restrictions) | Not Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
