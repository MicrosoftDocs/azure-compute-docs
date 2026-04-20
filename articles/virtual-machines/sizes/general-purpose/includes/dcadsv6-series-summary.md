---
title: DCadsv6-series summary include file
description: Include file for DCadsv6-series summary
author: linuxelf001
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 09/01/2025
ms.author: raginjup
ms.reviewer: raginjup
ms.custom: include file
---

Azure's DCadsv6-series virtual machines provide robust [confidential computing](/azure/confidential-computing/confidential-vm-overview) capabilities, ensuring code and data remain secure during processing. Companies can migrate sensitive workloads to cloud infrastructure without modifying existing applications. The series runs on AMD's fourth-generation EPYC™ processors configured for multi-threaded operations with enhanced L3 cache architecture.

AMD's SEV-SNP technology creates hardware-isolated environments that prevent unauthorized access from hypervisors, host management systems, and administrative users. This isolation protects virtual machines from neighboring VMs, hypervisor vulnerabilities, and both hardware and software-based security threats.

The platform includes hardware-based VM memory encryption alongside native support for [confidential disk encryption](/azure/virtual-machines/disk-encryption-overview) feature. Organizations can implement OS disk encryption at boot through customer-managed keys (CMK) or platform-managed keys (PMK). This feature is fully integrated with [Azure KeyVault](/azure/key-vault/general/overview) and [Azure Managed HSM](/azure/key-vault/managed-hsm/overview) ensuring FIPS 140-2 Level 3 compliance.

DCadsv6 instances provide balanced memory-to-vCPU ratios suitable for production environments. Configurations scale up to 96 vCPUs paired with 384 GiB RAM and remote storage connectivity. These specifications support diverse workloads including e-commerce platforms, web services, desktop virtualization, secure databases, enterprise applications and more.
