---
title: NCv3 and NC6s NC12s NC24s Retirement
description: Migration guide for sizes NC6s_v3 NC12s_v3 NC24s_v3
author: sherrywangms
ms.author: sherrywang
ms.service: azure-virtual-machines
ms.topic: conceptual
ms.date: 03/19/2024
ms.subservice: sizes
---
# Migrate your NC6s_v3, NC12s_v3, NC24s_v3 virtual machine sizes by September 30, 2025

On September 30, 2025, Microsoft Azure will retire the Standard_NC6s_v3, Standard_NC12s_v3 and Standard_NC24s_v3 virtual machines (VMs) in NCv3-series virtual machines (VMs). To avoid any disruption to your service, we recommend that you change the VM sizing for your workloads from the current NCv3-series VMs to the newer VM series in the same NC product line.

Microsoft is recommending the Azure [NCads H100 v5-series ](/azure/virtual-machines/ncads-h100-v5?source=recommendations)VMs, which offer greater GPU memory bandwidth per GPU, improved [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli) capabilities, and larger and faster local solid state drives. These VMs are targeted for GPU accelerated midrange AI training, batch inferencing, and high-performance computing simulation workloads.

Other VMs that may be migrated to from Standard_NC6s_v3, Standard_NC12s_v3, and Standard_NC24s_v3 include NVadsA10_v5, NCasT4_v3, and NVadsV710_v5.

## How does the retirement of the NC6s_v3, NC12s_v3, NC24s_v3 virtual machine sizes in NCv3-series affect me? 

**After** **September 30th, any remaining** **Standard_NC6s_v3, Standard_NC12s_v3 and Standard_NC24s_v3 virtual machines (VMs) subscriptions will be set to a deallocated state. They'll stop working and no longer incur billing charges. The NCv3 will no longer be under SLA or have support included.** 

> [!Note]
> This retirement only impacts the virtual machine sizes in the original NCv3-series powered by NVIDIA V100 GPUs. [See Standard_NC24rs_v3 retirement guide](https://aka.ms/nc24rsv3migrationguide). This retirement announcement doesn't apply to NCasT4 v3, and NC A100 v4 and NCads H100 v5 series virtual machines.

## What action do I need to take before the retirement date? 

You need to resize or deallocate your Standard_NC6s_v3, Standard_NC12s_v3 and Standard_NC24s_v3 VMs. We recommend that you change VM sizes for these workloads, from the original Standard_NC6s_v3, Standard_NC12s_v3 and Standard_NC24s_v3 VMs to the NCads H100 v5-series (or an alternative).

The [NCads H100 v5-series](/azure/virtual-machines/ncads-h100-v5?source=recommendations) is powered by NVIDIA H100 NVL GPU and 4th-generation AMD EPYC™ Genoa processors. The VMs feature up to 2 NVIDIA H100 NVL GPUs with 94GB memory each, up to 96 non-multithreaded AMD EPYC Genoa processor cores and 640 GiB of system memory. Check [Azure Regions by Product page](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/) for region availability. Visit the [Azure Virtual Machine pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/) for pricing information.

| Current VM Size| Target VM Size | Difference in Specification |
|---|---|---|
| Standard_NC6s_v3 |Standard_NC40ads_H100_v5|vCPU: 40 (+34) <br>GPU Count: 1 (Same)<br>Memory: GiB 320 (+208)<br>Temp storage GiB: 3576 (+2840)<br>Max data disks: 8 (+0)<br>Accelerated networking: Yes<br>Premium storage: Yes |
| Standard_NC12s_v3 |Standard_NC40ads_H100_v5|vCPU: 40 (+28) <br>GPU Count: 1 (+0)<br>Memory: GiB 320 (+96)<br>Temp storage GiB: 3576 (+2102) <br>Max data disks: 8 (+0)<br>Accelerated networking: Yes (+)<br>Premium storage: Yes |
| Standard_NC24s_v3 | Standard_NC80adis_H100_v5 |vCPU: 80 (+56) <br>GPU Count: 2 (+0)<br>Memory: GiB 640 (+192)<br>Temp storage (SSD) GiB: 7152 (+4204)<br>Max data disks: 16 (+0)<br>Accelerated networking: Yes (+)<br>Premium storage: Yes |

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

