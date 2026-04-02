---
title: Dalsv6 size series
description: Information on and specifications of the Dalsv6-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to compare the specifications of the Dalsv6 series virtual machine sizes, so that I can select the right size for my application's performance and resource requirements.
---

# Dalsv6 sizes series
[!INCLUDE [dalsv6-summary](./includes/dalsv6-series-summary.md)]

## Host specifications
[!INCLUDE [dalsv6-series-specs](./includes/dalsv6-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2als_v6 | 2 | 4 |
| Standard_D4als_v6 | 4 | 8 |
| Standard_D8als_v6 | 8 | 16 |
| Standard_D16als_v6 | 16 | 32 |
| Standard_D32als_v6 | 32 | 64 |
| Standard_D48als_v6 | 48 | 96 |
| Standard_D64als_v6 | 64 | 128 |
| Standard_D96als_v6 | 96 | 192 |

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

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2als_v6 | 4 | 4,000 | 90 | 20,000 | 1,250 | 4,000 | 90 | 20,000 | 1,250 |
| Standard_D4als_v6 | 8 | 7,600 | 180 | 20,000 | 1,250 | 7,600 | 180 | 20,000 | 1,250 |
| Standard_D8als_v6 | 16 | 15,200 | 360 | 20,000 | 1,250 | 15,200 | 360 | 20,000 | 1,250 |
| Standard_D16als_v6 | 32 | 30,400 | 720 | 40,000 | 1,250 | 30,400 | 720 | 40,000 | 1,250 |
| Standard_D32als_v6 | 32 | 57,600 | 1,440 | 80,000 | 1,700 | 57,600 | 1,440 | 80,000 | 1,700 |
| Standard_D48als_v6 | 32 | 86,400 | 2,160 | 90,000 | 2,550 | 86,400 | 2,160 | 90,000 | 2,550 |
| Standard_D64als_v6 | 32 | 115,200 | 2,880 | 120,000 | 3,400 | 115,200 | 2,880 | 120,000 | 3,400 |
| Standard_D96als_v6 | 32 | 175,000 | 4,320 | 175,000 | 5,090 | 175,000 | 4,320 | 175,000 | 5,090 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D2als_v6 | 2 | 12,500 |
| Standard_D4als_v6 | 2 | 12,500 |
| Standard_D8als_v6 | 4 | 12,500 |
| Standard_D16als_v6 | 8 | 16,000 |
| Standard_D32als_v6 | 8 | 20,000 |
| Standard_D48als_v6 | 8 | 28,000 |
| Standard_D64als_v6 | 8 | 36,000 |
| Standard_D96als_v6 | 8 | 40,000 |

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
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Not Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Supported |


> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you'll see an error message. [NVMe](../../../virtual-machines/enable-nvme-interface.md) support is available on the most popular OS images, and we're continuously improving OS image compatibility.

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]


