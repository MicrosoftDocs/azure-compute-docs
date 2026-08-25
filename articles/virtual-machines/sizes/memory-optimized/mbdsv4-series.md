---
title: Mbdsv4 size series (Preview)
description: Information on and specifications of the Mbdsv4-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/05/2026
ms.author: zhangjay
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to assess the Mbdsv4 VM series specifications, so that I can determine the best virtual machine options for storage-intensive workloads like SQL Server and data analytics."
---

# Mbdsv4 sizes series (Preview)

[!INCLUDE [mbdsv4-summary](./includes/mbdsv4-series-summary.md)]

## Host specifications
[!INCLUDE [mbdsv4-series-specs](./includes/mbdsv4-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Limited Support <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-virtual-machine-accelerated-networking): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>[Hibernation](../../hibernate-resume.md): Not Supported <br> [Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Not Supported

## Sizes in series (NVMe)

### [Basics](#tab/sizebasic)

vCPUs and memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_M16bds_v4 | 16 | 128 |
| Standard_M32bds_v4 | 32 | 256 |
| Standard_M48bds_v4 | 48 | 384 |
| Standard_M64bds_v4 | 64 | 512 |
| Standard_M96bds_1_v4 | 96 | 768 |
| Standard_M128bds_1_v4 | 128 | 1024 |
| Standard_M64bds_1_v4 | 64 | 1440 |
| Standard_M96bds_2_v4 | 96 | 1946 |
| Standard_M128bds_3_v4 | 128 | 2796 |
| Standard_M176bds_4_v4 | 176 | 3892 |
| Standard_M256bds_2_v4 | 256 | 2048 |
| Standard_M304bds_4_v4 | 304 | 3800 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

| Size Name | MAX Temp Storage Disks (Qty.) | Temp Storage (GiB) | Temp Disk Random Read IOPS | Temp Disk Sequential ReadThroughput (MBps) | Temp Disk Random Write IOPS | Temp Disk Sequential Write Throughpit (MBps) |
| --- | --- | --- | --- | --- | --- |
| Standard_M16bds_v4 | 2 | 400 | 200,000 | 2,000 | 100,000 | 1,000 |
| Standard_M32bds_v4 | 2 | 400 | 400,000 | 4,000 | 200,000 | 2,000 |
| Standard_M48bds_v4 | 2 | 400 | 400,000 | 4,000 | 200,000 | 2,000 |
| Standard_M64bds_v4 | 2 | 800 | 800,000 | 6,000 | 400,000 | 3,000 |
| Standard_M96bds_1_v4 | 2 | 800 | 1,200,000 | 8,000 | 600,000 | 4,000 |
| Standard_M128bds_1_v4 | 2 | 800 | 1,600,000 | 10,000 | 800,000 | 5,000 |
| Standard_M64bds_1_v4 | 2 | 1500 | 1,200,000 | 8,000 | 600,000 | 4,000 |
| Standard_M96bds_2_v4 | 2 | 2250 | 1,600,000 | 10,000 | 800,000 | 5,000 |
| Standard_M128bds_3_v4 | 2 | 3000 | 2,000,000 | 12,000 | 1,000,000 | 6,000 |
| Standard_M176bds_4_v4 | 2 | 4000 | 3,000,000 | 16,000 | 1,500,000 | 8,000 |
| Standard_M256bds_2_v4 | 2 | 3000 | 2,000,000 | 12,000 | 1,000,000 | 6,000 |
| Standard_M304bds_4_v4 | 2 | 6000 | 3,200,000 | 16,000 | 1,600,000 | 8,000 |


#### Table definitions
- Total local temporary storage is calculated by multiplying the max number of storage disks with the temp disk size. For example, for the Standard_M304bds_4_v4, the total local temporary storage capacity is 2 x 6000 GiB = 12000 GiB. 
- Temp disk performance depends on many factors including block size, workload patterns of read/writes, queue depth (QD), and others. Temp disk performance specifications should be viewed as best case performance numbers, assuming 4k block sizes and QD=256 for IOPS, and 256k block sizes with QD=64 for throughput. Read performance specs assume 100% reads, and write performance specs assume 100% writes. Additionally, write performance is heavily impacted by how many blocks in use on a device. Temp disk write performance specs assume a device has a clean slate to enable the best performance. During steady state operations, write performance is expected to be lower than the published specs. - IOPS/MBps listed here refer to uncached mode for data disks.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB. 
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec. 
- To learn how to get the best storage performance for your VMs, see Virtual machine and disk performance. 



### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Max Uncached Premium SSD IOPS | Max Uncached Premium SSD Throughput (MB/s) | Max Uncached Ultra Disk and Premium SSD v2 IOPS | Max Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_M16bds_v4 | 64 | 64,000 | 2,000 | 64,000 | 2,000 |
| Standard_M32bds_v4 | 64 | 88,000 | 4,000 | 88,000 | 4,000 |
| Standard_M48bds_v4 | 64 | 88,000 | 4,000 | 130,000 | 4,000 |
| Standard_M64bds_v4 | 64 | 160,000 | 6,000 | 160,000 | 6,000 |
| Standard_M96bds_1_v4 | 64 | 260,000 | 12,000 | 400,000 | 12,000 |
| Standard_M128bds_1_v4 | 64 | 400,000 | 12,000 | 500,000 | 12,000 |
| Standard_M64bds_1_v4 | 64 | 160,000 | 6,000 | 160,000 | 6,000 |
| Standard_M96bds_2_v4 | 64 | 260,000 | 12,000 | 400,000 | 12,000 |
| Standard_M128bds_3_v4 | 64 | 400,000 | 14,000 | 500,000 | 16,000 |
| Standard_M176bds_4_v4 | 64 | 500,000 | 14,000 | 780,000| 16,000 |
| Standard_M256bds_2_v4 | 64 | 500,000 | 14,000 | 780,000 | 16,000 |
| Standard_M304bds_4_v4 | 64 | 500,000 | 14,000 | 780,000 | 16,000 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB might appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- IOPS and MBps listed here refer to uncached mode for data disks.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).
- The IOPS specification uses common small random block sizes like 4 KiB or 8 KiB. Maximum IOPS is an "up-to" value and is measured using 4 KiB random reads workloads.
- The throughput specification uses common large sequential block sizes like 128 KiB or 1024 KiB. Maximum throughput is an "up-to" value and is measured using 128 KiB sequential reads workloads.


### [Network](#tab/sizenetwork)

Network interface information for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_M16bds_v4 | 8 | 16,000 |
| Standard_M32bds_v4 | 8 | 16,000 |
| Standard_M48bds_v3 | 8 | 24,000 |
| Standard_M64bds_v4 | 8 | 24,000 |
| Standard_M96bds_1_v4 | 8 | 50,000 |
| Standard_M128bds_1_v4 | 8 | 80,000 |
| Standard_M64bds_1_v4 | 8 | 24,000 |
| Standard_M96bds_2_v4 | 8 | 50,000 |
| Standard_128bds_3_v4 | 8 | 80,000 |
| Standard_176bds_4_v4 | 8 | 100,000 |
| Standard_M256bds_2_v4 | 8 | 100,000 |
| Standard_M304bds_4_v4 | 8 | 100,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual Machine network bandwidth](/virtual-network/virtual-machine-network-throughput).
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance depends on several factors, including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/virtual-network/virtual-network-optimize-network-bandwidth). 
- To achieve the expected network performance on Linux or Windows, you might need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, and other accelerators) information for each size.

> [!NOTE]
> This series doesn't include any accelerators.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
