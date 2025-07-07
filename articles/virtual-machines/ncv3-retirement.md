---
title: NCv3-series Retirement
description: Migration guide for NCv3-series virtual machines
author: sherrywangms
ms.author: sherrywang
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 05/28/2025
ms.subservice: sizes
# Customer intent: As a cloud infrastructure administrator, I want to migrate my NCv3-series virtual machines to a supported VM size by the retirement date, so that I can maintain my workloads without service disruption and ensure ongoing performance enhancements.
---
# Migrate your NCv3-series virtual machines by September 30, 2025

On September 30, 2025, Microsoft Azure will retire the Standard_NC6s_v3, Standard_NC12s_v3, Standard_NC24s_v3, and Standard_NC24rs_v3 virtual machines (VMs) in NCv3-series virtual machines (VMs). To avoid any disruption to your service, we recommend that you change the VM sizing for your workloads from the current NCv3-series VMs to the newer VM series in the same NC product line.

Microsoft is recommending the Azure [NCadsH100_v5-series ](/azure/virtual-machines/ncads-h100-v5?source=recommendations)VMs, which offer greater GPU memory bandwidth per GPU, improved [accelerated networking](/azure/virtual-network/create-vm-accelerated-networking-cli) capabilities, and larger and faster local solid state drives. These VMs are targeted for GPU accelerated midrange AI training, batch inferencing, and high-performance computing simulation workloads.

Depending on the workload being run, regional affinity, and cost preferences, other VMs that may be migrated to from the NCv3-series VMs include [NVadsA10_v5](/azure/virtual-machines/sizes/gpu-accelerated/nvadsa10v5-series?tabs=sizebasic), [NCasT4_v3](/azure/virtual-machines/sizes/gpu-accelerated/ncast4v3-series?tabs=sizebasic), and [NVadsV710_v5](/azure/virtual-machines/sizes/gpu-accelerated/nvadsv710-v5-series?tabs=sizebasic): 

|Workload|Recommended SKU to Migrate to|
| -------- | -------- |
|High GPU compute workloads such as real-time inferencing and LLM inferencing.|NCadsH100_v5 which has the best inference per dollar value.|
|Offline inferencing workloads where latency is not the primary concern, and there is an interest in purchasing smaller VM SKUs or reducing costs.|NCasT4_v3|
|Graphics, visualization, or small AI workloads where optimal performance is not a priority. |NVadsA10_v5, NVadsV710_v5|

## How does the retirement of the NCv3-series virtual machines affect me?

**After** **September 30th, any remaining** **NCv3-series virtual machines (VMs) subscriptions will be set to a deallocated state. They'll stop working and no longer incur billing charges. NCv3 will no longer be under SLA or have support included.** 

> [!Note]
> This retirement only impacts the virtual machine sizes in the original NCv3-series powered by NVIDIA V100 GPUs. This retirement announcement doesn't apply to NCasT4_v3, NC_A100_v4, and NCadsH100_v5 series virtual machines.

## What action do I need to take before the retirement date? 

You need to resize or deallocate your NCv3-series VMs. We recommend that you change VM sizes for these workloads from the original NCv3-series VMs to the NCadsH100_v5-series VMs (or an alternative).

The [NCadsH100_v5-series](/azure/virtual-machines/ncads-h100-v5?source=recommendations) is powered by NVIDIA H100 NVL GPU and 4th-generation AMD EPYC™ Genoa processors. The VMs feature up to 2 NVIDIA H100 NVL GPUs with 94GB memory each, up to 96 non-multithreaded AMD EPYC Genoa processor cores and 640 GiB of system memory. Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information.

## Steps to change VM size 

1. Choose a series and size. Refer to the above tables for Microsoft’s recommendation. You can also file a support request if more assistance is needed.
2. [Request quota for the new target VM](/azure/azure-portal/supportability/per-vm-quota-requests).
3. You can [resize the virtual machine](resize-vm.md). 

   

## Help and support

If you have a support plan and you need technical help, create a [support request](https://portal.azure.com/). 

1. Under _Issue type_, select **Technical**. 
2. Under _Subscription_, select your subscription. 
3. Under _Service_, click **My services**.  
4. Under _Service type_, select **Virtual Machine running Windows/Linux**.
5. Under _Summary_, enter the summary of your request.
6. Under _Problem type_, select **Assistance with resizing my VM.**
1. Under _Problem subtype_, select the option that applies to you.

