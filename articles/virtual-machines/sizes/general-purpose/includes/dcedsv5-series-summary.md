---
title: DCedsv5-series summary include file
description: Include file for DCedsv5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to assess the capabilities of DCedsv5-series confidential VMs, so that I can implement secure and efficient solutions for processing sensitive workloads in the cloud."
---
The DCedsv5-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that protect the confidentiality and integrity of code and data while it's being processed. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. These machines are powered by Intel® 4th Generation Xeon® Scalable processors with Base Frequency of 2.1 GHz, All Core Turbo Frequency of 2.9 GHz and [Intel® AMX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

Featuring [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code and administrators access to the VM memory and state. It helps to protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

These VMs have native support for [confidential disk encryption](/azure/virtual-machines/disk-encryption-overview) meaning organizations can encrypt their VM disks at boot with either a customer-managed key (CMK), or platform-managed key (PMK). This feature is fully integrated with [Azure KeyVault](/azure/key-vault/general/overview) or [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) with validation for FIPS 140-2 Level 3. 

The DCedsv5 offer a balance of memory to vCPU performance that is suitable most production workloads. With up to 96 vCPUs, 384 GiB of RAM, and support for up to 2.8 TB of local disk storage. These VMs work well for many general computing workloads, e-commerce systems, web front ends, desktop virtualization solutions, sensitive databases, other enterprise applications and more.

> [!IMPORTANT]
> These virtual machines are in public preview and not recommended for production usage.
> These VMs are available in West Europe, Central US, East US 2 and North Europe.
