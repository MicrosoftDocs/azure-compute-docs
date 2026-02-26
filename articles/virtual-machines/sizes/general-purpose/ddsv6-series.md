---
title: Ddsv6 size series
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

# Ddsv6 sizes series 

[!INCLUDE [ddsv6-summary](./includes/ddsv6-series-summary.md)]

## Host specifications
[!INCLUDE [ddsv6-series-specs](./includes/ddsv6-series-specs.md)]

## Feature support
- [Premium Storage](../../premium-storage-performance.md): Supported
- [Premium Storage caching](../../premium-storage-performance.md): Supported
- [Live Migration](../../maintenance-and-updates.md#live-migration): Supported
- [Memory Preserving Updates](../../maintenance-and-updates.md): Supported
- [Generation 2 VMs](../../generation-2.md): Supported
- [Generation 1 VMs](../../generation-2.md): Not Supported
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported
- [Temporary local NVMe disks](../../enable-nvme-temp-faqs.yml): Supported
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported 
- [Azure Disk Encryption for Linux VMs](../../../virtual-machines/linux/disk-encryption-linux.md?tabs=azcliazure%2Cenableadecli%2Cefacli%2Cadedatacli#restrictions): Not Supported
- [Azure Disk Encryption for Windows VMs](../../../virtual-machines/windows/disk-encryption-windows.md#restrictions): Not Supported

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

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB)<sup>1</sup> | Temp Disk Random Read (RR)<sup>2</sup> IOPS<sup>3</sup> | Temp Disk Random Read (RR)<sup>2</sup> Throughput (MB/s)<sup>3</sup> | Temp Disk Random Write (RW)<sup>2</sup> IOPS<sup>3</sup> | Temp Disk Random Write (RW)<sup>2</sup> Throughput (MB/s)<sup>3</sup> |
| --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v6 | 1 | 110 | 37500 | 180 | 15000 | 90 |
| Standard_D4ds_v6 | 1 | 220 | 75000 | 360 | 30000 | 180 |
| Standard_D8ds_v6 | 1 | 440 | 150000 | 720 | 60000 | 360 |
| Standard_D16ds_v6 | 2 | 440 | 300000 | 1440 | 120000 | 720 |
| Standard_D32ds_v6 | 4 | 440 | 600000 | 2880 | 240000 | 1440 |
| Standard_D48ds_v6 | 6 | 440 | 900000 | 4320 | 360000 | 2160 |
| Standard_D64ds_v6 | 4 | 880 | 1200000 | 5760 | 480000 | 2880 |
| Standard_D96ds_v6 | 6 | 880 | 1800000 | 8640 | 720000 | 4320 |
| Standard_D128ds_v6 | 4 | 1760 | 2400000 | 11520 | 960000 | 5760 |
| Standard_D192ds_v6 | 6 | 1760 | 3600000 | 17280 | 1440000 | 8640 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup> Total local temporary storage is calculated by multiplying the max number of storage disks with the temp disk size. For example, for the Standard_D192ds_v6, the total local temporary storage capacity is `6 x 1760 GiB = 10560 GiB`.
- <sup>2</sup> Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- <sup>3</sup> The IOPS and throughput values shown are the combined performance across all temp disks.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup>  Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_D2ds_v6 | 8 | 3750 | 106 | 40000 | 1250 | 4167 | 124 | 44444 | 1463 |
| Standard_D4ds_v6 | 12 | 6400 | 212 | 40000 | 1250 | 8333 | 248 | 52083 | 1463 |
| Standard_D8ds_v6 | 24 | 12800 | 424 | 40000 | 1250 | 16667 | 496 | 52083 | 1463 |
| Standard_D16ds_v6 | 48 | 25600 | 848 | 40000 | 1250 | 33333 | 992 | 52083 | 1463 |
| Standard_D32ds_v6 | 64 | 51200 | 1696 | 80000 | 1696 | 66667 | 1984 | 104167 | 1984 |
| Standard_D48ds_v6 | 64 | 76800 | 2544 | 80000 | 2544 | 100000 | 2976 | 104167 | 2976 |
| Standard_D64ds_v6 | 64 | 102400 | 3392 | 102400 | 3392 | 133333 | 3969 | 133333 | 3969 |
| Standard_D96ds_v6 | 64 | 153600 | 5088 | 153600 | 5088 | 200000 | 5953 | 200000 | 5953 |
| Standard_D128ds_v6 | 64 | 204800 | 6782 | 204800 | 6782 | 266667 | 7935 | 266667 | 7935 |
| Standard_D192ds_v6 | 64 | 260000 | 12000 | 260000 | 12000 | 400400 | 12000 | 400400 | 12000 |

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
| Standard_D2ds_v6 | 2 | 12500 |
| Standard_D4ds_v6 | 2 | 12500 |
| Standard_D8ds_v6 | 4 | 12500 |
| Standard_D16ds_v6 | 8 | 12500 |
| Standard_D32ds_v6 | 8 | 16000 |
| Standard_D48ds_v6 | 8 | 24000 |
| Standard_D64ds_v6 | 8 | 30000 |
| Standard_D96ds_v6 | 8 | 41000 |
| Standard_D128ds_v6 | 8 | 54000 |
| Standard_D192ds_v6 | 8 | 82000 |

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
