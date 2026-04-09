---
title: Dpsv6 size series
description: Information on and specifications of the Dpsv6-series sizes
author: noahwood28
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.custom:
  - build-2024
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: noahwood
ms.reviewer: mattmcinnes, tomvcassidy
# Customer intent: As a cloud architect, I want to understand the specifications and capabilities of the Dpsv6 series virtual machine sizes, so that I can select the appropriate VM size for my application's performance and resource requirements.
---

# Dpsv6 sizes series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows Client VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

[!INCLUDE [dpsv6-summary](./includes/dpsv6-series-summary.md)]

## Host specifications
[!INCLUDE [dpsv6-series-specs](./includes/dpsv6-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_D2ps_v6 | 2 | 8 |
| Standard_D4ps_v6 | 4 | 16 |
| Standard_D8ps_v6 | 8 | 32 |
| Standard_D16ps_v6 | 16 | 64 |
| Standard_D32ps_v6 | 32 | 128 |
| Standard_D48ps_v6 | 48 | 192 |
| Standard_D64ps_v6 | 64 | 256 |
| Standard_D96ps_v6 | 96 | 384 |

#### VM Basics resources
- [What are vCPUs (Qty.)](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series. For similar sizes with local storage, see the [Dpdsv6-series](./dpdsv6-series.md).
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ps_v6 | 8 | 3,750 | 106 | 10,000 | 1,250 | 4,163 | 124 | 11,100 | 1,463 |
| Standard_D4ps_v6 | 12 | 6,400 | 212 | 20,000 | 1,250 | 8,333 | 248 | 26,040 | 1,463 |
| Standard_D8ps_v6 | 24 | 12,800 | 424 | 20,000 | 1,250 | 16,666 | 496 | 26,040 | 1,463 |
| Standard_D16ps_v6 | 48 | 25,600 | 848 | 40,000 | 1,250 | 33,331 | 992 | 52,080 | 1,463 |
| Standard_D32ps_v6 | 64 | 51,200 | 1,696 | 80,000 | 2000 | 66,662 | 1984 | 104,160 | 2,340 |
| Standard_D48ps_v6 | 64 | 76,800 | 2,544 | 80,000 | 3,000 | 99,994 | 2,976 | 104,160 | 3,510 |
| Standard_D64ps_v6 | 64 | 102,400 | 3,392 | 102,400 | 3,392 | 133,325 | 3,969 | 133,325 | 3,969 |
| Standard_D96ps_v6 | 64 | 153,600 | 5,000 | 153,600 | 5,000 | 199,987 | 5,850 | 199,987 | 5,850 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D2ps_v6 | 2 | 12,500 |
| Standard_D4ps_v6 | 2 | 12,500 |
| Standard_D8ps_v6 | 4 | 15,000 |
| Standard_D16ps_v6 | 8 | 15,000 |
| Standard_D32ps_v6 | 8 | 20,000 |
| Standard_D48ps_v6 | 8 | 30,000 |
| Standard_D64ps_v6 | 8 | 40,000 |
| Standard_D96ps_v6 | 8 | 60,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

> [!NOTE]
> Accelerated networking is required and turned on by default on all Dpsv6 machines.

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---



## Feature Support
|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Supported |
|[Live Migration](../../maintenance-and-updates.md#live-migration)| Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Supported |
|[VM Generation Support](../../generation-2.md)| Generation 2 |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disks](../../ephemeral-os-disks.md)| Not supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Not supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]



