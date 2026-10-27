---
title: Msv4 Medium Memory size series (Preview)
description: Information on and specifications of the Msv4-series sizes
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 09/03/2026
ms.author: wwilliams
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to assess the Mbsv3 and Msv4 Medium Memory VM series specifications, so that I can determine the best virtual machine options for storage-intensive workloads like SQL Server and data analytics."
---

# Msv4 Medium Memory sizes series (Preview)

[!INCLUDE [msv4-summary](./includes/msv4-series-summary.md)]

## Host specifications
[!INCLUDE [msv4-series-specs](./includes/msv4-series-specs.md)]

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series (NVMe)

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_M16ls_v4 | 16 | 256 |
| Standard_M32ls_v4 | 32 | 512 |
| Standard_M48ls_v4 | 48 | 768 |
| Standard_M64ls_v4 | 64 | 1024 |
| Standard_M96ls_v4 | 96 | 1536 |
| Standard_M128ls_v4 | 128 | 1944 |
| Standard_M192ls_v4 | 192 | 3072 |
| Standard_M256ls_v4 | 256 | 3892 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage information for each size:

> [!NOTE]
> This series doesn't have local storage.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).



### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage information for each size:

| Size Name | Max Remote Storage Disks (Qty.) | Max Uncached Premium SSD IOPS | Max Uncached Premium SSD Throughput (MB/s) | Max Uncached Ultra Disk and Premium SSD v2 IOPS | Max Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_M16ls_v4  | 64 | 20,000 | 20,000 | 400 | 400 |
| Standard_M32ls_v4 | 64 | 40,000 | 40,000 | 800 | 800 |
| Standard_M48ls_v4 | 64 | 65,000 | 65,000 | 1,600 | 1,600 |
| Standard_M64ls_v4 | 64 | 80,000 | 80,000 | 1,600 | 1,600 |
| Standard_M96ls_v4 | 64 | 120,000 | 120,000 | 3,200 | 3,200 |
| Standard_M128ls_v4 | 64 | 160,000 | 160,000 | 4,000 | 4,000 |
| Standard_M192ls_v4 | 64 | 200,000 | 200,000 | 4,750 | 4,750 |
| Standard_M256ls_v4 | 64 | 225,000 | 225,000 | 5,500 | 5,500 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3), remember that capacity numbers given in GiB might appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- IOPS and MBps listed here refer to uncached mode for data disks.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).
- IOPS spec is defined using common small random block sizes like 4KiB or 8KiB. Maximum IOPS is defined as "up-to" and measured using 4KiB random reads workloads.
- TPUT spec is defined using common large sequential block sizes like 128KiB or 1024KiB. Maximum TPUT is defined as "up-to" and measured using 128KiB sequential reads workloads.


### [Network](#tab/sizenetwork)

Network interface information for each size:

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_M16ls_v4 | 8 | 4,500 |
| Standard_M32ls_v4 | 8 | 8,000 |
| Standard_M48ls_v4 | 8 | 16,000 |
| Standard_M64ls_v4 | 8 | 16,000 |
| Standard_M96ls_v4 | 8 | 30,000 |
| Standard_M128ls_v4 | 8 | 30,000 |
| Standard_M192ls_v4 | 8 | 50,000 |
| Standard_M256ls_v4 | 8 | 50,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/articles/virtual-network/virtual-machine-network-throughput).
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/articles/virtual-network/virtual-network-optimize-network-bandwidth).
- To achieve the expected network performance on Linux or Windows, you might need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/support/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> This series doesn't include any accelerators.

---
## Feature support

| Feature name | Support status |
| --- | --- |
| [Premium Storage](../../premium-storage-performance.md) | Supported |
| [Premium Storage caching](../../premium-storage-performance.md) | Supported |
| [Live Migration](../../maintenance-and-updates.md) | Limited Support |
| [Memory Preserving Updates](../../maintenance-and-updates.md) | Not Supported |
| [Generation 2 VMs](../../generation-2.md) | Supported |
| [Generation 1 VMs](../../generation-2.md) | Not Supported |
| [Accelerated Networking](/azure/virtual-network/create-virtual-machine-accelerated-networking) | Supported |
| [Ephemeral OS Disk](../../ephemeral-os-disks.md) | Not Supported |
| [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) | Not Supported |
| [Hibernation](../../hibernate-resume.md) | Not Supported |
| [Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator) | Not Supported |

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
