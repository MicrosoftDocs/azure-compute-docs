---
title: DCedsv5 size series
description: Information on and specifications of the DCedsv5-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to assess the specifications and capabilities of the DCedsv5 series virtual machine sizes, so that I can determine the appropriate size for my workloads based on CPU, memory, and storage requirements."
---

# DCedsv5 sizes series

[!INCLUDE [dcedsv5-summary](./includes/dcedsv5-series-summary.md)]

## Host specifications
[!INCLUDE [dcedsv5-series-specs](./includes/dcedsv5-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Not Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_DC2eds_v5 | 2 | 8 |
| Standard_DC4eds_v5 | 4 | 16 |
| Standard_DC8eds_v5 | 8 | 32 |
| Standard_DC16eds_v5 | 16 | 64 |
| Standard_DC32eds_v5 | 32 | 128 |
| Standard_DC48eds_v5 | 48 | 192 |
| Standard_DC64eds_v5 | 64 | 256 |
| Standard_DC96eds_v5 | 96 | 384 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- |
| Standard_DC2eds_v5 | 1 | 47 | 9300 | 100 |
| Standard_DC4eds_v5 | 1 | 105 | 19500 | 200 |
| Standard_DC8eds_v5 | 1 | 227 | 38900 | 500 |
| Standard_DC16eds_v5 | 1 | 463 | 76700 | 1000 |
| Standard_DC32eds_v5 | 1 | 935 | 153200 | 2000 |
| Standard_DC48eds_v5 | 1 | 1407 | 229700 | 3000 |
| Standard_DC64eds_v5 | 1 | 2823 | 306200 | 4000 |
| Standard_DC96eds_v5 | 1 | 2823 | 459200 | 4000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_DC2eds_v5 | 4 | 3750 | 80 | 10000 | 1200 |
| Standard_DC4eds_v5 | 8 | 6400 | 140 | 20000 | 1200 |
| Standard_DC8eds_v5 | 16 | 12800 | 300 | 20000 | 1200 |
| Standard_DC16eds_v5 | 32 | 25600 | 600 | 40000 | 1200 |
| Standard_DC32eds_v5 | 32 | 51200 | 860 | 80000 | 2000 |
| Standard_DC48eds_v5 | 32 | 76800 | 1320 | 80000 | 3000 |
| Standard_DC64eds_v5 | 32 | 80000 | 1740 | 80000 | 3000 |
| Standard_DC96eds_v5 | 32 | 80000 | 2600 | 120000 | 4000 |

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
| Standard_DC2eds_v5 | 2 | 3000 |
| Standard_DC4eds_v5 | 2 | 5000 |
| Standard_DC8eds_v5 | 4 | 5000 |
| Standard_DC16eds_v5 | 8 | 10000 |
| Standard_DC32eds_v5 | 8 | 12500 |
| Standard_DC48eds_v5 | 8 | 15000 |
| Standard_DC64eds_v5 | 8 | 20000 |
| Standard_DC96eds_v5 | 8 | 30000 |

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


