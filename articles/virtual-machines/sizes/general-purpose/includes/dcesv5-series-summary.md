---
title: DCesv6-series summary include file
description: Include file for DCesv6-series summary
author: simranparkhe
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 04/15/2025
ms.author: simranparkhe
ms.reviewer: simranparkhe
ms.custom: include file
# Customer intent: As a cloud architect, I want to explore the capabilities of DCesv6-series virtual machines, so that I can assess their suitability for running confidential workloads and enhancing data security in our organization.
---
The DCesv6-series are [Azure confidential VMs](/azure/confidential-computing/confidential-vm-overview) that protect the confidentiality and integrity of code and data while it's being processed. Organizations can use these VMs to seamlessly bring confidential workloads to the cloud without any code changes to the application. These machines are powered by Intel® 5th Generation Xeon® Scalable processors reaching an all-core turbo clock speed of 3.0 GHz and [Intel® AMX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/advanced-matrix-extensions/overview.html) for AI acceleration. 

With the support of [Intel® Trust Domain Extensions (TDX)](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html), these VMs are hardened from the cloud virtualized environment by denying the hypervisor, other host management code, and administrators access to the VM memory and state. It helps to protect VMs against a broad range of sophisticated [hardware and software attacks](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-trust-domain-extensions.html). 

These VMs have native support for [confidential disk encryption](/azure/virtual-machines/disk-encryption-overview) meaning organizations can encrypt their VM disks at boot with either a customer-managed key (CMK), or platform-managed key (PMK). This feature is fully integrated with [Azure Key Vault](/azure/key-vault/general/overview) or [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) with validation for FIPS 140-2 Level 3. 

The DCesv6-series offers a balance of memory to vCPU performance. With up to 128 vCPUs, 512 GB of RAM, and support for remote disk storage, this size series works well for many general computing workloads, e-commerce systems, web front ends, desktop virtualization solutions, sensitive databases, other enterprise applications.

> [!IMPORTANT]
> These virtual machines are in public preview and not recommended for production usage. Sign up at aka.ms/acc/v6preview for access.
> These VMs are available in West Europe, East US, West US, and West US 3.
