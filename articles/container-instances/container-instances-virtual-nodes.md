---
title: Virtual nodes on Azure Container Instances
description: Learn more about virtual nodes.
ms.topic: concept-article
ms.author: jussong
author: jussong
ms.service: azure-container-instances
services: container-instances
ms.date: 11/10/2024
ms.custom: mvc
# Customer intent: As a cloud architect, I want to utilize virtual nodes on Azure Container Instances to quickly scale my containerized workloads, so that I can efficiently manage resources and reduce costs with per-second billing while maintaining optimal performance and security through confidential containers.
---

# Virtual nodes on Azure Container Instances

This article introduces how virtual nodes on Azure Container Instances (ACI) enable you to orchestrate containerized workloads. This article provides background about the feature set, availability, and resources.

Virtual nodes on Azure Container Instances allow you to deploy pods in your Azure Kubernetes Service (AKS) cluster that run as container groups in ACI. Since virtual nodes are backed by ACI's serverless infrastructure, you can quickly scale up your workload without needing to wait for the Kubernetes cluster autoscaler to deploy VM compute nodes. You can rapidly scale your application workloads and only pay for the time that your pods are running.

Virtual nodes on Azure Container Instances is an evolution on the existing [virtual nodes](/azure/aks/virtual-nodes) offering for AKS. It offers significantly better support for Kubernetes functionality, removing most of the limitations of the prior implementation. Additionally, it enables you to take advantage of advanced ACI functionality like [confidential containers](./container-instances-confidential-overview.md).

Virtual nodes on Azure Container Instances was featured in the AKS keynote at KubeCon NA 2023, with a [segment showing some of its many improvements](https://www.youtube.com/watch?v=yJOc3D52_Is&t=2330s).

## Features of virtual nodes on Azure Container Instances

### Rapidly scale your application workloads

Pods deployed in a virtual node run on ACI's serverless infrastructure. You don't need to first provision additional VM compute nodes in your AKS cluster when scaling up your workload.

### Only pay per second of execution time

With standard AKS cluster nodes, you pay for provisioned VMs even if you are not actively utilizing their capacity. Pods in a virtual node use ACI's per-second billing, so you pay based on what you need.

### Deploy confidential container groups

Virtual nodes can be configured to run your pods as [confidential container groups in ACI](./container-instances-confidential-overview.md). Confidential containers on Azure Container Instances can run with verifiable execution policies that enable customers to have control over what software and actions are allowed to run within the TEE. These execution policies help to protect against bad actors creating unexpected application modifications that could potentially leak sensitive data. Customers author execution policies through provided [tooling](https://github.com/Azure/azure-cli-extensions/blob/main/src/confcom/azext_confcom/README.md), and cryptographic proofs verify the policies.

## Region availability

Virtual nodes on Azure Container Instances is supported in all regions where ACI is available. Virtual nodes on confidential Azure Container Instances is supported in all regions where confidential ACI is available.

For more information, see [Resource availability & quota limits for ACI](container-instances-resource-and-quota-limits.md).

## Resources

* [Helm chart for deploying virtual nodes on Azure Container Instances](https://github.com/microsoft/virtualnodesOnAzureContainerInstances/tree/main/Helm/virtualnode)
* [Guidance for customizing pods running on virtual nodes](https://github.com/microsoft/virtualnodesOnAzureContainerInstances/blob/main/Docs/PodCustomizations.md)
* [Guidance for customizing your virtual nodes installation](https://github.com/microsoft/virtualnodesOnAzureContainerInstances/blob/main/Docs/NodeCustomizations.md)
* [Guidance for deploying high-scale workloads on virtual nodes](https://github.com/microsoft/virtualnodesOnAzureContainerInstances/blob/main/Docs/HighScaleBestPractices.md)

## Next steps

* To try out virtual nodes on Azure Container Instances, see [Deploy virtual nodes on Azure Container Instances in your AKS cluster](./container-instances-tutorial-virtual-nodes-helm.md)

* If you want to manage multi-instance container groups, see [About NGroups](container-instance-ngroups/container-instances-about-ngroups.md).
