---
title: Ddsv6 size series
description: Information on and specifications of the Ddsv6-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to evaluate the Ddsv6 size series specifications, so that I can choose the appropriate virtual machine size for my applications based on their resource requirements and performance characteristics."
---

# Ddsv6 sizes series 

[!INCLUDE [ddsv6-summary](./includes/ddsv6-series-summary.md)]

## Host specifications
[!INCLUDE [ddsv6-series-specs](./includes/ddsv6-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2ds_v6 | 2 | 8 |
| Standard_D4ds_v6 | 4 | 16 |
| Standard_D8ds_v6 | 8 | 32 |
| Standard_D16ds_v6 | 16 | 64 |
| Standard_D32ds_v6 | 32 | 128 |
| Standard_D48ds_v6 | 48 | 192 |
| Standard_D64ds_v6 | 64 | 256 |
| Standard_D96ds_v6 | 96 | 384 |
| Standard_D128ds_v6 | 128 | 512 |
| Standard_D192ds_v6 | 192 | 768 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB)<sup>1</sup> | Temp Disk Random Read (RR)<sup>2</sup> IOPS<sup>3</sup> | Temp Disk Random Read (RR)<sup>2</sup> Throughput (MBps)<sup>3</sup> | Temp Disk Random Write (RW)<sup>2</sup> IOPS<sup>3</sup> | Temp Disk Random Write (RW)<sup>2</sup> Throughput (MBps)<sup>3</sup> |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v6 | 1 | 110 | 37,500 | 180 | 15,000 | 90 |
| Standard_D4ds_v6 | 1 | 220 | 75,000 | 360 | 30,000 | 180 |
| Standard_D8ds_v6 | 1 | 440 | 150,000 | 720 | 60,000 | 360 |
| Standard_D16ds_v6 | 2 | 440 | 300,000 | 1,440 | 120,000 | 720 |
| Standard_D32ds_v6 | 4 | 440 | 600,000 | 2,880 | 240,000 | 1,440 |
| Standard_D48ds_v6 | 6 | 440 | 900,000 | 4,320 | 360,000 | 2,160 |
| Standard_D64ds_v6 | 4 | 880 | 1,200,000 | 5,760 | 480,000 | 2,880 |
| Standard_D96ds_v6 | 6 | 880 | 1,800,000 | 8,640 | 720,000 | 4,320 |
| Standard_D128ds_v6 | 4 | 1,760 | 2,400,000 | 11,520 | 960,000 | 5,760 |
| Standard_D192ds_v6 | 6 | 1,760 | 3,600,000 | 17,280 | 1,440,000 | 8,640 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup> Total local temporary storage is calculated by multiplying the max number of storage disks with the temp disk size. For example, for the Standard_D192ds_v6, the total local temporary storage capacity is `6 x 1,760 GiB = 10,560 GiB`.
- <sup>2</sup> Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- <sup>3</sup> The IOPS and throughput values shown are the combined performance across all temp disks.
- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MBps) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MBps) | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 Throughput (MBps) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v6 | 8 | 3,750 | 106 | 40,000 | 1,250 | 4,167 | 124 | 44,444 | 1,463 |
| Standard_D4ds_v6 | 12 | 6,400 | 212 | 40,000 | 1,250 | 8,333 | 248 | 52,083 | 1,463 |
| Standard_D8ds_v6 | 24 | 12,800 | 424 | 40,000 | 1,250 | 16,667 | 496 | 52,083 | 1,463 |
| Standard_D16ds_v6 | 48 | 25,600 | 848 | 40,000 | 1,250 | 33,333 | 992 | 52,083 | 1,463 |
| Standard_D32ds_v6 | 64 | 51,200 | 1,696 | 80,000 | 1,696 | 66,667 | 1984 | 104,167 | 1984 |
| Standard_D48ds_v6 | 64 | 76,800 | 2,544 | 80,000 | 2,544 | 100,000 | 2,976 | 104,167 | 2,976 |
| Standard_D64ds_v6 | 64 | 102,400 | 3,392 | 102,400 | 3,392 | 133,333 | 3,969 | 133,333 | 3,969 |
| Standard_D96ds_v6 | 64 | 153,600 | 5,088 | 153,600 | 5,088 | 200,000 | 5,953 | 200,000 | 5,953 |
| Standard_D128ds_v6 | 64 | 204,800 | 6,782 | 204,800 | 6,782 | 266,667 | 7,935 | 266,667 | 7,935 |
| Standard_D192ds_v6 | 64 | 260,000 | 12,000 | 260,000 | 12,000 | 400,400 | 12,000 | 400,400 | 12,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.
- Storage capacity is shown in units of GiB or 1,024^3 bytes. When you compare disks measured in GB (1,000^3 bytes) to disks measured in GiB (1,024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1,023 GiB = 1,098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_D2ds_v6 | 2 | 12,500 |
| Standard_D4ds_v6 | 2 | 12,500 |
| Standard_D8ds_v6 | 4 | 12,500 |
| Standard_D16ds_v6 | 8 | 12,500 |
| Standard_D32ds_v6 | 8 | 16,000 |
| Standard_D48ds_v6 | 8 | 24,000 |
| Standard_D64ds_v6 | 8 | 30,000 |
| Standard_D96ds_v6 | 8 | 41,000 |
| Standard_D128ds_v6 | 8 | 54,000 |
| Standard_D192ds_v6 | 8 | 82,000 |

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
|[Live Migration](../../maintenance-and-updates.md#live-migration)| Supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Supported |
|[Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml)| Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Supported |
|[Azure Disk Encryption for Linux VMs](../../../virtual-machines/linux/disk-encryption-linux.md?tabs=azcliazure%2Cenableadecli%2Cefacli%2Cadedatacli#restrictions)| Not Supported |
|[Azure Disk Encryption for Windows VMs](../../../virtual-machines/windows/disk-encryption-windows.md#restrictions)| Not Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
