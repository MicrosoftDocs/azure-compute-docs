---
title: NVads V710 v5 size series
description: Information on and specifications of the NVads-V710 v5-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 09/16/2024
ms.author: wwilliams
ms.reviewer: mattmcinnes
---

# NVads V710 v5-series

[!INCLUDE [nvadsv710-v5-summary](./includes/nvadsv710-v5-summary.md)]

## Host specifications
[!INCLUDE [nvadsv710-v5-series-specs](./includes/nvadsv710-v5-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_NV4ads_V710_v5 | 4 | 16 |
| Standard_NV8ads_V710_v5 | 8 | 32 |
| Standard_NV12ads_V710_v5 | 12 | 64 |
| Standard_NV24ads_V710_v5 | 24 | 128 |
| Standard_NV28adms_V710_v5 | 28 | 160 |

#### VM Basics resources
- [Check vCPU quotas](../../quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage information for each size.

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) |
| --- | --- | --- |
| Standard_NV4ads_V710_v5 | 2 | 256 |
| Standard_NV8ads_V710_v5 | 4 | 512 |
| Standard_NV12ads_V710_v5 | 6 | 768 |
| Standard_NV24ads_V710_v5 | 12 | 1536 |
| Standard_NV28adms_V710_v5 | 14 | 1536 |

#### Storage resources
- [Introduction to Azure managed disks](../../managed-disks-overview.md)
- [Azure managed disk types](../../disks-types.md)
- [Share an Azure managed disk](../../disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage information for each size.

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Disk IOPS | Uncached Disk Speed (MBps) |
| --- | --- | --- | --- |
| Standard_NV4ads_V710_v5 | 2 | 7600 | 180 |
| Standard_NV8ads_V710_v5 | 4 | 1520 | 360 |
| Standard_NV12ads_V710_v5 | 6 | 2280 | 540 |
| Standard_NV24ads_V710_v5 | 12 | 4560 | 1080 |
| Standard_NV28adms_V710_v5 | 14 | 5320 | 1260 |

#### Storage resources
- [Introduction to Azure managed disks](../../managed-disks-overview.md)
- [Azure managed disk types](../../disks-types.md)
- [Share an Azure managed disk](../../disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.
- <sup>2</sup>Special Storage refers to either [Ultra Disk](../../disks-enable-ultra-ssd.md) or [Premium SSD v2](../../disks-deploy-premium-v2.md) storage.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface information for each size.

| Size Name | Max NICs (Qty.) | Max Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_NV4ads_V710_v5 | 2 | 3300 |
| Standard_NV8ads_V710_v5 | 4 | 6600 |
| Standard_NV12ads_V710_v5 | 4 | 10000 |
| Standard_NV24ads_V710_v5 | 8 | 20000 |
| Standard_NV28adms_V710_v5 | 8 | 20000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](../../network-overview.md)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) information for each size.

| Size Name | Accelerators (Qty.) | Accelerator-Memory (GB) |
| --- | --- | --- |
| Standard_NV4ads_V710_v5 | 1/6 | 4 |
| Standard_NV8ads_V710_v5 | 1/3 | 8 |
| Standard_NV12ads_V710_v5 | 1/2 | 12 |
| Standard_NV24ads_V710_v5 | 1 | 24 |
| Standard_NV28adms_V710_v5 | 1 | 24 |

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
