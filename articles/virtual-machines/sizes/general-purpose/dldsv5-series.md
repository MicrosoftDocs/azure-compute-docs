---
title: Dldsv5 size series
description: Information on and specifications of the Dldsv5-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/18/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud infrastructure planner, I want to understand the specifications and features of the Dldsv5 virtual machine sizes, so that I can select the appropriate size for my workload requirements and optimize resource allocation.
---

# Dldsv5 sizes series

[!INCLUDE [dldsv5-summary](./includes/dldsv5-series-summary.md)]

## Host specifications
[!INCLUDE [dldsv5-series-specs](./includes/dldsv5-series-specs.md)]

## Feature support
Premium Storage: Supported<br>
Premium Storage caching: Supported<br>
Live Migration: Supported<br>
Memory Preserving Updates: Supported<br>
VM Generation Support: Generation 1 and 2<br>
Accelerated Networking: Required <br>
Ephemeral OS Disks: Supported <br>
Nested Virtualization: Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2lds_v5 | 2 | 4 |
| Standard_D4lds_v5 | 4 | 8 |
| Standard_D8lds_v5 | 8 | 16 |
| Standard_D16lds_v5 | 16 | 32 |
| Standard_D32lds_v5 | 32 | 64 |
| Standard_D48lds_v5 | 48 | 96 |
| Standard_D64lds_v5 | 64 | 128 |
| Standard_D96lds_v5 | 96 | 192 |

#### VM Basics resources
- [What are vCPUs (Qty.)](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage (Qty.) | Temp Storage Size (GiB) | Temp ReadWrite Storage IOPS | Temp ReadWrite Storage Speed (MBps) |
| --- | --- | --- | --- | --- |
| Standard_D2lds_v5 | 1 | 75 | 9000 | 125 |
| Standard_D4lds_v5 | 1 | 150 | 19000 | 250 |
| Standard_D8lds_v5 | 1 | 300 | 38000 | 500 |
| Standard_D16lds_v5 | 1 | 600 | 75000 | 1000 |
| Standard_D32lds_v5 | 1 | 1200 | 150000 | 2000 |
| Standard_D48lds_v5 | 1 | 1800 | 225000 | 3000 |
| Standard_D64lds_v5 | 1 | 2400 | 300000 | 4000 |
| Standard_D96lds_v5 | 1 | 3600 | 450000 | 4000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly (R-O) or ReadWrite (R-W). For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage (Qty.) | Uncached Storage IOPS | Uncached Storage Speed (MBps) | Uncached Storage Burst<sup>1</sup> IOPS | Uncached Storage Burst<sup>1</sup> Speed (MBps) |
| --- | --- | --- | --- | --- | --- |
| Standard_D2lds_v5 | 4 | 3750 | 85 | 10000 | 1200 |
| Standard_D4lds_v5 | 8 | 6400 | 145 | 20000 | 1200 |
| Standard_D8lds_v5 | 16 | 12800 | 290 | 20000 | 1200 |
| Standard_D16lds_v5 | 32 | 25600 | 600 | 40000 | 1200 |
| Standard_D32lds_v5 | 32 | 51200 | 865 | 80000 | 2000 |
| Standard_D48lds_v5 | 32 | 76800 | 1315 | 80000 | 3000 |
| Standard_D64lds_v5 | 32 | 80000 | 1735 | 80000 | 3000 |
| Standard_D96lds_v5 | 32 | 80000 | 2600 | 80000 | 4000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D2lds_v5 | 2 | 12500 |
| Standard_D4lds_v5 | 2 | 12500 |
| Standard_D8lds_v5 | 4 | 12500 |
| Standard_D16lds_v5 | 8 | 12500 |
| Standard_D32lds_v5 | 8 | 16000 |
| Standard_D48lds_v5 | 8 | 24000 |
| Standard_D64lds_v5 | 8 | 30000 |
| Standard_D96lds_v5 | 8 | 35000 |

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


