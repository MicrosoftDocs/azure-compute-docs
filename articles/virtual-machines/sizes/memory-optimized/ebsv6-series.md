---
title: Ebsv6 size series
description: Information on and specifications of the Esv6-series sizes
author: bansal-misha
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 11/17/2025
ms.author: bansal-misha
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to review the specifications and feature support of various Esv6-series virtual machine sizes, so that I can select the most appropriate size for my workloads and optimize performance in the cloud environment.
---

# Ebsv6 sizes series 

[!INCLUDE [ebsv6-summary](./includes/ebsv6-series-summary.md)]
 

## Host specifications
[!INCLUDE [ebsv6-series-specs](./includes/ebsv6-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_E2bs_v6 | 2 | 16 |
| Standard_E4bs_v6 | 4 | 32 |
| Standard_E8bs_v6 | 8 | 64 |
| Standard_E16bs_v6 | 16 | 128 |
| Standard_E32bs_v6 | 32 | 256 |
| Standard_E48bs_v6 | 48 | 384 |
| Standard_E64bs_v6 | 64 | 512 |
| Standard_E96bs_v6 | 96 | 768 |
| Standard_E128bs_v6 | 128 | 1024 |
| Standard_E192ibs_v6 | 192 | 1832 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_E2bs_v6 | 8 | 8800 | 253 | 30000 | 1200 | 13200 | 330 | 45000 | 1565 |
| Standard_E4bs_v6 | 12 | 17600 | 506 | 35000 | 1200 | 23540 | 660 | 46813 | 1565 |
| Standard_E8bs_v6 | 24 | 36300 | 1023 | 44000 | 1500 | 48620 | 1320 | 58933 | 1935 |
| Standard_E16bs_v6 | 48 | 72600 | 1980 | 77000 | 2500 | 97240 | 2530 | 103133 | 3194 |
| Standard_E32bs_v6 | 64 | 143000 | 4070 | 154000 | 5000 | 191620 | 5280 | 206360 | 6486 |
| Standard_E48bs_v6 | 64 | 209000 | 6160 | 220000 | 7000 | 278630 | 8030 | 293295 | 9124 |
| Standard_E64bs_v6 | 64 | 242000 | 6600 | 253000 | 8000 | 324280 | 8580 | 339020 | 10400 |
| Standard_E96bs_v6 | 64 | 286000 | 8250 | 286000 | 10000 | 429000 | 9900 | 429000 | 12000 |
| Standard_E128bs_v6 | 64 | 330000 | 10000 | 350000 | 11500 | 500000 | 12000 | 530303 | 13800 |
| Standard_E192ibs_v6 | 64 | 400000 | 14000 | 400000 | 14000 | 800000 | 14000 | 800000 | 14000 |

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
| Standard_E2bs_v6 | 2 | 12500 |
| Standard_E4bs_v6 | 2 | 12500 |
| Standard_E8bs_v6 | 4 | 12500 |
| Standard_E16bs_v6 | 8 | 12500 |
| Standard_E32bs_v6 | 8 | 16000 |
| Standard_E48bs_v6 | 8 | 24000 |
| Standard_E64bs_v6 | 8 | 30000 |
| Standard_E96bs_v6 | 8 | 41000 |
| Standard_E128bs_v6 | 8 | 54000 |
| Standard_E192ibs_v6 | 8 | 200000 |

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

