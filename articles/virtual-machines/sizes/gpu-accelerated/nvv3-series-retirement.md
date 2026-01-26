---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: NVv3 series retirement
description: 'This article contains the details of the NVv3-series retirement. '
author:      yangnicole-ml # GitHub alias
ms.author: yangnicole
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date:     06/13/2025
---

# Migrate your NVv3-series virtual machines by September 30, 2026
> [!NOTE]
> 1-year and 3-year RI purchases for the NVv3-series ended November 1, 2025.  

On September 30, 2026, Microsoft Azure will retire the Standard_NV12s_v3, Standard_NV12hs_v3, Standard_NV24s_v3, Standard_NV24ms_v3, Standard_NV32ms_v3, and Standard_NV48s_v3 virtual machines (VMs) in the NVv3-series virtual machines (VMs). To avoid any disruptions to your service, we recommend that you change the VM sizing for your workloads from the current NVv3-series VMs to the newer VM series in the same NV product line. 

Microsoft is recommending the Azure [NVadsA10_v5-series](/azure/virtual-machines/sizes/gpu-accelerated/nvadsa10v5-series?tabs=sizebasic) VMs, which offer greater GPU memory bandwidth per GPU. With the NVadsA10_v5-series VMs, Azure introduces VMs with partial NVIDIA GPUs and each VM instance comes with a GRID license. This license gives you the flexibility to use an NV instance as a virtual workstation for a single user or 25 concurrent users can connect to the VM for a virtual application scenario. These VMs are targeted for GPU accelerated graphics applications, virtual desktops, visualizations, or small AI workloads. 

Depending on the workload being run, regional affinity, and cost preferences, other VMs that may be migrated to from the NVv3-series VMs include [NCasT4_v3](/azure/virtual-machines/sizes/gpu-accelerated/ncast4v3-series?tabs=sizebasic) and [NVadsV710_v5](/azure/virtual-machines/sizes/gpu-accelerated/nvadsv710-v5-series?tabs=sizebasic): 

|Workload|Recommended SKU to Migrate to|
| -------- | -------- |
|GPU accelerated graphics applications, virtual desktops, visualizations, or small AI workloads. |NVadsA10_v5|
|Offline inferencing workloads where latency is not a primary concern, and there is an interest in purchasing smaller VM SKUs or reducing costs. |NCasT4_v3|
|Graphics, visualization, or small AI workloads such as SLM inferencing and semantic search where optimal performance is not a priority or there is an interest in reducing costs.|NVadsV710_v5|

## How does the retirement of the NVv3-series virtual machines affect me? 

**After September 30th, 2026, any remaining NVv3-series virtual machines (VMs) subscriptions will be set to a deallocated state. They'll stop working and no longer incur billing charges. NVv3 will no longer be under SLA or have support included.** 

> [!NOTE]
> This retirement only impacts the virtual machine sizes in the NVv3-series powered by NVIDIA Tesla M60 GPUs. For NVv4-series virtual machines, please refer to the [NVv4-series virtual machines retirement guide](/azure/virtual-machines/sizes/gpu-accelerated/nvv4-retirement). This retirement announcement does not apply to NVadsA10_v5 or NVadsV710_v5 series virtual machines. 

## What action do I need to take before the retirement date? 

You need to resize or deallocate your NVv3-series VMs. We recommend that you change VM sizes for these workloads from the original NVv3-series VMs to the NVadsA10_v5-series VMs (or an alternative).

The [NVadsA10_v5-series](/azure/virtual-machines/sizes/gpu-accelerated/nvadsa10v5-series?tabs=sizebasic) is powered by NVIDIA A10 GPUs and AMD EPYC™ 74F3V(Milan) processors. The VMs feature up to 2 NVIDIA A10 GPUs with 24GB memory each, up to 72 non-multithreaded AMD EPYC 74F3V processor cores and 880 GiB of system memory. Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information.

## Steps to change VM size

1. Choose a series and size. Refer to the above tables for Microsoft’s recommendation. You can also file a support request if more assistance is needed.

1. [Request quota for the new target VM](/azure/azure-portal/supportability/per-vm-quota-requests).

1. You can [resize the virtual machine](/azure/virtual-machines/resize-vm).

## Help and support

If you have a support plan and you need technical help, create a [support request](https://portal.azure.com/).

1. Under *Issue type*, select **Technical**.

1. Under *Subscription*, select your subscription.

1. Under *Service*, click **My services**.

1. Under *Service type*, select **Virtual Machine running Windows/Linux**.

1. Under *Summary*, enter the summary of your request.

1. Under *Problem type*, select **Assistance with resizing my VM.**

1. Under *Problem subtype*, select the option that applies to you.

