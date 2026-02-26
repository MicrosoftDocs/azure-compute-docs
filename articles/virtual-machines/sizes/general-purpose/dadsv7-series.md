---
title: Dadsv7 size series
description: Information on and specifications of the Dadsv7-series sizes
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 01/26/2026
ms.author: archat
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand the specifications and features of the Dadsv7 series virtual machine sizes, so that I can select the appropriate resources for my applications based on performance and storage requirements."
---

# Dadsv7 sizes series

[!INCLUDE [dadsv7-summary](./includes/dadsv7-series-summary.md)]

## Host specifications
[!INCLUDE [dadsv7-series-specs](./includes/dadsv7-series-specs.md)]

## Feature support
- [Premium Storage](../../premium-storage-performance.md): Supported
- [Premium Storage caching](../../premium-storage-performance.md): Supported
- [Live Migration](../../maintenance-and-updates.md): Supported
- [Memory Preserving Updates](../../maintenance-and-updates.md): Supported
- [Generation 2 VMs](../../generation-2.md): Supported
- [Generation 1 VMs](../../generation-2.md): Not Supported
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported
- [Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml): Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2ads_v7 | 2 | 8 |
| Standard_D4ads_v7 | 4 | 16 |
| Standard_D8ads_v7 | 8 | 32 |
| Standard_D16ads_v7 | 16 | 64 |
| Standard_D32ads_v7 | 32 | 128 |
| Standard_D48ads_v7 | 48 | 192 |
| Standard_D64ads_v7 | 64 | 256 |
| Standard_D96ads_v7 | 96 | 384 |
| Standard_D128ads_v7 | 128 | 512 |
| Standard_D160ads_v7 | 160 | 640 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup>,<sup>2</sup> IOPS<sup>3</sup> | Temp Disk Random Read (RR)<sup>1</sup>,<sup>2</sup> Throughput (MB/s)<sup>3</sup> | Temp Disk Random Write (RW)<sup>1</sup> IOPS | Temp Disk Random Write (RW)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ads_v7 | 1 | 110 | 37500 | 280 | 15000 | 140 |
| Standard_D4ads_v7 | 1 | 220 | 75000 | 560 | 30000 | 280 |
| Standard_D8ads_v7 | 1 | 440 | 150000 | 1120 | 60000 | 560 |
| Standard_D16ads_v7 | 2 | 440 | 300000 | 2240 | 120000 | 1120 |
| Standard_D32ads_v7 | 4 | 440 | 600000 | 4480 | 240000 | 2240 |
| Standard_D48ads_v7 | 6 | 440 | 900000 | 6720 | 360000 | 3360 |
| Standard_D64ads_v7 | 4 | 880 | 1200000 | 8960 | 480000 | 4480 |
| Standard_D96ads_v7 | 6 | 880 | 1800000 | 13440 | 720000 | 6720 |
| Standard_D128ads_v7 | 4 | 1760 | 2400000 | 17920 | 960000 | 8960 |
| Standard_D160ads_v7 | 4 | 2200 | 3000000 | 22400 | 12000000 | 11200 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup> Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed. Temp disk performance can vary based on workload, block size, and system conditions. Published numbers show peak performance under controlled testing, temp disk performance specifications should be viewed as best case performance numbers and may differ in real-world scenarios. Factors such as data patterns, SSD wear, and background processes can affect write speeds. For more details, please see [FAQ for Temp NVMe disks](../../../virtual-machines/enable-nvme-temp-faqs.yml)
- <sup>2</sup> Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- <sup>3</sup> The IOPS and throughput values shown are the combined performance across all temp disks.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ads_v7 | 10 | 4000 | 118 | 44000 | 1412 | 4400 | 136 | 48400 | 1653 |
| Standard_D4ads_v7 | 12 | 8000 | 234 | 47200 | 1412 | 8800 | 273 | 52083 | 1653 |
| Standard_D8ads_v7 | 26 | 16000 | 468 | 47200 | 1412 | 17600 | 547 | 52083 | 1653 |
| Standard_D16ads_v7 | 48 | 32000 | 936 | 72700 | 1412 | 35200 | 1095 | 80000 | 1653 |
| Standard_D32ads_v7 | 64 | 64000 | 1872 | 94400 | 1916 | 70400 | 2191 | 104167 | 2241 |
| Standard_D48ads_v7 | 64 | 96000 | 2808 | 99000 | 2874 | 105600 | 3291 | 108900 | 3362 |
| Standard_D64ads_v7 | 64 | 128000 | 3744 | 132000 | 3832 | 140800 | 4382 | 145200 | 4484 |
| Standard_D96ads_v7 | 64 | 192000 | 5663 | 192500 | 5749 | 211200 | 6573 | 211750 | 6669 |
| Standard_D128ads_v7 | 64 | 204800 | 7488 | 225280 | 7663 | 281600 | 8764 | 310886 | 8966 |
| Standard_D160ads_v7 | 64 | 212000 | 10344 | 242640 | 11410 | 310000 | 10356 | 355443 | 11450 |

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

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D2ads_v7 | 2 | 16000 |
| Standard_D4ads_v7 | 2 | 16000 |
| Standard_D8ads_v7 | 4 | 25000 |
| Standard_D16ads_v7 | 8 | 25000 |
| Standard_D32ads_v7 | 8 | 25000 |
| Standard_D48ads_v7 | 8 | 35000 |
| Standard_D64ads_v7 | 8 | 45000 |
| Standard_D96ads_v7 | 8 | 70000 |
| Standard_D128ads_v7 | 15 | 75000 |
| Standard_D160ads_v7 | 15 | 80000 |

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
> [!NOTE]
> This VM series will only work on OS images that support NVMe. If your current OS image doesn't have NVMe support, you’ll see an error message. [NVMe](../../../virtual-machines/enable-nvme-interface.md) support is available on the most popular OS images, and we're continuously improving OS image compatibility.

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
