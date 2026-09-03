---
title: DCsv3-series and DCdsv3-series retirement
description: Retirement information for the DCsv3 and DCdsv3 series virtual machine sizes. Before retirement, migrate your workloads to recommended options.
author: milicaspuzic
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: how-to
ms.date: 09/15/2026
ms.author: milicaspuzic
ms.custom: references_regions
ai-usage: ai-assisted
---

# DCsv3 and Dcdsv3-series retirement

This migration guide is designed for users of DCsv3 and Dcdsv3-series virtual machines (VMs), which are scheduled for retirement on **October 31, 2029**. To ensure minimal disruption and to continue optimizing cost and performance, this guide helps you transition to the latest series of confidential VMs.

This document covers:
- Recommended options for migration
- Detailed migration steps
- Frequently asked questions

By migrating to newer VM series, you gain access to improved price-performance ratios, broader regional availability, and the latest hardware capabilities.

## Recommended options for migration

**Before October 31, 2029**, migrate your workloads to one of the following options that best aligns with your business needs:

- To use a VM-based programming model on AMD's fourth-generation EPYC™ processors, consider [DCasv6](../general-purpose/dcasv6-series.md)/[DCadsv6](../general-purpose/dcadsv6-series.md) or memory optimized [ECasv6](../memory-optimized/ecasv6-series.md)/[ECadsv6](../memory-optimized/ecadsv6-series.md) series.
- To use a VM-based programming model on  Intel® 5th Generation Xeon® Scalable processors, consider [DCesv6](../general-purpose/dcesv6-series.md)/[DCedsv6](../general-purpose/dcedsv6-series.md) or memory optimized [ECesv6](../../../virtual-machines/ecesv6-series.md)/[ECedsv6](../../../virtual-machines/ecedsv6-series.md) confidential VMs (CVMs).
- For containerized applications, consider using [Azure Confidential Container Instances (C-ACI)](../../../container-instances/container-instances-confidential-overview.md) or [Virtual nodes on Azure Container Instances (C-VN2) for Azure Kubernetes Service (AKS)](../../../container-instances/container-instances-virtual-nodes.md).

Additionally, there might be changes to your Azure Virtual Machines billing because of this retirement. Refer to the Azure Virtual Machines [pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/linux-previous/) for more information.

## Migration steps
Start planning your migration from DCsv3/DCdsv3-series today.

### Identify the target migration option
- Learn more about the different migration options and their benefits in the section [*Recommended options for migration*](#recommended-options-for-migration).
- Evaluate your current VM's workload and performance requirements and identify the target migration option.

### Check and request quota increases
- Before resizing and migration, verify that your subscription has sufficient quota for the target VM series.
- Request more quota through the [Azure portal](/azure/azure-portal/supportability/per-vm-quota-requests) if needed.

### Complete migration
- Complete migration as soon as possible to prevent business impact and to take advantage of the improved performance, and extensive regional coverage of the new confidential computing offerings.
- Follow the documentation for your chosen migration option.
- For technical questions and issues, get help from community experts in [Microsoft Q&A](/search/?terms=confidential%20computing&category=QnA).

## Migrate to new generation offerings 
- To use a VM-based programming model on AMD's fourth-generation EPYC™ processors, consider [DCasv6](../general-purpose/dcasv6-series.md)/[DCadsv6](../general-purpose/dcadsv6-series.md) or memory optimized [ECasv6](../memory-optimized/ecasv6-series.md)/[ECadsv6](../memory-optimized/ecadsv6-series.md) series.
- To use a VM-based programming model on  Intel® 5th Generation Xeon® Scalable processors, consider [DCesv6](../general-purpose/dcesv6-series.md)/[DCedsv6](../general-purpose/dcedsv6-series.md) or memory optimized [ECesv6](../../../virtual-machines/ecesv6-series.md)/[ECedsv6](../../../virtual-machines/ecedsv6-series.md) confidential VMs (CVMs).
- If you already have or plan to transition to containerized workloads and want to lift and shift your containerized applications, consider using [Azure Confidential Container Instances (C-ACI)](../../../container-instances/container-instances-confidential-overview.md) serverless infrastructure.
- If you need to orchestrate containerized workloads, consider using [Virtual nodes on Azure Container Instances (C-VN2) for Azure Kubernetes Service (AKS)](../../../container-instances/container-instances-virtual-nodes.md).

## Frequently asked questions

### How does the DCsv3/DCdsv3-series retirement affect me?
If you're running your workload on DCsv3/DCdsv3-series, either by using Azure Linux, Windows, and Dedicated Host virtual machines, Virtual Machine Scale Sets, or by having app-enclave aware containers running on Azure Kubernetes Service, this retirement affects you.

### What is the migration timeline?
On October 31, 2029, DCsv3/DCdsv3-series virtual machines (VMs) retire. Before that date, migrate your workloads to new generation Confidential Virtual Machines (CVMs) or Azure Confidential Container Instances. If you prefer global availability and want to lift and shift your workloads, consider using [DCasv6](../general-purpose/dcasv6-series.md)/[DCadsv6](../general-purpose/dcadsv6-series.md)/[DCesv6](../general-purpose/dcesv6-series.md)/[DCedsv6](../general-purpose/dcedsv6-series.md) or memory optimized [ECasv6](../memory-optimized/ecasv6-series.md)/[ECadsv6](../memory-optimized/ecadsv6-series.md)/[ECesv6](../../../virtual-machines/ecesv6-series.md)/[ECedsv6](../../../virtual-machines/ecedsv6-series.md) confidential VMs (CVMs), or [Azure Confidential Container Instances (C-ACI)](../../../container-instances/container-instances-confidential-overview.md) serverless infrastructure.

### Will DCsv3/DCdsv3-series VMs still allow new customer sign-ups?
**Starting November 1, 2026**, capacity restrictions apply to DCsv3/DCdsv3-series virtual machines and no new subscription is allowed.

### Will Microsoft continue to support my current workload?
Yes, support continues for your workloads on DCsv3/DCdsv3 virtual machines until the retirement date. You continue to receive SLA assurance, infrastructure updates, and maintenance.

### Will other services built on top of the DCsv3/DCdsv3 SKU still be available after the SKU retires?
No, all uses of the DCsv3/DCdsv3 SKU retire simultaneously in October 2029, including those on Azure Kubernetes Service and Azure Virtual Machine Scale Sets.

### Will DCsv3/DCdsv3-series VMs provide any new features during the retirement period?
No, we aren't taking any feature requests or building new features for the DCsv3/DCdsv3-series VMs. Instead, we focus on next-generation lift-and-shift offerings with more memory per vCPU, faster SSD storage, global availability, and a cloud-native approach with containerized workloads and serverless infrastructure.

### Will DCsv3-series and DCdsv3-series VMs be available in new regions?
No, we won't deploy DCsv3-series and DCdsv3-series VMs in new Azure regions. Check availability in the existing DCsv3/DCdsv3 regions.

### How can I get a quota for the target VM size?
Follow the guide to [request an increase in vCPU quota by VM family](/azure/quotas/per-vm-quota-requests).

### What should I do if there are no CVMs available in my current region?
If there are no CVMs available in your current region, you can consider the following options:
- Check availability in nearby regions: Look for CVMs in nearby regions that might have the required capacity.
- Contact Azure Support: Reach out to Azure Support for assistance and to explore alternative solutions that meet your requirements.

### How does migration affect my current billing? 
 There might be changes to your Azure Virtual Machines billing because of this retirement. Refer to the Azure Virtual Machines [pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/linux-previous/) for more information.

### I'm on Reserved Instances (RIs) with DCsv3/DCdscv3. How Do I Handle Migration?
If you have active 3-year or 1-year Reserved Instances for DCsv3/DCdsv3-series VMs, follow these steps:
1. Review Current Reservations
    * Check your active RIs in the [Azure portal](/azure/cost-management-billing/reservations/manage-reserved-vm-instance).
   * Identify which RIs expire or are affected by the VM retirement.

1. Migrate and Manage Your RIs <br>Depending on your business needs, consider these options:
- Exchange Existing Reservations:
   * Swap current RIs for a new VM series without any penalties.
   * Refer to the [RI Exchange Guide](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations).
- Trade-In for Savings Plan:
   * Convert your existing RIs into an Azure Savings Plan for compute.
   * This offer flexibility across VM families and regions.
   * Follow the [Azure RI Trade-In Tutorial](/azure/cost-management-billing/savings-plan/reservation-trade-in).
- Purchase New RIs:
    * Buy new reservations that align with your new VM series.
   * Consider shorter terms (1-year) for flexibility.

### How can I get transition help and support during migration?
If you have any questions, you can [create a support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) through the Azure portal for technical help.

### What happens after retirement date?
After October 31, 2029, any remaining DCsv3/DCdsv3-series virtual machine subscriptions stop working and no longer incur billing charges. To avoid disruption, migrate ahead of the retirement schedule.

## Help and support

If you have questions, ask community experts in [Microsoft Q&A](/answers/topics/azure-virtual-machines.html). If you have a support plan and need technical help, [create a support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest):

In the [Help + support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) page, select **Create a support request**. Follow the **New support request** page instructions. Use the following values:
   * For **Issue type**, select **Technical**.
   * For **Service**, select **My services**.
   * For **Service type**, select **Virtual Machine running Windows/Linux**.
   * For **Resource**, select your VM.
   * For **Problem type**, select **Assistance with resizing my VM**`.
   * For **Problem subtype**, select the option that applies to you.

Follow instructions in the **Solutions** and **Details** tabs, as applicable, and then **Review + create**.

## Next steps

- Learn more about CVMs on AMD's fourth-generation EPYC™ processors [DCasv6](../general-purpose/dcasv6-series.md)/[DCadsv6](../general-purpose/dcadsv6-series.md) 
- Learn more about memory optimized CVMs on AMD's fourth-generation EPYC™ processors [ECasv6](../memory-optimized/ecasv6-series.md)/[ECadsv6](../memory-optimized/ecadsv6-series.md) series.
- Learn more about CVMs on Intel® 5th Generation Xeon® Scalable processors [DCesv6](../general-purpose/dcesv6-series.md)/[DCedsv6](../general-purpose/dcedsv6-series.md) 
- Learn more about memory optimized CVMs on Intel® 5th Generation Xeon® Scalable processors [ECesv6](../../../virtual-machines/ecesv6-series.md)/[ECedsv6](../../../virtual-machines/ecedsv6-series.md) confidential VMs (CVMs).
- Learn more about [Azure Confidential Container Instances (C-ACI)](../../../container-instances/container-instances-confidential-overview.md).
- Learn more about [Virtual nodes on Azure Container Instances (C-VN2) for Azure Kubernetes Service (AKS)](../../../container-instances/container-instances-virtual-nodes.md).

