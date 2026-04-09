---
title: Migrate your HC-series virtual machines by May 31, 2027
description: Learn about the retirement of HC-series virtual machines on May 31, 2027, how it affects you, and how to migrate to recommended alternative Azure VM sizes.
author: mattmcinnes
ms.author: mattmcinnes
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 01/06/2026
---

# Migrate your HC-series virtual machines by May 31, 2027
> [!NOTE]
> 1-year and 3-year purchases for the HC-series end April 2, 2026.

Microsoft Azure has announced the retirement of its HC-series virtual machines effective May 31, 2027. Introduced in 2019, the HC-series is equipped with Intel Xeon Platinum 8,168 processor cores, 8 GB of RAM per CPU core, and a 100 Gb/sec Mellanox EDR InfiniBand. Since launching this series, Microsoft introduced HBv5, HBv4, HX, and HBv3 series virtual machines. These newer offerings pack the latest technological improvements in compute, memory, and backend networking. This enables superior performance, scaling efficiency, and cost-effectiveness for a wide range of high-performance computing workloads.

## How does the retirement of the HC-series virtual machines affect me?

After May 31, 2027, any remaining HC-series virtual machines (VMs) subscriptions will be set to a deallocated state. They'll stop working and no longer incur billing charges. HC-series will no longer be under SLA or have support included.

## What action do I need to take before the retirement date?

To avoid service disruption, deallocate your HC-series VMs and migrate these workloads to newer HPC-optimized SKUs such as HBv5, HBv4, HX, or HBv3 series (or an alternative that meets your workload requirements).

## Suggested alternative Azure VM sizes

To ensure continuity and optimal performance, we recommend transitioning from the current HC-series VMs to newer VM series within Azure's HPC portfolio.

| Workload | Recommended VM to migrate to |
|---|---|
| Memory-bandwidth intensive HPC applications (for example, CFD, structural mechanics, energy research, weather, and climate, chemistry & materials) | HBv5, HBv4, HBv3 |
| Memory capacity/latency bound (for example, silicon design/EDA) | HX, HBv5, HBv3 |
| Compute bound (for example, financial calculations, rendering) | HX, HBv4, HBv3 |
| Intel-specific requirements (for example, Intel MKL, ISV certification on Xeon) | Ddsv6/Edsv6, Ddsv5/Edsv5, Ddsv4/Edsv4 |

> [!NOTE]
> These virtual machine recommendations are listed in order of cost performance benefit to customers from most to least.

Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information.

### Steps to change VM size

1. Choose a series and size. Review the table above for Microsoft's recommendation. You can also file a support request if more assistance is needed.

1. [Request quota for the new target VM](/azure/azure-portal/supportability/per-vm-quota-requests).

1. [Resize the virtual machine](/azure/virtual-machines/resize-vm).

## Help and support

If you have a support plan and you need technical help, create a [support request](https://ms.portal.azure.com/#home).

1. Under *Issue type*, select **Technical**.

1. Under *Subscription*, select your subscription.

1. Under *Service*, select **My services.**

1. Under *Service type*, select **Virtual Machine running Windows/Linux.**

1. Under *Summary*, enter the summary of your request.

1. Under *Problem type*, select **Assistance with resizing my VM.**

1. Under *Problem subtype*, select the option that applies to you.