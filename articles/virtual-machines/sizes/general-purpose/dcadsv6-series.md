---
title: DCadsv6 size series
description: Information on and specifications of the DCadsv6-series sizes
author: linuxelf001
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: raginjup
ms.reviewer: raginjup
---

# DCadsv6 sizes series
[!INCLUDE [dcadsv6-summary](./includes/dcadsv6-series-summary.md)]

## Host specifications
[!INCLUDE [dcadsv6-series-specs](./includes/dcadsv6-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_DC2ads_v6 | 2 | 8 |
| Standard_DC4ads_v6 | 4 | 16 |
| Standard_DC8ads_v6 | 8 | 32 |
| Standard_DC16ads_v6 | 16 | 64 |
| Standard_DC32ads_v6 | 32 | 128 |
| Standard_DC48ads_v6 | 48 | 192 |
| Standard_DC64ads_v6 | 64 | 256 |
| Standard_DC96ads_v6 | 96 | 384 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MBps) |
| --- | --- | --- | --- | --- |
| Standard_DC2ads_v6 | 1 | 75 | 9,000 | 125 |
| Standard_DC4ads_v6 | 1 | 150 | 19,000 | 250 |
| Standard_DC8ads_v6 | 1 | 300 | 38,000 | 500 |
| Standard_DC16ads_v6 | 1 | 600 | 75,000 | 1,000 |
| Standard_DC32ads_v6 | 1 | 1,200 | 150,000 | 2000 |
| Standard_DC48ads_v6 | 1 | 1,800 | 225,000 | 3,000 |
| Standard_DC64ads_v6 | 1 | 2,400 | 300,000 | 4,000 |
| Standard_DC96ads_v6 | 1 | 3,600 | 450,000 | 4,000 |


#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions

- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_DC2ads_v6 | 2 | 12,500 |
| Standard_DC4ads_v6 | 2 | 12,500 |
| Standard_DC8ads_v6 | 4 | 12,500 |
| Standard_DC16ads_v6 | 8 | 12,500 |
| Standard_DC32ads_v6 | 8 | 16,000 |
| Standard_DC48ads_v6 | 8 | 24,000 |
| Standard_DC64ads_v6 | 8 | 30,000 |
| Standard_DC96ads_v6 | 8 | 34,000 |

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



## Feature support
|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Supported |
|[Live Migration](../../maintenance-and-updates.md)| Not Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Not Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Not Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Supported |
|[Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Not Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
