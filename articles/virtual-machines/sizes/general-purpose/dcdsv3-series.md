---
title: DCdsv3 size series
description: Information on and specifications of the DCdsv3-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to review the specifications and features of the DCdsv3 series virtual machines, so that I can select the right instance type for my applications based on cores, memory, storage options, and networking capabilities.
---

# DCdsv3 sizes series

[!INCLUDE [dcdsv3-summary](./includes/dcdsv3-series-summary.md)]

## Host specifications
[!INCLUDE [dcdsv3-series-specs](./includes/dcdsv3-series-specs.md)]

## Feature support
- [Premium Storage](../../premium-storage-performance.md): Supported
- [Premium Storage caching](../../premium-storage-performance.md): Not Supported
- [Live Migration](../../maintenance-and-updates.md): Not Supported
- [Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported
- [Generation 2 VMs](../../generation-2.md): Supported
- [Generation 1 VMs](../../generation-2.md): Not Supported
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported
- [Local temporary storage](../../overview.md#local-temporary-storage): Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported

## Sizes in series

### [Basics](#tab/sizebasic)

Cores (Qty.) and Memory for each size

| Size Name | Cores (Qty.) | Memory (GB) | EPC Memory (GiB) |
| --- | --- | --- | --- |
| Standard_DC1ds_v3 | 1 | 8 | 4 |
| Standard_DC2ds_v3 | 2 | 16 | 8 |
| Standard_DC4ds_v3 | 4 | 32 | 16 |
| Standard_DC8ds_v3 | 8 | 64 | 32 |
| Standard_DC16ds_v3 | 16 | 128 | 64 |
| Standard_DC24ds_v3 | 24 | 192 | 128 |
| Standard_DC32ds_v3 | 32 | 256 | 192 |
| Standard_DC48ds_v3 | 48 | 384 | 256 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) |
| --- | --- | --- |
| Standard_DC1ds_v3 | 1 | 75 |
| Standard_DC2ds_v3 | 1 | 150 |
| Standard_DC4ds_v3 | 1 | 300 |
| Standard_DC8ds_v3 | 1 | 600 |
| Standard_DC16ds_v3 | 1 | 1200 |
| Standard_DC24ds_v3 | 1 | 1800 |
| Standard_DC32ds_v3 | 1 | 2400 |
| Standard_DC48ds_v3 | 1 | 2400 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) |
| --- | --- |
| Standard_DC1ds_v3 | 4 |
| Standard_DC2ds_v3 | 8 |
| Standard_DC4ds_v3 | 16 |
| Standard_DC8ds_v3 | 32 |
| Standard_DC16ds_v3 | 32 |
| Standard_DC24ds_v3 | 32 |
| Standard_DC32ds_v3 | 32 |
| Standard_DC48ds_v3 | 32 |

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

| Size Name | Max NICs (Qty.) |
| --- | --- |
| Standard_DC1ds_v3 | 2 |
| Standard_DC2ds_v3 | 2 |
| Standard_DC4ds_v3 | 4 |
| Standard_DC8ds_v3 | 8 |
| Standard_DC16ds_v3 | 8 |
| Standard_DC24ds_v3 | 8 |
| Standard_DC32ds_v3 | 8 |
| Standard_DC48ds_v3 | 8 |

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


