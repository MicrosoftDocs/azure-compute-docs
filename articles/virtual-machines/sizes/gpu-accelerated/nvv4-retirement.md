---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: NVv4 series retirement
description: 'This article contains the details of the NVv4-series retirement. '
author:      yangnicole-ml # GitHub alias
ms.author: yangnicole
ms.service: azure-virtual-machines
ms.topic: article
ms.date:     06/13/2025
---

# Migrate your NVv4-series virtual machines by September 30, 2026
> [!NOTE]
> There is a known resize operation error that occurs when migrating from the NVv4-series to the NVads_V710_v5-series. Microsoft is working on a fix that will be implemented in April 2026. In the meantime, we suggest that you follow this [workaround](/azure/virtual-machines/azure-vms-no-temp-disk#how-do-i-migrate-from-a-vm-size-with-local-temp-disk-to-a-vm-size-with-no-local-temp-disk---).

> [!NOTE]
> 1-year and 3-year RI purchases for the NVv4-series will end September 30, 2025.  

On September 30, 2026, Microsoft Azure will retire the Standard_NV4as_v4, Standard_NV4ahs_v4, Standard_NV8as_v4, Standard_NV8ahs_v4, Standard_NV16as_v4, Standard_NV16ahs_v4, Standard_NV32as_v4, and Standard_NV32ahs_v4 virtual machines (VMs) in NVv4-series virtual machines (VMs). To avoid any disruption to your service, we recommend that you change the VM sizing for your workloads from the current NVv4-series VMs to the newer VM series in the same NV product line.

Microsoft is recommending the Azure [NVads_V710_v5-series](/azure/virtual-machines/sizes/gpu-accelerated/nvadsv710-v5-series?tabs=sizebasic) VMs, which offer greater GPU memory bandwidth per GPU. The NVads_V710_v5-series VMs take advantage of the AMD Simultaneous Multithreading technology to assign dedicated vCPU threads to each VM and support NVMe for ephemeral local storage capability. 

The NVads_V710_v5-series enables right-sizing for demanding GPU-accelerated graphics applications and cloud-based virtual desktops to provide a seamless end user experience while providing a cost-effective choice for a full range of graphics-enabled virtual desktop experiences. The VMs are also sized to deliver high-quality, interactive gaming experiences in the cloud, optimized for rendering and streaming complex graphics. In addition, the NVads_V710_v5-series supports small AI inference workloads such as Small Language Model (SLMs), recommendation systems, and semantic indexing, by taking advantage of the computational IP blocks in the Radeon Pro V710 GPUs.

|Workload|Recommended SKU to Migrate to|
| -------- | -------- |
|Small AI workloads such as SLM inferencing and semantic search where optimal performance is not a priority or there is an interest in reducing costs.|NVads_V710_v5|
|GPU accelerated graphics applications, virtual desktops, and visualizations.|NVads_V710_v5|

## How does the retirement of the NVv4-series virtual machines affect me? 

__After__ __September 30th, 2026, any remaining__ __NVv4-series virtual machines (VMs) subscriptions will be set to a deallocated state. They'll stop working and no longer incur billing charges. NVv4 will no longer be under SLA or have support included.__

> [!NOTE]
> This retirement only impacts the virtual machine sizes in the NVv4-series powered by AMD Radeon Instinct MI25 GPUs. For NVv3-series virtual machines, please refer to the [NVv3-series virtual machines retirement guide](/azure/virtual-machines/sizes/gpu-accelerated/nvv3-series-retirement). This retirement announcement does not apply to NVadsA10_v5 or NVads_V710_v5 series virtual machines. 

## What action do I need to take before the retirement date? 

You need to resize or deallocate your NVv4-series VMs. We recommend that you change VM sizes for these workloads from the original NVv4-series VMs to the NVads_V710_v5-series VMs (or an alternative).

The [NVads_V710_v5-series](/azure/virtual-machines/sizes/gpu-accelerated/nvadsv710-v5-series?tabs=sizebasic) is powered by AMD Radeon™ Pro V710 GPUs and AMD EPYC™ 9V64 F (Genoa) processors. The VMs feature up to 1 AMD Radeon™ Pro V710 GPU with 24GB memory each, up to 28 multithreaded AMD EPYC 9V64 F processor cores and 160 GiB of system memory. Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information.

## Steps to change VM size

1. Choose a series and size. Refer to the above tables for Microsoft’s recommendation. You can also file a support request if more assistance is needed.

1. [Request quota for the new target VM](/azure/azure-portal/supportability/per-vm-quota-requests).

1. You can [resize the virtual machine](/azure/virtual-machines/resize-vm).

## Help and support

If you have a support plan and you need technical help, create a [support request](https://portal.azure.com/).

1. Under _Issue type_, select __Technical__.

1. Under _Subscription_, select your subscription.

1. Under _Service_, click __My services__.

1. Under _Service type_, select __Virtual Machine running Windows/Linux__.

1. Under _Summary_, enter the summary of your request.

1. Under _Problem type_, select __Assistance with resizing my VM.__

1. Under _Problem subtype_, select the option that applies to you.

