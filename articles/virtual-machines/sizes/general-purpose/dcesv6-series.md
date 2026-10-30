---
title: DCesv6-series
description: Information on and specifications of the DCesv6-series sizes
author: michamcr
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 03/10/2026
ms.author: mmcrey
# Customer intent: As a cloud architect, I want to review the specifications and features of the DCesv6-series virtual machine sizes, so that I can select the appropriate size for my applications based on performance and resource needs.
---

# DCesv6-series sizes

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs 

The DCesv6-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that help protect the confidentiality and integrity of code and data while it's being processed. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. These machines are powered by Intel® 5th Generation Xeon® Scalable processors reaching an all-core turbo clock speed of 3.0 GHz and [Intel® AMX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

By using [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code, and administrators access to the VM memory and state. It helps to protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

The DCesv6-series offers a balance of memory to vCPU performance. This series is designed for general computing workloads, e-commerce systems, web front ends, desktop virtualization solutions, sensitive databases, other enterprise applications.

## Host specifications

| Part | Quantity min - max | Specs <br><sup>SKU ID, Performance Units, etc.</sup> |
| --- | --- | --- |
| Processor | 2 - 128 vCPUs | Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory | 8 - 512 GiB | |
| Local Storage | None | |
| Remote Storage | 8 - 64 Disks | 3,750 - 204,800 IOPS <br>106 - 4,000 MBps |
| Network | 2 - 8 NICs | 7,000 - 40,000 Mbps |
| Accelerators | None | |

For features supported by this series, see the [Feature support](#feature-support) section.

## Sizes in series

### [Basics](#tab/sizebasic)

This table shows the number of vCPUs and amount of memory for each DCesv6-series size.

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_DC2es_v6 | 2 | 8 |
| Standard_DC4es_v6 | 4 | 16 |
| Standard_DC8es_v6 | 8 | 32 |
| Standard_DC16es_v6 | 16 | 64 |
| Standard_DC32es_v6 | 32 | 128 |
| Standard_DC48es_v6 | 48 | 192 |
| Standard_DC64es_v6 | 64 | 256 |
| Standard_DC96es_v6 | 96 | 384 |
| Standard_DC128es_v6 | 128 | 512 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage available for each size.

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).



### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage available for each size.

| Size Name | Max Remote Storage Disks | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) |
| --- | --- | --- | --- |
| Standard_DC2es_v6 | 8 | 3,750 | 106 |
| Standard_DC4es_v6 | 12 | 6,400 | 212 |
| Standard_DC8es_v6 | 24 | 12,800 | 424 |
| Standard_DC16es_v6 | 48 | 25,600 | 848 |
| Standard_DC32es_v6 | 64 | 51,200 | 1,696 |
| Standard_DC48es_v6 | 64 | 76,800 | 2,544 |
| Standard_DC64es_v6 | 64 | 102,400 | 3,392 |
| Standard_DC96es_v6 | 64 | 153,600 | 4,000 |
| Standard_DC128es_v6 | 64 | 204,800 | 4,000 |

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

Network interface information for each size.

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_DC2es_v6 | 2 | 7,000 |
| Standard_DC4es_v6 | 2 | 7,000 |
| Standard_DC8es_v6 | 4 | 7,000 |
| Standard_DC16es_v6 | 8 | 7,000 |
| Standard_DC32es_v6 | 8 | 9,000 |
| Standard_DC48es_v6 | 8 | 12,500 |
| Standard_DC64es_v6 | 8 | 25,000 |
| Standard_DC96es_v6 | 8 | 30,000 |
| Standard_DC128es_v6 | 8 | 40,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
- To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

> [!NOTE]
> Some [pricing differences based on your encryption settings](/azure/confidential-computing/confidential-vm-overview#encryption-pricing-differences) exist for confidential VMs.

## Feature support

|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Supported |
|[Live Migration](../../maintenance-and-updates.md)| Coming Soon |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Not Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Not Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Not Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Not Supported |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## Next steps

> [!div class="nextstepaction"]
> [Create a confidential VM in Azure Portal](/azure/confidential-computing/quick-create-confidential-vm-portal)
> [Create a confidential VM in Azure CLI](/azure/confidential-computing/quick-create-confidential-vm-azure-cli)

