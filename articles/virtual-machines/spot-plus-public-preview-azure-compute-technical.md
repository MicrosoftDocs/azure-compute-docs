---
title: "Azure Spot Plus VMs: Public Preview Overview"
description: Explore Azure Spot Plus, a new Spot VM tier offering better runtimes, earlier eviction notices, and billing protection. Learn how to get started today.
ms.date: 11/05/2026
ms.topic: concept-article
ms.reviewer: mattmcinnes
ai-usage: ai-assisted
---

# Azure Spot Plus VMs: public preview

Azure Compute Spot Plus is now in public preview. This enhanced tier of Azure Spot VMs brings greater runtime consistency and earlier eviction signals to interruption-tolerant workloads, giving them more time to prepare and adapt as capacity conditions change.

Standard Spot remains a great fit for highly resilient workloads focused on maximizing savings. Spot Plus delivers an enhanced experience for workloads that need more predictability and time to respond to interruptions.

## Key Spot Plus capabilities

### Improved runtime

Spot Plus delivers better runtime than standard Spot VMs, while remaining evictable.

### Billing protection for early eviction

If Azure evicts a Spot Plus VM to reclaim capacity within the first 60 minutes, the VM's usage isn't charged.

### Earlier eviction notice

Spot Plus provides up to 2 minutes of eviction notice, giving workloads time to checkpoint and drain.

### Rebalance notifications

Spot Plus provides early signals, typically 30 seconds to 15 minutes ahead, when a VM is at elevated risk of eviction.

### Transparent pricing and cost visibility

Spot Plus is priced at a premium over standard Spot but remains well below pay-as-you-go. See pricing for [Linux Virtual Machines](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) and [Windows Virtual Machines](https://azure.microsoft.com/pricing/details/virtual-machines/windows/).

### Familiar deployment surfaces

Deploy Spot Plus through the Compute API, ARM templates, the Azure portal, Azure CLI, Azure PowerShell, and the Azure SDKs on single-instance VMs and Virtual Machine Scale Sets (Flexible and Uniform orchestration).

## Spot vs. Spot Plus differences

| Capability             | Spot                                                       | Spot Plus                                                                           |
|------------------------|------------------------------------------------------------|-------------------------------------------------------------------------------------|
| Best for               | Highly resilient workloads where lowest price matters most | Interruption-tolerant workloads that need more runtime consistency and more warning |
| Runtime                | No minimum runtime target                                  | Improved runtime targeting at least 60 minutes (Best Effort)                        |
| Billing protection     | None                                                       | No charge if Azure reclaims capacity within 60 minutes                              |
| Price                  | Up to 90% off pay-as-you-go                                | Premium over Spot, still below pay-as-you-go                                        |
| Eviction notice        | Up to 30 seconds                                           | 30 seconds to 2 minutes                                                             |
| Rebalance notification | Up to 15 minutes                                           | 30 seconds – 15 minutes                                                             |

## Get started

1. Register the subscription for:

   - Compute API AFEC that supports Spot Plus priority (`Microsoft.Compute/SpotPlus`), using [Features - Register - REST API (Azure Resource Management)](/rest/api/resources/features/register?tabs=HTTP).
   - `Microsoft.Compute` and `Microsoft.Resources` providers, using [Providers - Register - REST API (Azure Resource Management)](/rest/api/resources/providers/register?tryIt=true&source=docs#code-try-0).

1. Deploy a VM or scale set with `SpotPlus` priority. The API version must be 2026-04-01 or later.

   - [Create a VM with SpotPlus priority](/rest/api/compute/virtual-machines/create-or-update?view=rest-compute-2026-04-01&preserve-view=true&tabs=HTTP#create-a-vm-with-spotplus-priority).
   - [Virtual machine priority types](/rest/api/compute/virtual-machines/create-or-update?view=rest-compute-2026-04-01&preserve-view=true&tabs=HTTP#virtualmachineprioritytypes).

1. Listen to rebalance signals via Scheduled Events for [Linux VMs](linux/scheduled-events.md) or [Windows VMs](windows/scheduled-events.md).
