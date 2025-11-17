---
title: Container instances and container orchestration
description: Understand how Azure container instances interact with container orchestrators.
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: concept-article
ms.date: 11/17/2025
ms.custom: mvc
# Customer intent: As a cloud architect, I want to understand how Azure Container Instances integrate with orchestration platforms, so that I can effectively manage and scale containerized applications in a cloud environment.
---

# Azure Container Instances and container orchestrators

Because of their small size and application orientation, containers are well suited for agile delivery environments and microservice-based architectures. The task of automating and managing a large number of containers and how they interact is known as *orchestration*. Popular container orchestrators include Kubernetes, DC/OS, and Docker Swarm.

Azure Container Instances provides some of the basic scheduling capabilities of orchestration platforms. And while it doesn't cover the higher-value services that those platforms provide, Azure Container Instances can be complementary to them. This article describes the scope of what Azure Container Instances handles, and how full container orchestrators might interact with it.

## Traditional orchestration

The standard definition of orchestration includes the following tasks:

- **Scheduling**: Given a container image and a resource request, find a suitable machine on which to run the container.
- **Affinity/Anti-affinity**: Specify that a set of containers should run nearby each other (for performance) or sufficiently far apart (for availability).
- **Health monitoring**: Watch for container failures and automatically reschedule them.
- **Failover**: Keep track of what is running on each machine, and reschedule containers from failed machines to healthy nodes.
- **Scaling**: Add or remove container instances to match demand, either manually or automatically.
- **Networking**: Provide an overlay network for coordinating containers to communicate across multiple host machines.
- **Service discovery**: Enable containers to locate each other automatically, even as they move between host machines and change IP addresses.
- **Coordinated application upgrades**: Manage container upgrades to avoid application downtime, and enable rollback if something goes wrong.

## Orchestration with Azure Container Instances: A layered approach

Azure Container Instances enables a layered approach to orchestration, providing all of the scheduling and management capabilities required to run a single container, while allowing orchestrator platforms to manage multi-container tasks on top of it.

Because Azure manages the underlying infrastructure for container instances, an orchestrator platform doesn't need to concern itself with finding an appropriate host machine on which to run a single container. The elasticity of the cloud ensures that one is always available. Instead, the orchestrator can focus on the tasks that simplify the development of multi-container architectures, including scaling and coordinated upgrades.

## Scenarios

While orchestrator integration with Azure Container Instances is still nascent, we anticipate that a few different environments emerge:

### Orchestration of container instances exclusively

Because they start quickly and bill by the second, an environment based exclusively on Azure Container Instances offers the fastest way to get started and to deal with highly variable workloads.

### Combination of container instances and containers in Virtual Machines

For long-running, stable workloads, orchestrating containers in a cluster of dedicated virtual machines is typically cheaper than running the same containers with Azure Container Instances. However, container instances offer a great solution for quickly expanding and contracting your overall capacity to deal with unexpected or short-lived spikes in usage.

Rather than scaling out the number of virtual machines in your cluster, then deploying more containers onto those machines, the orchestrator can schedule the additional containers in Azure Container Instances, and delete them when they're no longer needed.

## Sample implementation: virtual nodes on Azure Container Instances

To rapidly scale application workloads in an [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) cluster, you can use *virtual nodes* created dynamically in Azure Container Instances. Virtual nodes register as nodes having unlimited capacity with your AKS cluster control plane. When you deploy pods in a virtual node in your AKS cluster, they run as container groups in ACI.

Virtual nodes currently support Linux container instances. See [Virtual nodes on Azure Container Instances](./container-instances-virtual-nodes.md) to learn more.

## Next steps

Create your first container with Azure Container Instances using the [quickstart guide](container-instances-quickstart.md).

<!-- IMAGES -->

<!-- LINKS -->
[aci-connector-k8s]: https://github.com/virtual-kubelet/azure-aci
[kubelet-doc]: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
[pod-doc]: https://kubernetes.io/docs/concepts/workloads/pods/pod/
