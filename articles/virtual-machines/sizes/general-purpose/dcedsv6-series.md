---
title: DCedsv6-series
description: Information on and specifications of the DCedsv6-series sizes
author: simranparkhe
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 02/17/2026
ms.author: simranparkhe
ms.reviewer: simranparkhe
# Customer intent: As a cloud architect, I want to review the specifications and features of the DCesv6-series virtual machine sizes, so that I can select the appropriate size for my applications based on performance and resource needs.
---

# DCedsv6-series sizes

The DCedsv6-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that protect the confidentiality and integrity of code and data while it's being processed. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. These machines are powered by Intel® 5th Generation Xeon® Scalable processors reaching an all-core turbo clock speed of 3.0 GHz and [Intel® AMX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

Featuring [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code, and administrators access to the VM memory and state. It helps to protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

These VMs have native support for [confidential disk encryption](/azure/virtual-machines/disk-encryption-overview) meaning organizations can encrypt their VM disks at boot with either a customer-managed key (CMK), or platform-managed key (PMK). This feature is fully integrated with [Azure Key Vault](/azure/key-vault/general/overview) or [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) with validation for FIPS 140-2 Level 3. 

The DCedsv6 offer a balance of memory to vCPU performance that's suitable most production workloads. With up to 128 vCPUs, 512 GiB of RAM, and support for up to 7 TB of local disk storage. These VMs work well for many general computing workloads, e-commerce systems, web front ends, desktop virtualization solutions, sensitive databases, other enterprise applications and more.

## Host specifications
| Part | Quantity min - max | Specs <br><sup>SKU ID, Performance Units, etc.  |
|---|---|---|
| Processor      | 2 - 128 vCPUs       | Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64]                               |
| Memory         | 8 - 512 GB          |                                  |
| Local Storage  | 110 - 7040 GiB           |                                |
| Remote Storage | 8 - 64 Disks    | 3750 to 204800 IOPS <br> and 80 to 4000 MBps   |
| Network        | 2 - 8 NICs          | 12500 to 40000 Mbps                          |
| Accelerators   | None              |                                   |


## Feature support

- [Premium Storage](../../premium-storage-performance.md): Supported <br>
- [Premium Storage caching](../../premium-storage-performance.md): Supported <br>
- [Live Migration](../../maintenance-and-updates.md): Not Supported <br>
- [Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>
- [Generation 2 VMs](../../generation-2.md): Supported <br>
- [Generation 1 VMs](../../generation-2.md): Not Supported <br>
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Not Supported <br>
- [Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

This table shows the number of vCPUs and amount of memory for each DCesv6-series size.

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_DC2eds_v6 | 2 | 8 |
| Standard_DC4eds_v6 | 4 | 16 |
| Standard_DC8eds_v6 | 8 | 32 |
| Standard_DC16eds_v6 | 16 | 64 |
| Standard_DC32eds_v6 | 32 | 128 |
| Standard_DC48eds_v6 | 48 | 192 |
| Standard_DC64eds_v6 | 64 | 256 |
| Standard_DC96eds_v6 | 96 | 384 |
| Standard_DC128eds_v6 | 128 | 512 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage available for each size.

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR) IOPS | Temp Disk Random Read (RR) Throughput (MB/s)|
| --- | --- | --- | --- | --- |
| Standard_DC2eds_v6 | 1 | 110 | 37500 | 180 |
| Standard_DC4eds_v6 | 1 | 220 | 75000 | 360 |
| Standard_DC8eds_v6 | 1 | 440 | 150000 | 720 |
| Standard_DC16eds_v6 | 2 | 440 | 300000 | 1440 |
| Standard_DC32eds_v6 | 4 | 440 | 600000 | 2880 |
| Standard_DC48eds_v6 | 6 | 440 | 900000 | 4320 |
| Standard_DC64eds_v6 | 4 | 880 | 1200000 | 5760 |
| Standard_DC96eds_v6 | 6 | 880 | 1800000 | 8640 |
| Standard_DC128eds_v6 | 4 | 1760 | 2400000 | 11520 |

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage available for each size.

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MB/s) 
| --- | --- | --- | --- |
| Standard_DC2eds_v6 | 8 | 3750 | 106 |
| Standard_DC4eds_v6 | 12 | 6400 | 212 |
| Standard_DC8eds_v6 | 24 | 12800 | 424 |
| Standard_DC16eds_v6 | 48 | 25600 | 848 |
| Standard_DC32eds_v6 | 64 | 51200 | 1696 |
| Standard_DC48eds_v6 | 64 | 76800 | 2544 |
| Standard_DC64eds_v6 | 64 | 102400 | 3392 |
| Standard_DC96eds_v6 | 64 | 153600 | 4000 |
| Standard_DC128eds_v6 | 64 | 204800 | 4000 |

#### Storage resources
- [Introduction to Azure Managed Disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure Managed Disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure Managed Disk](../../../virtual-machines/disks-shared.md)

#### Table definitions

- Total local temporary storage is calculated by multiplying the max number of storage disks with the temp disk size. For example, for the Standard_DC128eds_v6, the total local temporary storage capacity is `4 x 1760 GiB = 7040 GiB`
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB can appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface information for each size.

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_DC2eds_v6 | 2 | 7000 |
| Standard_DC4eds_v6 | 2 | 7000 |
| Standard_DC8eds_v6 | 4 | 7000 |
| Standard_DC16eds_v6 | 8 | 7000 |
| Standard_DC32eds_v6 | 8 | 9000 |
| Standard_DC48eds_v6 | 8 | 12500 |
| Standard_DC64eds_v6 | 8 | 25000 |
| Standard_DC96eds_v6 | 8 | 30000 |
| Standard_DC128eds_v6 | 8 | 40000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance depends on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]