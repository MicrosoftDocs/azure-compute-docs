---
title: Migrate your NP-series virtual machines by May 31, 2027
description: Learn about the retirement of NP-series virtual machines on May 31, 2027, how it affects you, and how to migrate to recommended alternative Azure VM sizes.
author: mattmcinnes
ms.author: mattmcinnes
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 01/06/2026
---

# Migrate your NP-series virtual machines by May 31, 2027
> [!NOTE]
> 1-year and 3-year purchases for the NP-series end April 2, 2026. 

Microsoft Azure has announced the retirement of its NP-series virtual machines. The NP-series, introduced for FPGA-accelerated workloads, is powered by Intel Xeon 8171M (Skylake) CPUs and AMD Xilinx Alveo U250 FPGAs. These VMs are designed for accelerating workloads including machine learning inference, video transcoding, and database search & analytics. 

## How does the retirement of the NP-series virtual machines affect me?

After May 31, 2027, any remaining NP-series virtual machines (VMs) subscriptions will be set to a deallocated state. They’ll stop working and no longer incur billing charges. NP-series will be no longer under SLA or have support included.  

## What action do I need to take before the retirement date?

To avoid service disruption, please resize or deallocate your NP-series VMs and change your VM sizes to a product that meets your workload requirements.    

## Suggested alternative Azure VM sizes 

To ensure continuity and optimal performance, we recommend transitioning to one of the following Azure GPU VM families based on your workload needs: 

- NDv2 VMs – designed for the needs of the most demanding GPU-accelerated AI, machine learning, simulation, and HPC workloads. 

- NCads_H100_v5 VMs – real-world Azure Applied AI training and batch inference workloads.  

- NCasT4_v3 – ideal for deploying AI services, such as real-time inferencing of user-generated requests, or for interactive graphics and visualization workloads.  

Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information. 

### Steps to change VM size

1. Choose a series and size. Review the documentation for the series to understand the capabilities and limitations of the VMs in that series.

1. [Request quota for the new target VM](/azure/azure-portal/supportability/per-vm-quota-requests). 

1. You can [resize the virtual machine](/azure/virtual-machines/resize-vm). 

## Help and support

If you have a support plan and you need technical help, create a [support request](https://ms.portal.azure.com/#home).

1. Under *Issue type*, select **Technical**.

1. Under *Subscription*, select your subscription.

1. Under *Service*, click **My services.**

1. Under *Service type*, select **Virtual Machine running Windows/Linux.**

1. Under *Summary*, enter the summary of your request.

1. Under *Problem type*, select **Assistance with resizing my VM.**

1. Under *Problem subtype*, select the option that applies to you.
