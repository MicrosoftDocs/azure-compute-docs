---
title: NGads_V620 size series
description: Information on and specifications of the NGads_V620-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a solution architect, I want to review the specifications and feature support of the NGads_V620 series, so that I can determine the most suitable virtual machine size for my application’s performance needs.
---

# NGads_V620 sizes series

[!INCLUDE [ngads_v620-summary](./includes/ngadsv620-series-summary.md)]

## Host specifications
[!INCLUDE [ngadsv620-series-specs](./includes/ngadsv620-series-specs.md)]

## Feature support
- [Premium Storage](../../premium-storage-performance.md): Supported
- [Premium Storage caching](../../premium-storage-performance.md): Supported
- [Live Migration](../../maintenance-and-updates.md#live-migration): Not Supported
- [Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported
- [Generation 2 VMs](../../generation-2.md): Supported
- [Generation 1 VMs](../../generation-2.md): Supported
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported
- [Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml): Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_NG8ads_V620_v1 | 8 | 16 |
| Standard_NG16ads_V620_v1 | 16 | 32 |
| Standard_NG32ads_V620_v1 | 32 | 64 |
| Standard_NG32adms_V620_v1 | 32 | 176 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Max NVMe Disks (Qty.) | NVMe Disk Size (GiB) |
| --- | --- | --- | --- | --- |
| Standard_NG8ads_V620_v1  | 1 | 256  | 1  | 960 |
| Standard_NG16ads_V620_v1 | 1 | 512 | 2  | 960 |
| Standard_NG32ads_V620_v1 | 1 | 1024 | 4  | 960 |
| Standard_NG32adms_V620_v1 | 1 | 1024 | 4  | 960 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Disk IOPS | Uncached Disk Speed (MBps) |
| --- | --- | --- | --- |
| Standard_NG8ads_V620_v1 | 8 | 12800 | 200 |
| Standard_NG16ads_V620_v1 | 16 | 25600 | 384 |
| Standard_NG32ads_V620_v1 | 32 | 51200 | 768 |
| Standard_NG32adms_V620_v1 | 32 | 51200 | 768 |

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

| Size Name | Max NICs (Qty.) | Max Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_NG8ads_V620_v1 | 2 | 10000 |
| Standard_NG16ads_V620_v1 | 4 | 20000 |
| Standard_NG32ads_V620_v1 | 8 | 40000 |
| Standard_NG32adms_V620_v1 | 8 | 40000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

| Size Name | Accelerators (Qty.) | Accelerator-Memory (GB) |
| --- | --- | --- |
| Standard_NG8ads_V620_v1 | 1/4 | 8 |
| Standard_NG16ads_V620_v1 | 1/2 | 16 |
| Standard_NG32ads_V620_v1 | 1 | 32 |
| Standard_NG32adms_V620_v1 | 1 | 32 |

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
