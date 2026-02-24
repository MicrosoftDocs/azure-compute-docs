---
title: Azure ECedsv6-series
description: Specifications for Azure Confidential Computing's ECedsv6-series confidential virtual machines.
author: michamcr
ms.author: simranparkhe
ms.reviewer: mimckitt
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.custom:
  - ignite-2023
ms.topic: concept-article
ms.date: 2/15/2026
# Customer intent: As a cloud architect, I want to evaluate ECedsv6-series confidential virtual machines, so that I can understand their specifications and determine if they meet the security and performance needs of my sensitive workloads in the cloud.
---

# ECedsv6 series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs 

The ECedsv6-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that can be used to protect the confidentiality and integrity of your code and data while it's being processed in the public cloud. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. 

These machines are powered by Intel® 5th Generation Xeon® Scalable processors reaching an all-core turbo clock speed of 3.0 GHz and [Intel® Advanced Matrix Extensions (AMX)](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

Featuring [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code and administrators access to the VM memory and state. It helps to protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

These VMs have native support for [confidential disk encryption](disk-encryption-overview.md) meaning organizations can encrypt their VM disks at boot with either a customer-managed key (CMK), or platform-managed key (PMK). This feature is fully integrated with [Azure KeyVault](/azure/key-vault/general/overview) or [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) with validation for FIPS 140-2 Level 3. 

> [!NOTE]
> There are some [pricing differences based on your encryption settings](/azure/confidential-computing/confidential-vm-overview#encryption-pricing-differences) for confidential VMs.

### ECedsv6-series feature support

*Supported* features in ECedsv6-series VMs:

- [Premium Storage](premium-storage-performance.md)
- [Premium Storage caching](premium-storage-performance.md)
- [VM Generation 2](generation-2.md)

*Unsupported* features in ECedsv6-series VMs:

- [Live Migration](maintenance-and-updates.md)
- [Memory Preserving Updates](maintenance-and-updates.md)
- [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli)
- [Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)

## ECedsv6-series

The ECedsv6 VMs offer even higher memory to vCPU ratio and an all new VM size with up to 64 vCPUs and 512 GiB of RAM. These VMs are ideal for memory intensive applications, large relational database servers, business intelligence applications, and critical applications that process sensitive and regulated data. 

This series supports Standard SSD, Standard HDD, and Premium SSD disk types. Billing for disk storage and VMs is separate. To estimate your costs, use the [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/). 

### ECedsv6-series specifications

| Size | vCPU | RAM (GiB) | Temp storage (SSD) GiB | Max data disks | Max temp disk throughput IOPS/MBps | Max uncached disk throughput IOPS/MBps | Max NICs | Max Network Bandwidth (Mbps) |
|:------:|:----:|:---------:|:------------------------:|:--------------:|:-------------------------------------:|:--------------------------------------:|:--------:|:-------------------------------------:|
| Standard_EC2eds_v6 | 2 | 16 | 110 | 8 | N/A | 3750/106 | 2 | 7000 |
| Standard_EC4eds_v6 | 4 | 32 | 220 | 12 | N/A | 6400/212  | 2 | 7000 |
| Standard_EC8eds_v6 | 8 | 64 | 440 | 24 | N/A | 12800/424 | 4 | 7000 |
| Standard_EC16eds_v6 | 16 | 128 | 880 | 48 | N/A | 25600/848   	|8  	|7000  	|
| Standard_EC32eds_v6  	|32  	|256  	|1760  	|64  	| N/A  	|51200/1696  	|8  	|9000  	|
| Standard_EC48eds_v6  	|48  	|384  	|2640 	|64  	| N/A  	|76800/2544  	|8  	|12500  	|
| Standard_EC64eds_v6  	|64  	|512  	|3520  	|64  	| N/A  	|80000/3392 	|8  	|30000    |

[!INCLUDE [virtual-machines-common-sizes-table-defs](./includes/virtual-machines-common-sizes-table-defs.md)]

## Next steps

> [!div class="nextstepaction"]
> [Create a confidential VM in Azure Portal](/azure/confidential-computing/quick-create-confidential-vm-portal)
> [Create a confidential VM in Azure CLI](/azure/confidential-computing/quick-create-confidential-vm-azure-cli)
