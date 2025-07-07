---
title: Endsv6 size series
description: Information on and specifications of the Endsv6-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 05/08/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to evaluate the specifications of the Endsv6 series VM sizes, so that I can select the appropriate size for my workloads based on resource requirements like CPU, memory, and storage performance."
---

# Endsv6 sizes series (Preview)

[!INCLUDE [endsv6-summary](./includes/endsv6-series-summary.md)]

## Host specifications
[!INCLUDE [endsv6-series-specs](./includes/endsv6-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_E2nds_v6    | 2    | 16    |
| Standard_E4nds_v6    | 4    | 32    |
| Standard_E8nds_v6    | 8    | 64    |
| Standard_E16nds_v6   | 16   | 128   |
| Standard_E32nds_v6   | 32   | 256   |
| Standard_E48nds_v6   | 48   | 384   |
| Standard_E64nds_v6   | 64   | 512   |
| Standard_E96nds_v6   | 96   | 768   |
| Standard_E128nds_v6  | 128  | 1024  |


#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

|  Size Name | Max Temp Storage Disks (Qty.)  | Temp Disk Size (GiB)  | Temp Disk Random Read (RR)1 IOPS  | Temp Disk Random Read (RR)1 Throughput (MB/s)  | Temp Disk Random Write (RW)1 IOPS  | Temp Disk Random Write (RW)1 Throughput (MB/s)  |
|---|---|---|---|---|---|---|
|    Standard_E2nds_v6    | 1  | 110 | 37500 | 180 | 15000 | 90 |
|    Standard_E4nds_v6    | 1  | 220 | 75000 | 360 | 30000 | 180 |
|    Standard_E8nds_v6    | 1  | 440 | 150000 | 720 | 60000 | 360 |
|    Standard_E16nds_v6   | 2  | 440 | 300000 | 1440 | 120000 | 720 |
|    Standard_E32nds_v6   | 4  | 440 | 600000 | 2880 | 240000 | 1440 |
|    Standard_E48nds_v6   | 6  | 440 | 900000 | 4320 | 360000 | 2160 |
|    Standard_E64nds_v6   | 4  | 880 | 1200000 | 5760 | 480000 | 2880 |
|    Standard_E96nds_v6   | 6  | 880 | 1800000 | 8640 | 720000 | 4320 |
|    Standard_E128nds_v6  | 4  | 1760 | 2400000 | 11520 | 960000 | 5760 |

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s)
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_E2nds_v6 | 8 | 3750 | 106 | 40000 | 1250 | 4167 | 124 | 44444 | 1463 |
| Standard_E4nds_v6 | 12 | 6400 | 212 | 40000 | 1250 | 8333 | 248 | 52083 | 1463 |
| Standard_E8nds_v6 | 24 | 12800 | 424 | 40000 | 1250 | 16667 | 496 | 52083 | 1463 |
| Standard_E16nds_v6 | 48 | 25600 | 848 | 40000 | 1250 | 33333 | 992 | 52083 | 1463 |
| Standard_E32nds_v6 | 64 | 51200 | 1696 | 80000 | 1696 | 66667 | 1984 | 104167 | 1984 |
| Standard_E48nds_v6 | 64 | 76800 | 2544 | 80000 | 2544 | 1000000 | 2976 | 104167 | 2976 |
| Standard_E64nds_v6 | 64 | 76800 | 2544 | 102400 | 3392 | 133333 | 3969 | 133333 | 3969 |
| Standard_E96nds_v6 | 64 | 153600 | 5088 | 153600 | 5088 | 200000 | 5953 | 200000 | 5953 |
| Standard_E128nds_v6 | 64 | 204800 | 6782 | 204800 | 6782 | 266667 | 7935 | 266667 | 7935 |

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

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_E2nds_v6 | 4 | 25000 |
| Standard_E4nds_v6 | 4 | 30000 |
| Standard_E8nds_v6 | 8 | 40000 |
| Standard_E16nds_v6 | 8 | 50000 |
| Standard_E32nds_v6 | 8 | 50000 |
| Standard_E48nds_v6 | 8 | 75000 |
| Standard_E64nds_v6 | 8 | 100000 |
| Standard_E96nds_v6 | 15 | 150000 |
| Standard_E128nds_v6 | 15 | 200000 |

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

