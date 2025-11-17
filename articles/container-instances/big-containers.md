---
title: Big containers on Azure container instances
description: Capabilities and benefits of Big Containers on Azure Container Instances.
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: concept-article
ms.date: 03/27/2025
# Customer intent: As a cloud developer, I want to deploy high-performance container workloads with increased vCPU and memory on Azure Container Instances, so that I can efficiently handle data-intensive applications and scale resources seamlessly according to business demands.
---
# Big Containers on Azure Container Instances
This article outlines the capabilities and benefits of Big Containers on Azure Container Instances. Customers can now deploy workloads with higher vCPU and memory for standard containers, confidential containers, containers with virtual networks, as well as containers utilizing virtual nodes to connect to AKS. This setup supports vCPU counts greater than 4 and memory capacities of 16 GB, with a maximum of 31 vCPU and 240 GB per standard container group and 31 vCPU and 180 GB per confidential container group. This feature removes limitations for compute and memory intensive workloads!   

## Benefits of Big Containers  

### Enhanced Performance  

More vCPUs mean better processing power, allowing for more efficient handling of complex tasks and applications. The enhanced performance from more vCPUs and larger GB capacity can lead to faster processing times and reduced latency, which can translate to cost savings in terms of time and productivity.  

### Increased Memory Capacity  

Larger container groups with more GB can handle bigger datasets and more extensive workloads, making them ideal for data-intensive applications.  

### Simplified Scalability  

Larger container groups provide the flexibility to scale up resources even higher as needed, accommodating growing business demands without compromising performance. Larger container SKUs can simplify the scaling process. Instead of managing many smaller containers, you can scale your applications with fewer, larger ones, potentially reducing the need for frequent scaling adjustments.  

## Scenarios for Big Containers  

These are a few scenarios that will benefit from Big Containers.   

### Data Inferencing  

Larger container SKUs are ideal for data inferencing tasks that require robust computational power. Examples include real-time fraud detection in financial transactions, predictive maintenance in manufacturing, and personalized recommendation engines in e-commerce. These containers ensure efficient and secure processing of large datasets for accurate predictions and insights.  

### Collaborative Analytics  

When multiple parties need to share and analyze data, larger container SKUs provide a secure and efficient solution. For instance, companies in healthcare can collaborate on patient data analytics while maintaining confidentiality. Similarly, research institutions can share large datasets for scientific studies without compromising data privacy.  

### Big Data Processing  

Organizations dealing with large-scale data processing can benefit from the enhanced capacity of larger container SKUs. Examples include processing customer data for targeted marketing campaigns, analyzing social media trends for sentiment analysis, and conducting large-scale financial modeling for risk assessment. These containers ensure efficient handling of extensive workloads.  

### High-Performance Computing  

High-performance computing applications, such as climate modeling, genomic research, and computational fluid dynamics, demand substantial computational power. Larger container SKUs provide the necessary resources to support these intensive tasks, enabling precise simulations and faster results.  

## Next Steps

To begin using Big Containers, follow these steps. 

1. If you plan to run containers larger than 4 vCPU and 16 GB, you must submit an [Azure support request][azure-support] (select "Quota" for **Support type**). 
2. Once your quota has been allocated, you can deploy your container groups through Azure portal, Azure CLI, PowerShell, ARM template, or any other medium that allows you to connect to your container groups in Azure. 

To learn more about Azure Container Instances, see [Serverless containers in Azure - Azure Container Instances | Microsoft Learn](./container-instances-overview.md)

[azure-support]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest 

