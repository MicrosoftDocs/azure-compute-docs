---
title: What is IBM Spectrum Symphony on Azure?
description: Learn how IBM Spectrum Symphony integrates with Azure Compute Fleet to extend enterprise HPC workload orchestration to Azure compute capacity.
author: rayoef
ms.author: rayoflores
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.topic: overview
ms.date: 09/22/2026
ai-usage: ai-assisted
# Customer intent: As an HPC architect, I want to understand how IBM Spectrum Symphony integrates with Azure Compute Fleet, so that I can extend my workloads to Azure compute capacity.
---

# What is IBM Spectrum Symphony on Azure?

IBM Spectrum Symphony integrates with Azure Compute Fleet to extend enterprise high-performance computing (HPC) workload orchestration environments to Azure. The integration dynamically provisions Azure compute capacity as workload demand changes while maintaining your existing Symphony workload policies, queues, service-level objectives, and operational processes.

## How the integration works

The IBM Spectrum Symphony Provider Plugin for Microsoft Azure Compute Fleet connects IBM Symphony Host Factory to Azure Compute Fleet. The plugin translates Symphony resource requirements into Azure Compute Fleet requests. Symphony schedules and orchestrates workloads, while Azure Compute Fleet provides access to eligible compute capacity.

:::image type="complex" source="./media/ibm-spectrum-symphony/architecture.svg" alt-text="Diagram that shows IBM Spectrum Symphony connected to Azure Compute Fleet.":::
IBM Spectrum Symphony sends job, policy, and resource-demand information to Host Factory. The Azure resource connector translates the demand into an Azure Compute Fleet Launch mode request. Azure Compute Fleet selects eligible Spot and pay-as-you-go VMs. The provisioned VMs join the Symphony cluster as execution hosts. Symphony manages the workloads and execution-host lifecycle, while Azure provides the compute capacity.
:::image-end:::

The integration uses the following flow:

1. IBM Spectrum Symphony evaluates queued jobs, workload priorities, policies, and resource demand.
1. Host Factory sends the resource requirements to the Azure resource connector.
1. The resource connector translates the requirements into an Azure Compute Fleet API request.
1. Azure Compute Fleet selects eligible virtual machine (VM) types and a mix of Spot and pay-as-you-go capacity.
1. The provisioned VMs join the Symphony cluster as execution hosts and run queued jobs.
1. As demand decreases, Symphony can release the Azure resources.

Symphony manages the workload and its policies. Azure Compute Fleet manages access to the underlying compute capacity.

The provider uses [Azure Compute Fleet Launch mode](/azure/azure-compute-fleet/launch-mode). Launch mode provisions VMs in a single request and then hands off their lifecycle to Symphony. The fleet resource is deleted automatically after provisioning, but the VMs continue to run until Symphony releases them.

## Key capabilities

The integration provides the following capabilities:

- **Dynamic scaling:** Provision and scale Azure compute capacity as workload demand changes.
- **Policy-driven elasticity:** Maintain existing workload priorities and service-level objectives when you extend workloads to Azure.
- **Flexible VM selection:** Provision across eligible Azure VM families based on workload requirements.
- **Flexible purchasing:** Combine [Azure Spot Virtual Machines](spot-vms.md) and pay-as-you-go VMs.
- **Hybrid orchestration:** Extend an existing Symphony environment to Azure without replacing your workload orchestration processes.
- **Workload-based provisioning:** Select capacity based on attributes such as virtual CPU (vCPU), memory, and storage.

Azure Compute Fleet can use multiple VM types and allocation strategies to optimize for cost, capacity, or both. This flexibility increases the range of eligible capacity compared to requesting a single VM size.

## Workload scenarios

Use the integration for compute-intensive and data-intensive distributed applications that benefit from elastic batch capacity. Example scenarios include:

- Electronic design automation.
- Engineering analysis.
- Financial risk calculations.
- Scientific computing.

## Considerations

Before you use the integration, consider the following factors:

- Subscription, region, and VM family determine the standard Azure VM and vCPU quotas that apply. Azure Compute Fleet doesn't increase these quotas.
- Spot VMs can be evicted when Azure needs the capacity or when the price exceeds the maximum price you set. Design workloads to tolerate interruptions.
- Available VM types and capacity vary by Azure region.
- Your IBM Spectrum Symphony licensing and support terms apply to the Symphony environment and provider plugin.

## Next steps

- Review the [IBM Spectrum Symphony Provider Plugin for Microsoft Azure Compute Fleet](https://marketplace.microsoft.com/product/ibm-usa-ny-armonk-hq-6275750-ibmcloud-asperia.azure-compute-fleet-for-ibm-spectrum-symphony).
- Learn more about [IBM Spectrum Symphony](https://www.ibm.com/products/spectrum-symphony).
- Learn [what Azure Compute Fleet is](/azure/azure-compute-fleet/overview).
- Review [allocation strategies for Azure Compute Fleet](/azure/azure-compute-fleet/allocation-strategies).
