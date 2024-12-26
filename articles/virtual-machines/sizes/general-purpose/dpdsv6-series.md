---
title: Dpdsv6 size series
description: Information on and specifications of the Dpdsv6-series sizes
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.custom:
  - build-2024
ms.topic: conceptual
ms.date: 08/27/2024
ms.author: archat
ms.reviewer: mattmcinnes, tomcassidy
---

# Dpdsv6 sizes series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows Client VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

[!INCLUDE [dpdsv6-summary](./includes/dpdsv6-series-summary.md)]

## Host specifications
[!INCLUDE [dpdsv6-series-specs](./includes/dpdsv6-series-specs.md)]

## Feature Support
- [Premium Storage](../../premium-storage-performance.md): Supported 
- [Premium Storage caching](../../premium-storage-performance.md): Supported 
- [Memory Preserving Updates](../../maintenance-and-updates.md): Supported 
- [VM Generation Support](../../generation-2.md): Generation 2 
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported 
- [Ephemeral OS Disks](../../ephemeral-os-disks.md): Not Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not supported

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_D2pds_v6 | 2 | 8 |
| Standard_D4pds_v6 | 4 | 16 |
| Standard_D8pds_v6 | 8 | 32 |
| Standard_D16pds_v6 | 16 | 64 |
| Standard_D32pds_v6 | 32 | 128 |
| Standard_D48pds_v6 | 48 | 192 |
| Standard_D64pds_v6 | 64 | 256 |
| Standard_D96pds_v6 | 96 | 384 |

> [!NOTE]
> The Dpdsv6 VM series will only work on OS images that support NVMe (i.e.  NVMe drivers required for the local storage). If your current OS image doesn't have NVMe support, you’ll see an error message. NVMe support is available on the most popular OS images, and we're continuously improving OS image compatibility.

#### VM Basics resources
- [What are vCPUs (Qty.)](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage (Qty.) | Temp Storage Size (GiB) | Temp ReadWrite Disk IOPS | Temp ReadWrite Disk Speed (MBps) | Temp ReadOnly Disk IOPS | Temp ReadOnly Disk Speed (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2pds_v6 | 1 | 110 | 15000 | 90 | 37500 | 180 |
| Standard_D4pds_v6 | 1 | 220 | 30000 | 180 | 75000 | 360 |
| Standard_D8pds_v6 | 1 | 440 | 60000 | 360 | 150000 | 720 |
| Standard_D16pds_v6 | 2 | 440 | 120000 | 720 | 300000 | 1440 |
| Standard_D32pds_v6 | 4 | 440 | 240000 | 1440 | 600000 | 2880 |
| Standard_D48pds_v6 | 6 | 440 | 360000 | 2160 | 900000 | 4320 |
| Standard_D64pds_v6 | 4 | 880 | 480000 | 2880 | 1200000 | 5760 |
| Standard_D96pds_v6 | 6 | 880 | 720000 | 4320 | 1800000 | 8640 |

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

| Size Name | Max Remote Storage (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2pds_v6 | 8 | 3750 | 106 | 10000 | 1250 | 4163 | 124 | 11100 | 1463 |
| Standard_D4pds_v6 | 12 | 6400 | 212 | 20000 | 1250 | 8333 | 248 | 26040 | 1463 |
| Standard_D8pds_v6 | 24 | 12800 | 424 | 20000 | 1250 | 16666 | 496 | 26040 | 1463 |
| Standard_D16pds_v6 | 48 | 25600 | 848 | 40000 | 1250 | 33331 | 992 | 52080 | 1463 |
| Standard_D32pds_v6 | 64 | 51200 | 1696 | 80000 | 2000 | 66662 | 1984 | 104160 | 2340 |
| Standard_D48pds_v6 | 64 | 76800 | 2544 | 80000 | 3000 | 99994 | 2976 | 104160 | 3510 |
| Standard_D64pds_v6 | 64 | 102400 | 3392 | 102400 | 3392 | 133325 | 3969 | 133325 | 4680 |
| Standard_D96pds_v6 | 64 | 153600 | 5000 | 153600 | 5000 | 199987 | 5850 | 199987 | 5953 |

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
| Standard_D2pds_v6 | 2 | 12500 |
| Standard_D4pds_v6 | 2 | 12500 |
| Standard_D8pds_v6 | 4 | 15000 |
| Standard_D16pds_v6 | 8 | 15000 |
| Standard_D32pds_v6 | 8 | 20000 |
| Standard_D48pds_v6 | 8 | 30000 |
| Standard_D64pds_v6 | 8 | 40000 |
| Standard_D96pds_v6 | 8 | 60000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

> [!NOTE]
> Accelerated networking is required and turned on by default on all Dpdsv6 machines.

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



