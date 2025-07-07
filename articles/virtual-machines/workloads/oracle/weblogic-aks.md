---
title: What are solutions for running Oracle WebLogic Server on the Azure Kubernetes Service
description: Learn how to run Oracle WebLogic Server on the Azure Kubernetes Service.
author: rezar
ms.service: oracle-on-azure
ms.collection: linux
ms.topic: how-to
ms.date: 10/08/2024 
ms.author: rezar
ms.reviewer: cynthn
ms.custom: devx-track-java, devx-track-javaee, devx-track-javaee-wls, devx-track-javaee-wls-aks
# Customer intent: As a Java application developer, I want to implement Oracle WebLogic Server on Azure Kubernetes Service, so that I can leverage Kubernetes for container orchestration and ensure reliable, scalable, and manageable deployments of my enterprise applications.
---
# What are solutions for running Oracle WebLogic Server on the Azure Kubernetes Service?

**Applies to:** :heavy_check_mark: Linux VMs 

This page describes the solutions for running Oracle WebLogic Server (WLS) on the Azure Kubernetes Service (AKS). These solutions are developed and supported by Oracle and Microsoft.

It's also possible to run WebLogic Server on Azure Virtual Machines. The solutions to do so are described in the article [Running Oracle WebLogic Server on the Azure Virtual Machines](./oracle-weblogic.md?toc=/azure/developer/java/ee/toc.json&bc=/azure/developer/java/breadcrumb/toc.json).

WebLogic Server is a leading Java application server running some of the most mission-critical enterprise Java applications across the globe. WebLogic Server forms the middleware foundation for the Oracle software suite. Oracle and Microsoft are committed to empowering WebLogic Server customers with choice and flexibility to run workloads on Azure as a leading cloud platform.

## WebLogic Server on AKS certified and supported
WebLogic Server is certified by Oracle and Microsoft to run on AKS. The WebLogic Server on AKS solutions are aimed at making it as easy as possible to run your containerized and orchestrated Java applications on Kubernetes. The solutions are focused on reliability, scalability, manageability, and enterprise support.

WebLogic Server clusters are fully enabled to run on Kubernetes via the WebLogic Kubernetes Operator (referred to simply as the 'Operator' from here onward). The Operator follows the standard Kubernetes Operator pattern. It simplifies the management and operation of WebLogic domains on Kubernetes by automating otherwise manual tasks and adding extra operational reliability features. The Operator supports Oracle WebLogic Server 12c, Oracle Fusion Middleware Infrastructure 12c and beyond. For details on the Operator, refer to the [official documentation from Oracle](https://oracle.github.io/weblogic-kubernetes-operator/).

## WebLogic Server on AKS solution template
Beyond certifying WebLogic Server on AKS, Oracle and Microsoft jointly provide the Azure Marketplace offer of [Oracle WebLogic Server on AKS](https://aka.ms/wls-aks-portal). The goal is to make it quick and easy to migrate WebLogic Server workloads to AKS. The offer does so by automating the provisioning of many Java and Azure resources. The automatically provisioned resources include an AKS cluster, the WebLogic Kubernetes Operator, WebLogic Server Docker images, and the Azure Container Registry (ACR). It's possible to use an existing AKS cluster or ACR instance with the offer. The offer 
supports configuring load balancing with Azure App Gateway or the Azure Load Balancer to ease database connectivity, publish metrics to Azure Monitor, and mount Azure Files as Kubernetes Persistent Volumes. The currently supported database integrations include Azure PostgreSQL, Azure MySQL, Azure SQL, and the Oracle Database on the Oracle Cloud or Azure.

:::image type="content" source="media/oracle-weblogic/wls-aks-demo.gif" alt-text="You can use the marketplace solution to deploy WebLogic Server on AKS":::

After the solution template performs boilerplate resource provisioning and configuration, you can deploy your application to AKS. This is typically done using a DevOps tool like, GitHub Actions and tools from WebLogic Kubernetes tooling like, the WebLogic Image Tool and WebLogic Deploy Tooling. You're free to customize the deployment further.

If you'd like to provide feedback or work closely on your migration scenarios with the engineering team developing WebLogic on AKS solutions, fill out this short [survey on WebLogic migration](https://aka.ms/wls-on-azure-survey) and include your contact information. The team of program managers, architects, and engineers will promptly get in touch with you to initiate close collaboration.

## Manual guidance, scripts, and samples for WebLogic Server on AKS

Oracle and Microsoft also provide basic step-by-step guidance, scripts, and samples for running WebLogic Server on AKS. The guidance is suitable for customers that wish to remain as close as possible to a native Kubernetes manual deployment experience as an alternative to using a solution template. The guidance is incorporated into the Azure Kubernetes Service sample section of the [Operator documentation](https://oracle.github.io/weblogic-kubernetes-operator/samples/azure-kubernetes-service/). The guidance allows a high degree of configuration and customization.

The guidance supports two ways of deploying WebLogic Server domains to AKS. Domains can be deployed directly to Kubernetes Persistent Volumes. This deployment option is good if you want to migrate to AKS but still want to administer WebLogic Server using the Admin Console or the WebLogic Scripting Tool (WLST). The option also allows you to move to AKS without adopting Docker development. The Kubernetes native way of deploying WebLogic Server domains to AKS is to build custom container images based on official WebLogic Server images from the Oracle Container Registry, publish the custom images to ACR, and deploy the domain to AKS using the Operator.

## Deployment architectures

The solutions for running Oracle WebLogic Server on the Azure Kubernetes Service enable a wide range of production-ready deployment architectures with relative ease.

:::image type="content" source="media/oracle-weblogic/wls-aks-architecture.jpg" alt-text="Complex WebLogic Server deployments are enabled on AKS":::

Beyond what the solutions provide you have complete flexibility to customize your deployments further. It's likely on top of deploying applications to integrate further Azure resources with your deployments or tune the deployments to your specific applications. You're encouraged to provide feedback in the [survey](https://aka.ms/wls-on-azure-survey) on further improving WebLogic on AKS solutions.

## Next steps

The following articles provide more information on getting started with these technologies.

* [Deploy a Java application with WebLogic Server on an Azure Kubernetes Service (AKS) cluster](/azure/aks/howto-deploy-java-wls-app?toc=/azure/developer/java/ee/toc.json&bc=/azure/developer/java/breadcrumb/toc.json)
* [What are solutions for running Oracle WebLogic Server on Azure Virtual Machines?](./oracle-weblogic.md?toc=/azure/developer/java/ee/toc.json&bc=/azure/developer/java/breadcrumb/toc.json)

For more information about the Oracle WebLogic offers at Azure Marketplace, see [Oracle WebLogic Server on Azure](https://aka.ms/wls-contact-me). These offers are all _Bring-Your-Own-License_. They assume that you already have the appropriate licenses with Oracle and are properly licensed to run offers in Azure.

You're encouraged to [connect with the development team](https://aka.ms/wls-on-azure-survey) and provide feedback on further improving WebLogic on AKS solutions.
