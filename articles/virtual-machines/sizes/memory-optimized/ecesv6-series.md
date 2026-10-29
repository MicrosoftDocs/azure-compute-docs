---
title: ECesv6-series
description: Specifications for Azure Confidential Computing's ECesv6-series confidential virtual machines.
author: michamcr
ms.author: mmcrey
ms.reviewer: mimckitt
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.custom:
  - ignite-2023
ms.topic: concept-article
ms.date: 4/15/2025
# Customer intent: As a cloud architect, I want to evaluate ECesv6-series confidential virtual machines, so that I can understand their specifications and determine if they meet the security and performance needs of my sensitive workloads in the cloud.
---

# ECesv6 series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs 

> [!NOTE]
> Some [pricing differences based on your encryption settings](/azure/confidential-computing/confidential-vm-overview#encryption-pricing-differences) exist for confidential VMs.

The ECesv6-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that can be used to protect the confidentiality and integrity of your code and data while it's being processed in the public cloud. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. 

These machines are powered by Intel® 5th Generation Xeon® Scalable processors reaching an all-core turbo clock speed of 3.0 GHz and [Intel® Advanced Matrix Extensions (AMX)](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

By using [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code, and administrators access to the VM memory and state. It helps protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

These VMs have native support for [confidential disk encryption](../../disk-encryption-overview.md) meaning organizations can encrypt their VM disks at boot with either a customer-managed key (CMK) or platform-managed key (PMK). This feature is fully integrated with [Azure KeyVault](/azure/key-vault/general/overview) or [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) with validation for FIPS 140-2 Level 3. 

The ECesv6 VMs offer an even higher memory-to-vCPU ratio and a new VM size with up to 64 vCPUs and 512 GiB of RAM. These VMs are ideal for memory-intensive applications, large relational database servers, business intelligence applications, and critical applications that process sensitive and regulated data. 


## Host specifications

| Part | Quantity min - max | Specs <br><sup>SKU ID, Performance Units, etc.</sup> |
| --- | --- | --- |
| Processor | 2 - 64 vCPUs | Intel Xeon Platinum 8573C (Emerald Rapids) [x86-64] |
| Memory | 16 - 512 GiB | |
| Local Storage | None | |
| Remote Storage | 8 - 64 Disks | 3,750 - 102,400 IOPS <br>106 - 3,392 MBps |
| Network | 2 - 8 NICs | 7,000 - 25,000 Mbps |
| Accelerators | None | |

## Sizes in series

### [Basics](#tab/sizebasic)

This table shows the number of vCPUs and amount of memory for each ECesv6-series size.

| Size Name | vCPUs | Memory (GiB) |
| --- | --- | --- |
| Standard_EC2es_v6 | 2 | 16 |
| Standard_EC4es_v6 | 4 | 32 |
| Standard_EC8es_v6 | 8 | 64 |
| Standard_EC16es_v6 | 16 | 128 |
| Standard_EC32es_v6 | 32 | 256 |
| Standard_EC48es_v6 | 48 | 384 |
| Standard_EC64es_v6 | 64 | 512 |

#### VM Basics resources

- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage available for each size.

> [!NOTE]
> This series doesn't have local storage.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage available for each size.

| Size Name | Max Remote Storage Disks | Uncached Premium SSD IOPS | Uncached Premium SSD Throughput (MBps) |
| --- | --- | --- | --- |
| Standard_EC2es_v6 | 8 | 3,750 | 106 |
| Standard_EC4es_v6 | 12 | 6,400 | 212 |
| Standard_EC8es_v6 | 24 | 12,800 | 424 |
| Standard_EC16es_v6 | 48 | 25,600 | 848 |
| Standard_EC32es_v6 | 64 | 51,200 | 1,696 |
| Standard_EC48es_v6 | 64 | 76,800 | 2,544 |
| Standard_EC64es_v6 | 64 | 102,400 | 3,392 |

### [Network](#tab/sizenetwork)

Network interface information for each size.

| Size Name | Max NICs | Max Network Bandwidth (Mbps) |
| --- | --- | --- |
| Standard_EC2es_v6 | 2 | 7,000 |
| Standard_EC4es_v6 | 2 | 7,000 |
| Standard_EC8es_v6 | 4 | 7,000 |
| Standard_EC16es_v6 | 8 | 7,000 |
| Standard_EC32es_v6 | 8 | 9,000 |
| Standard_EC48es_v6 | 8 | 12,500 |
| Standard_EC64es_v6 | 8 | 25,000 |

---

## Feature support

|Feature name | Support status |
| --- | --- |
|[Premium Storage](../../premium-storage-performance.md)| Supported |
|[Premium Storage caching](../../premium-storage-performance.md)| Supported |
|[Live Migration](../../maintenance-and-updates.md)| Not supported |
|[Memory Preserving Updates](../../maintenance-and-updates.md)| Not Supported |
|[Generation 2 VMs](../../generation-2.md)| Supported |
|[Generation 1 VMs](../../generation-2.md)| Not Supported |
|[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)| Not Supported |
|[Ephemeral OS Disk](../../ephemeral-os-disks.md)| Not Supported |
|[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)| Not Supported |### ECesv6-series feature support

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## Next steps

> [!div class="nextstepaction"]
> [Create a confidential VM in Azure Portal](/azure/confidential-computing/quick-create-confidential-vm-portal)
> [Create a confidential VM in Azure CLI](/azure/confidential-computing/quick-create-confidential-vm-azure-cli)
