---
title: Ddsv7 size series
description: Information on and specifications of the Ddsv6-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to evaluate the Ddsv6 size series specifications, so that I can choose the appropriate virtual machine size for my applications based on their resource requirements and performance characteristics."
---

# Ddsv7 sizes series 
[!INCLUDE [sizes-preview-tag](../includes/sizes-preview-tag.md)]

[!INCLUDE [ddsv7-summary](./includes/ddsv7-series-summary.md)]

## Host specifications
[!INCLUDE [ddsv7-series-specs](./includes/ddsv7-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br> 
## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_D2ds_v7 | 2 | 8 |
| Standard_D4ds_v7 | 4 | 16 |
| Standard_D8ds_v7 | 8 | 32 |
| Standard_D16ds_v7 | 16 | 64 |
| Standard_D32ds_v7 | 32 | 128 |
| Standard_D48ds_v7 | 48 | 192 |
| Standard_D64ds_v7 | 64 | 256 |
| Standard_D96ds_v7 | 96 | 384 |
| Standard_D128ds_v7 | 128 | 512 |
| Standard_D192ds_v7 | 192 | 768 |
| Standard_D248ds_v7 | 248 | 992 |
| Standard_D372ds_v7 | 372 | 1,488 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read<sup>1</sup> IOPS | Temp Disk Sequential Read <sup>1</sup> Throughput (MBps) | Temp Disk Random Write <sup>1</sup> IOPS | Temp Disk Sequential Write<sup>1</sup> Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v7 | 1 | 110 | 50,000 | 280 | 25,000 | 140 |
| Standard_D4ds_v7 | 1 | 220 | 100,000 | 560 | 50,000 | 280 |
| Standard_D8ds_v7 | 1 | 440 | 200,000 | 1,120 | 100,000 | 560 |
| Standard_D16ds_v7 | 2 | 440 | 400,000 | 2,240 | 200,000 | 1,120 |
| Standard_D32ds_v7 | 4 | 440 | 800,000 | 4,480 | 400,000 | 2,240 |
| Standard_D48ds_v7 | 6 | 440 | 1,200,000 | 6,720 | 600,000 | 3,360 |
| Standard_D64ds_v7 | 4 | 880 | 1,600,000 | 8,960 | 800,000 | 4,480 |
| Standard_D96ds_v7 | 6 | 880 | 2,400,000 | 13,440 | 1,200,000 | 6,720 |
| Standard_D128ds_v7 | 4 | 1,760 | 3,200,000 | 17,920 | 1,600,000 | 8,960 |
| Standard_D192ds_v7 | 6 | 1,760 | 4,800,000 | 26,880 | 2,400,000 | 13,440 |
| Standard_D248ds_v7 | 8 | 1,760 | 6,400,000 | 35,840 | 3,200,000 | 17,000 |
| Standard_D372ds_v7 | 6 | 3,520 | 9,600,000 | 53,760 | 4,800,000 | 20,400 |

#### Storage resources
- [NVMe Overview](/azure/virtual-machines/nvme-overview)
- [FAQ for temp NVMe disks](/azure/virtual-machines/enable-nvme-temp-faqs)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 Disk Throughput (MBps) |
| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v7 | 10 | 4,000 | 118 | 45,000 | 1,413 | 5,000 | 137 | 56,250 | 1,640 |
| Standard_D4ds_v7 | 12 | 8,000 | 236 | 45,000 | 1,413 | 10,000 | 274 | 56,250 | 1,640 |
| Standard_D8ds_v7 | 26 | 16,000 | 472 | 45,000 | 1,413 | 20,000 | 548 | 56,250 | 1,640 |
| Standard_D16ds_v7 | 48 | 32,000 | 944 | 75,000 | 1,413 | 40,000 | 1,096 | 93,750 | 1,640 |
| Standard_D32ds_v7 | 64 | 64,000 | 1,888 | 100,000 | 1,916 | 80,000 | 2,192 | 125,000 | 2,225 |
| Standard_D48ds_v7 | 64 | 96,000 | 2,832 | 160,000 | 2,875 | 120,000 | 3,288 | 200,000 | 3,338 |
| Standard_D64ds_v7 | 64 | 128,000 | 3,776 | 160,000 | 3,833 | 160,000 | 4,383 | 200,000 | 4,450 |
| Standard_D96ds_v7 | 64 | 192,000 | 5,664 | 200,000  | 5,749 | 240,000 | 6,576 | 205,000 | 6,675 |
| Standard_D128ds_v7 | 64 | 204,800 | 7,552 | 225,280 | 7,664 | 320,000 | 8,768 | 352,000 | 8,898 |
| Standard_D192ds_v7 | 64 | 307,200 | 11,328 | 350,000 | 12,000 | 480,000 | 13,152 | 546,875 | 13,932 |
| Standard_D248ds_v7 | 64 | 396,800 | 13,144 | 500,000 | 15,000 | 620,000 | 16,988 | 781,250 | 19,387 |
| Standard_D372ds_v7 | 64 | 500,000 | 16,000 | 500,000 | 16,000 | 800,000 | 20,000 | 800,000 | 20,000 |

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

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D2ds_v7 | 3 | 16,000 |
| Standard_D4ds_v7 | 4 | 25,000 |
| Standard_D8ds_v7 | 4 | 25,000 |
| Standard_D16ds_v7 | 8 | 25,000 |
| Standard_D32ds_v7 | 8 | 25,000 |
| Standard_D48ds_v7 | 8 | 35,000 |
| Standard_D64ds_v7 | 15 | 45,000 |
| Standard_D96ds_v7 | 15 | 70,000 |
| Standard_D128ds_v7 | 15 | 85,000 |
| Standard_D192ds_v7 | 15 | 100,000 |
| Standard_D258ds_v7 | 15 | 150,000 |
| Standard_D372ds_v7 | 15 | 200,000 |

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
