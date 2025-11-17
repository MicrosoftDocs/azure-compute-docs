---
title: Resource availability & quota limits for Azure Container Instances (ACI)
description: Availability and quota limits of compute and memory resources for the Azure Container Instances service in different Azure regions.
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: concept-article
ms.date: 03/27/2025
ms.custom: references_regions

# Customer intent: As a cloud architect, I want to understand the resource availability and quota limits for Azure Container Instances so that I can effectively plan and optimize container deployments based on regional capacities and service limitations.
---
# Resource availability & quota limits for ACI

This article details the availability and quota limits of Azure Container Instances compute, memory, and storage resources in Azure regions and by target operating system. For a general list of available regions for Azure Container Instances, see [available regions](https://azure.microsoft.com/regions/services/). 

Values presented are the maximum resources available per deployment of a [container group](container-instances-container-groups.md). Values are current at time of publication. 

> [!NOTE] 
> Container groups created within these resource limits are subject to availability within the deployment region. When a region is under heavy load, you may experience a failure when deploying instances. To mitigate such a deployment failure, try deploying instances with lower resource settings, or try your deployment at a later time or in a different region with available resources. 

## Default Quota Limits 

All Azure services include certain default limits and quotas for resources and features. This section details the default quotas and limits for Azure Container Instances.  

Use the [List Usage](/rest/api/container-instances/2022-09-01/location/list-usage) API to review current quota usage in a region for a subscription. 

Certain default limits and quotas can be increased. To request an increase of one or more resources that support such an increase, submit an [Azure support request][azure-support] (select "Quota" for **Issue type**). 

> [!IMPORTANT]  
> Not all limit increase requests are guaranteed to be approved.

### Unchangeable (Hard) Limits 

The following limits are default limits that can’t be increased through a quota request. Any quota increase requests for these limits won't be approved.  

| Resource | Actual Limit | 
| --- | :--- | 
| Number of containers per container group | 60 | 
| Number of volumes per container group | 20 | 
| Ports per IP | 5 | 
| Container instance log size - running instance | 4 MB | 
| Container instance log size - stopped instance | 16 KB or 1,000 lines | 


### Changeable Limits (Eligible for Quota Increases) 

| Resource | Actual Limit | 
| --- | :--- | 
| Standard sku container groups per region per subscription | 100 | 
| Standard sku cores (CPUs) per region per subscription | 100 |  
| Container group creates per hour |300<sup>1</sup> | 
| Container group creates per 5 minutes | 100<sup>1</sup> | 
| Container group deletes per hour | 300<sup>1</sup> | 
| Container group deletes per 5 minutes | 100<sup>1</sup> |

> [!NOTE]
> 1: Indicates that the feature maximum is configurable and may be increased through a support request. For more information on how to request a quota increase, see the [How to request a quota increase section of Increase VM-family vCPU quotes](/azure/quotas/per-vm-quota-requests).
>
> You can also create a support ticket if you'd like to discuss your specific needs with the support team.

## Standard Container Resources 

By default, the following resources are available general purpose (standard core SKU) containers in general deployments and [Azure virtual network](container-instances-vnet.md) deployments) for Linux & Windows containers. These maximums are hard limits and can't be increased.

For a general list of available regions for Azure Container Instances, see [available regions](https://azure.status.microsoft/status/#services/). 

> [!IMPORTANT]
> If you're encountering deployment failures when creating container groups with more than 4 vCPUs and 16 GB of memory, it may be due to regional capacity limits. To resolve this, you can request a quota increase to enable additional capacity for your workloads. In the meantime, consider deploying to another region, where additional capacity may be more available.

| Max CPU | Max Memory (GB) | Virtual network Max CPU | Virtual network Max Memory (GB) | Storage (GB) | 
| :---: | :---: | :----: | :-----: | :-------: |
| 31 | 240 | 31 | 240 | 50 | 

The following resources are available in all Azure Regions supported by Azure Container Instances. For a general list of available regions for Azure Container Instances, see [available regions](https://azure.status.microsoft/status/#services/). 

## Confidential Container Resources

The following maximum resources are available to a container group deployed using [Confidential Containers](container-instances-confidential-overview.md). These maximums are hard limits and can't be increased.

> [!IMPORTANT]
> If you're encountering deployment failures when creating container groups with more than 4 vCPUs and 16 GB of memory, it may be due to regional capacity limits. To resolve this, you can request a quota increase to enable additional capacity for your workloads. In the meantime, consider deploying to another region, where additional capacity may be more available.

| Max CPU | Max Memory (GB) | Virtual network Max CPU | Virtual network Max Memory (GB) | Storage (GB) | 
| :---: | :---: | :----: | :-----: | :-------: |
| 31 | 180 | 32 | 180 | 50 | 

## Spot Container Resources (Preview)

The following maximum resources are available to a container group deployed using [Spot Containers](container-instances-spot-containers-overview.md) (preview). These maximums are hard limits and can't be increased.

> [!NOTE]
> Spot Containers are only available in the following regions at this time: East US 2, West Europe, and West US.

| Max CPU | Max Memory (GB) | Virtual network Max CPU | Virtual network Max Memory (GB) | Storage (GB) | 
| :---: | :---: | :----: | :-----: | :-------: |
| 4 | 16 | N/A | N/A | 50 | 

## GPU Container Resources (Preview) 

> [!IMPORTANT]
> This product is retired as of July 14, 2025.

The following maximum resources are available to a container group deployed with [GPU resources](container-instances-gpu.md) (preview). These maximums are hard limits and can't be increased.

| GPU SKUs | GPU count | Max CPU | Max Memory (GB) | Storage (GB) | 
| --- | --- | --- | --- | --- | 
| V100 | 1 | 6 | 112 | 50 | 
| V100 | 2 | 12 | 224 | 50 | 
| V100 | 4 | 24 | 448 | 50 | 

## Next steps 

Certain default limits and quotas can be increased. To request an increase of one or more resources that support such an increase, submit an [Azure support request][azure-support] (select "Quota" for **Issue type**). 

Let the team know if you'd like to see other regions or increased resource availability at [aka.ms/aci/feedback](https://aka.ms/aci/feedback). 

For information on troubleshooting container instance deployment, see [Troubleshoot deployment issues with Azure Container Instances](container-instances-troubleshooting.md).

<!-- LINKS - External --> 

[az-region-support]: ../availability-zones/az-overview.md#regions 

[azure-support]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest 

 

 
