---
title: FXmsv2 size series
description: Information on and specifications of the FXmsv2-series sizes
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 05/23/2025
ms.author: archat
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to understand the specifications and features of the FXmsv2 size series, so that I can select the appropriate virtual machine sizes for my organization's diverse workloads.
---

# FXmsv2 sizes series

[!INCLUDE [fxmsv2-summary](./includes/fxmsv2-series-summary.md)]

## Host specifications
[!INCLUDE [fxmsv2-series-specs](./includes/fxmsv2-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br> [NVMe storage](../../nvme-overview.md): Supported <br> [Constrained core](../../constrained-vcpu.md) : Supported
## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_FX2ms_v2 | 2 | 42 |
| Standard_FX4ms_v2 | 4 | 84 |
| Standard_FX8ms_v2 | 8 | 168 |
| Standard_FX12ms_v2 | 12 | 252 |
| Standard_FX16ms_v2 | 16 | 336 |
| Standard_FX24ms_v2 | 24 | 504 |
| Standard_FX32ms_v2 | 32 | 672 |
| Standard_FX48ms_v2 | 48 | 1008 |
| Standard_FX64ms_v2 | 64 | 1344 |
| Standard_FX96ms_v2 | 96 | 1832 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).



### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_FX2ms_v2 | 8 | 8,000 | 273 | 40,000 | 1,250 | 12,000 | 300 | 60,000 | 1,375 |
| Standard_FX4ms_v2 | 12 | 16,000 | 545 | 65,000 | 1,800 | 21,400 | 600 | 86,938 | 1,980 |
| Standard_FX8ms_v2 | 24 | 33,000 | 1,091 | 65,000 | 1,800 | 44,200 | 1,200 | 87,061 | 1,980 |
| Standard_FX12ms_v2 | 48 | 49,500 | 1,636 | 67,500 | 2,400 | 66,300 | 1,750 | 90,409 | 2,567 |
| Standard_FX16ms_v2 | 48 | 66,000 | 2,182 | 70,000 | 3,000 | 88,400 | 2,300 | 93,758 | 3,163 |
| Standard_FX24ms_v2 | 48 | 98,000 | 3,273 | 105,000 | 4,500 | 131,300 | 3,550 | 140,679 | 4,881 |
| Standard_FX32ms_v2 | 64 | 130,000 | 4,364 | 140,000 | 6,000 | 174,200 | 4,800 | 187,600 | 6,600 |
| Standard_FX48ms_v2 | 64 | 190,000 | 6,545 | 200,000 | 9,000 | 253,300 | 7,300 | 266,632 | 10,038 |
| Standard_FX64ms_v2 | 64 | 220,000 | 8,727 | 230,000 | 9,750 | 294,800 | 9,600 | 308,200 | 10,725 |
| Standard_FX96ms_v2 | 64 | 260,000 | 10,000 | 260,000 | 10,625 | 400,000 | 11,250 | 400,000 | 12,000 |

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
| Standard_FX2ms_v2 | 2 | 12,500 |
| Standard_FX4ms_v2 | 2 | 25,000 |
| Standard_FX8ms_v2 | 4 | 25,000 |
| Standard_FX12ms_v2 | 4 | 25,000 |
| Standard_FX16ms_v2 | 8 | 25,000 |
| Standard_FX24ms_v2 | 8 | 50,000 |
| Standard_FX32ms_v2 | 8 | 50,000 |
| Standard_FX48ms_v2 | 8 | 50,000 |
| Standard_FX64ms_v2 | 8 | 50,000 |
| Standard_FX96ms_v2 | 8 | 70,000 |

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

