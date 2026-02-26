---
title: Mbdsv3 size series
description: Information on and specifications of the Mbdsv3-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/24/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
---

# Mbdsv3 sizes series

[!INCLUDE [mbdsv3-summary](./includes/mbdsv3-series-summary.md)]

## Host specifications
[!INCLUDE [mbdsv3-series-specs](./includes/mbdsv3-series-specs.md)]

## Feature support
- [Premium Storage](../../premium-storage-performance.md): Supported
- [Premium Storage caching](../../premium-storage-performance.md): Supported
- [Live Migration](../../maintenance-and-updates.md#live-migration): Supported
- [Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported
- [Generation 2 VMs](../../generation-2.md): Supported
- [Generation 1 VMs](../../generation-2.md): Not Supported
- [Accelerated Networking](/azure/virtual-network/create-virtual-machine-accelerated-networking): Supported
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported
- [Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml): Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported
- [Hibernation](../../hibernate-resume.md): Not Supported
- [Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Sizes in series (NVMe)

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_M16bds_v3 | 16 | 128 |
| Standard_M32bds_v3 | 32 | 256 |
| Standard_M48bds_v3 | 48 | 384 |
| Standard_M64bds_v3 | 64 | 512 |
| Standard_M96bds_v3 | 96 | 768 |
| Standard_M128bds_v3 | 128 | 1,024 |
| Standard_M176bds_v3 | 176 | 1,536 |
| Standard_M64bds_1_v3 | 64 | 1397 |
| Standard_M96bds_2_v3 | 96 | 1946 |
| Standard_M128bds_3_v3 | 128 | 2794 |
| Standard_M176bds_4_v3 | 176 | 3892 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Max Temp Disk Random Read (RR)<sup>1</sup> IOPS | Max Temp Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- |
| Standard_M16bds_v3 | 1 | 400 | 10,000 | 100 |
| Standard_M32bds_v3 | 1 | 400 | 20,000 | 200 |
| Standard_M48bds_v3 | 1 | 400 | 40,000 | 400 |
| Standard_M64bds_v3 | 1 | 400 | 40,000 | 400 |
| Standard_M96bds_v3 | 1 | 400 | 40,000 | 400 |
| Standard_M128bds_v3 | 1 | 400 | 160,000 | 1600 |
| Standard_M176bds_v3 | 1 | 400 | 160,000 | 1600 |
| Standard_M64bds_1_v3 | 1 | 3000 | 40,000 | 400 |
| Standard_M96bds_2_v3 | 1 | 4500 | 40,000 | 400 |
| Standard_M128bds_3_v3 | 1 | 6000 | 160,000 | 1600 |
| Standard_M176bds_4_v3 | 1 | 8000 | 160,000 | 1600 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- IOPS/MBps listed here refer to uncached mode for data disks.
- To learn how to get the best storage performance for your VMs, see Virtual machine and disk performance.
- IOPS spec is defined using common small random block sizes like 4KiB or 8KiB. Maximum IOPS is defined as "up-to" and measured using 4KiB random reads workloads.
- TPUT spec is defined using common large sequential block sizes like 128KiB or 1024KiB. Maximum TPUT is defined as "up-to" and measured using 128KiB sequential reads workloads.

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Max Uncached Premium SSD IOPS | Max Uncached Premium SSD Throughput (MB/s) | Max Uncached Ultra Disk and Premium SSD v2 IOPS | Max Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_M16bds_v3 | 64 | 44,000 | 1,000 |64,000 | 1,000 |
| Standard_M32bds_v3 | 64 | 88,000 | 2,000 |88,000 | 2,000 |
| Standard_M48bds_v3 | 64 | 88,000 | 2,000 |120,000 | 2,000 |
| Standard_M64bds_v3 | 64 | 88,000 | 2,000 |160,000 | 2,000 |
| Standard_M96bds_v3 | 64 | 260,000 | 4,000 |260,000 | 4,000 |
| Standard_M128bds_v3 | 64 | 260,000 | 4,000 |400,000 | 4,000 |
| Standard_M176bds_v3 | 64 | 260,000 | 6,000 |650,000 | 6,000 |
| Standard_M64bds_1_v3 | 64 | 130,000 | 6,000 |160, 000 | 6,000 |
| Standard_M96bds_2_v3 | 64 | 130,000 | 8,000 |260,000 | 8,000 |
| Standard_M128bds_3_v3 | 64 | 260,000 | 8,000 |400,000 | 10,000 |
| Standard_M176bds_4_v3 | 64 | 260,000 | 8,000 |650,000 | 10,000 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- IOPS/MBps listed here refer to uncached mode for data disks.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).
- IOPS spec is defined using common small random block sizes like 4KiB or 8KiB. Maximum IOPS is defined as "up-to" and measured using 4KiB random reads workloads.
- TPUT spec is defined using common large sequential block sizes like 128KiB or 1024KiB. Maximum TPUT is defined as "up-to" and measured using 128KiB sequential reads workloads.


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_M16bds_v3 | 8 | 8,000 |
| Standard_M32bds_v3 | 8 | 16,000 |
| Standard_M48bds_v3 | 8 | 16,000 |
| Standard_M64bds_v3 | 8 | 16,000 |
| Standard_M96bds_v3 | 8 | 25,000 |
| Standard_M128bds_v3 | 8 | 40,000 |
| Standard_M176bds_v3 | 8 | 50,000 |
| Standard_M64bds_1_v3 | 8 | 20,000 |
| Standard_M96bds_2_v3 | 8 | 20,000 |
| Standard_M128bds_3_v3 | 8 | 40,000 |
| Standard_M176bds_4_v3 | 8 | 40,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- IOPS/MBps listed here refer to uncached mode for data disks.
- To learn how to get the best storage performance for your VMs, see Virtual machine and disk performance.
- IOPS spec is defined using common small random block sizes like 4KiB or 8KiB. Maximum IOPS is defined as "up-to" and measured using 4KiB random reads workloads.
- TPUT spec is defined using common large sequential block sizes like 128KiB or 1024KiB. Maximum TPUT is defined as "up-to" and measured using 128KiB sequential reads workloads.

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
