---
title: Eadsv7 size series
description: Information on and specifications of the Eadsv7-series sizes
author: archatC
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
# Customer intent: "As an IT administrator evaluating virtual machine options, I want to compare specifications of the Eadsv7 size series, so that I can select the most suitable configuration for my workloads and ensure optimal performance."
---

# Eadsv7 sizes series (Preview)

[!INCLUDE [sizes-preview-tag](../includes/sizes-preview-tag.md)]

[!INCLUDE [eadsv7-summary](./includes/eadsv7-series-summary.md)]

## Host specifications
[!INCLUDE [eadsv7-series-specs](./includes/eadsv7-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>
[Azure Disk Encryption for Linux VMs](../../../virtual-machines/linux/disk-encryption-linux.md?tabs=azcliazure%2Cenableadecli%2Cefacli%2Cadedatacli#restrictions): Not Supported <br>
[Azure Disk Encryption for Windows VMs](../../../virtual-machines/windows/disk-encryption-windows.md#restrictions): Not Supported <br>


## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_E2ads_v7 | 2 | 16 |
| Standard_E4ads_v7 | 4 | 32 |
| Standard_E8ads_v7 | 8 | 64 |
| Standard_E16ads_v7 | 16 | 128 |
| Standard_E32ads_v7 | 32 | 256 |
| Standard_E48ads_v7 | 48 | 384 |
| Standard_E64ads_v7 | 64 | 512 |
| Standard_E96ads_v7 | 96 | 768 |
| Standard_E128ads_v7 | 128 | 1024 |
| Standard_E160ads_v7 | 160 | 1280 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- |
| Standard_E2ads_v7 | 1 | 110 | 37500 | 180 |
| Standard_E4ads_v7 | 1 | 220 | 75000 | 360 |
| Standard_E8ads_v7 | 1 | 440 | 150000 | 720 |
| Standard_E16ads_v7 | 2 | 440 | 300000 | 1440 |
| Standard_E32ads_v7 | 4 | 440 | 600000 | 2880 |
| Standard_E48ads_v7 | 6 | 440 | 900000 | 4320 |
| Standard_E64ads_v7 | 4 | 880 | 1200000 | 5760 |
| Standard_E96ads_v7 | 6 | 880 | 1800000 | 8640 |
| Standard_E128ads_v7 | 4 | 1760 | 2400000 | 11520 |
| Standard_E160ads_v7 | 5 | 1760 | 3000000 | 14400 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_E2ads_v7 | 10 | 4000 | 118 | 44000 | 1412 | 4400 | 136 | 48400 | 1653 |
| Standard_E4ads_v7 | 12 | 8000 | 234 | 47200 | 1412 | 8800 | 273 | 52083 | 1653 |
| Standard_E8ads_v7 | 26 | 16000 | 468 | 47200 | 1412 | 17600 | 547 | 52083 | 1653 |
| Standard_E16ads_v7 | 48 | 32000 | 936 | 72700 | 1412 | 35200 | 1095 | 80000 | 1653 |
| Standard_E32ads_v7 | 64 | 64000 | 1872 | 94400 | 1916 | 70400 | 2191 | 104167 | 2241 |
| Standard_E48ads_v7 | 64 | 96000 | 2808 | 99000 | 2874 | 105600 | 3291 | 108900 | 3362 |
| Standard_E64ads_v7 | 64 | 128000 | 3744 | 132000 | 3832 | 140800 | 4382 | 145200 | 4484 |
| Standard_E96ads_v7 | 64 | 192000 | 5663 | 192500 | 5749 | 211200 | 6573 | 211750 | 6669 |
| Standard_E128ads_v7 | 64 | 204800 | 7488 | 225280 | 7663 | 281600 | 8764 | 310886 | 8966 |
| Standard_E160ads_v7 | 64 | 212000 | 10344 | 242640 | 11410 | 310000 | 10356 | 355443 | 11450 |

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
| Standard_E2ads_v7 | 2 | 16000 |
| Standard_E4ads_v7 | 2 | 16000 |
| Standard_E8ads_v7 | 4 | 25000 |
| Standard_E16ads_v7 | 8 | 25000 |
| Standard_E32ads_v7 | 8 | 25000 |
| Standard_E48ads_v7 | 8 | 35000 |
| Standard_E64ads_v7 | 8 | 45000 |
| Standard_E96ads_v7 | 8 | 70000 |
| Standard_E128ads_v7 | 15 | 75000 |
| Standard_E160ads_v7 | 15 | 80000 |

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

