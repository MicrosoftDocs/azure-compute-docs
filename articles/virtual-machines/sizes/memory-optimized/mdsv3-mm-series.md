---
title: Mdsv3 Medium Memory size series
description: Information on and specifications of the Mdsv3-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/03/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
---

# Mdsv3 Medium Memory sizes series

[!INCLUDE [mdsv3-mm-summary](./includes/mdsv3-mm-series-summary.md)]

## Host specifications
[!INCLUDE [mdsv3-mm-series-specs](./includes/mdsv3-mm-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-virtual-machine-accelerated-networking): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>[Hibernation](../../hibernate-resume.md): Not Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_M12ds_v3 | 12 | 240 |
| Standard_M24ds_v3 | 24 | 480 |
| Standard_M48ds_1_v3 | 48 | 974 |
| Standard_M96ds_1_v3 | 96 | 974 |
| Standard_M96ds_2_v3 | 96 | 1,946 |
| Standard_M176ds_3_v3 | 176 | 2,794 |
| Standard_M176ds_4_v3 | 176 | 3,892 |

#### VM Basics resources
- To run SQL Server on a VM with more than 64 vCores per NUMA node, [disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#disable-smt-in-an-azure-virtual-machine).
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Max Temp Disk Sequential Read (SR)<sup>1</sup> IOPS | Max Temp Disk Sequential Read (SR)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- |
| Standard_M12ds_v3 | 1 | 400 | 10,000 | 100 |
| Standard_M24ds_v3 | 1 | 400 | 20,000 | 200 |
| Standard_M48ds_1_v3 | 1 | 400 | 40,000 | 400 |
| Standard_M96ds_1_v3 | 1 | 400 | 40,000 | 400 |
| Standard_M96ds_2_v3 | 1 | 400 | 160,000 | 1,600 |
| Standard_M176ds_3_v3 | 1 | 400 | 160,000 | 1,600 |
| Standard_M176ds_4_v3 | 1 | 400 | 160,000 | 1,600 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between SR (Sequential Read) and RW (Sequential Write) operations. SR operations are typically faster than SW operations. The SW speed is usually slower than the SR speed on series where only the SR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Max Uncached Premium SSD Disk IOPS | Max Uncached Premium SSD Throughput (MB/s) | Max Uncached Ultra Disk and Premium SSD v2 IOPS | Max Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_M12ds_v3 | 64 | 16,250 | 390 | 16,250 | 390 |
| Standard_M24ds_v3 | 64 | 32,500 | 780 | 32,500 | 780 |
| Standard_M48ds_1_v3 | 64 | 65,000 | 1,560 | 65,000 | 1,560 |
| Standard_M96ds_1_v3 | 64 | 65,000 | 1,560 | 65,000 | 1,560 |
| Standard_M96ds_2_v3 | 64 | 130,000 | 3,120 | 130,000 | 3,120 |
| Standard_M176ds_3_v3 | 64 | 130,000 | 4,000 | 130,000 | 4,000 |
| Standard_M176ds_4_v3 | 64 | 130,000 | 4,000 | 130,000 | 4,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.
- <sup>2</sup>Special Storage refers to either [Ultra Disk](../../../virtual-machines/disks-enable-ultra-ssd.md) or [Premium SSD v2](../../../virtual-machines/disks-deploy-premium-v2.md) storage.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_M12ds_v3 | 4 | 4,000 |
| Standard_M24ds_v3 | 8 | 8,000 |
| Standard_M48ds_1_v3 | 8 | 16,000 |
| Standard_M96ds_1_v3 | 8 | 16,000 |
| Standard_M96ds_2_v3 | 8 | 30,000 |
| Standard_M176ds_3_v3 | 8 | 40,000 |
| Standard_M176ds_4_v3 | 8 | 40,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth).
- To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
