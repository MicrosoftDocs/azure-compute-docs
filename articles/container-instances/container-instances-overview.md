---
title: Serverless containers in Azure
description: The Azure Container Instances service offers the fastest and simplest way to run isolated containers in Azure, without having to manage virtual machines and without having to adopt a higher-level orchestrator.
ms.topic: overview
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.date: 11/17/2025
ms.update-cycle: 1095-days
ms.custom: mvc, linux-related-content
# Customer intent: As a cloud developer, I want to deploy and manage containerized applications using serverless containers, so that I can simplify resource management and achieve faster startup times without the overhead of virtual machines.
---

# What is Azure Container Instances?

Containers are becoming the preferred way to package, deploy, and manage cloud applications. Azure Container Instances offers the fastest and simplest way to run Linux or Windows containers in Azure, without having to manage any virtual machines and without having to adopt a higher-level service.

ACI supports [regular](container-instances-container-groups.md), [confidential](container-instances-confidential-overview.md), and [Spot](container-instances-spot-containers-overview.md) containers. ACI can be used as [single-instance](container-instances-container-groups.md) or multi-instance via [NGroups](container-instance-ngroups/container-instances-about-ngroups.md), or you can get orchestration capabilities by deploying pods in your Azure Kubernetes Service (AKS) cluster via [virtual nodes on ACI](container-instances-virtual-nodes.md). For even faster startup times, ACI supports [standby pools](container-instances-standby-pool-overview.md).

## Fast startup times

Containers offer significant startup benefits over virtual machines (VMs). Azure Container Instances can start containers in Azure in seconds, without the need to provision and manage VMs.

Bring Linux or Windows container images from Docker Hub, a private [Azure container registry](/azure/container-registry/), or another cloud-based Docker registry. Visit the [FAQ](container-instances-faq.yml) to learn which registries ACI supports. Azure Container Instances caches several common base OS images, helping speed deployment of your custom application images.

For even faster startup times, ACI supports [standby pools](container-instances-standby-pool-overview.md).

## Container access

Azure Container Instances enables exposing your container groups directly to the internet with an IP address and a fully qualified domain name (FQDN). When you create a container instance, you can specify a custom DNS name label so your application is reachable at *customlabel*.*azureregion*.azurecontainer.io.

Azure Container Instances also supports executing a command in a running container by providing an interactive shell to help with application development and troubleshooting. Access takes places over HTTPS, using TLS to secure client connections.

> [!IMPORTANT]
> Azure Container Instances requires all secure connections from servers and applications to use TLS 1.2. Support for TLS 1.0 and 1.1 has been retired.

## Compliant deployments

### Hypervisor-level security

Historically, containers offered application dependency isolation and resource governance but were insufficiently hardened for hostile multitenant usage. Azure Container Instances guarantees your application is as isolated in a container as it would be in a VM.

### Customer data

The Azure Container Instances service doesn't store customer data. It does, however, store the subscription IDs of the Azure subscription used to create resources. Storing subscription IDs is required to ensure your container groups continue running as expected.

## Custom sizes

Containers are typically optimized to run just a single application, but the exact needs of those applications can differ greatly. Azure Container Instances provides optimum utilization by allowing exact specifications of CPU cores and memory. You pay based on what you need and get billed by the second, so you can fine-tune your spending based on actual need.

For compute-intensive jobs such as machine learning, Azure Container Instances can schedule Linux containers to use NVIDIA Tesla [GPU resources](container-instances-gpu.md) (preview).

## Persistent storage

To retrieve and persist state with Azure Container Instances, we offer direct [mounting of Azure Files shares](./container-instances-volume-azure-files.md) backed by Azure Storage.

## Linux and Windows containers

Azure Container Instances can schedule both Windows and Linux containers with the same API. You can specify your OS type preference when you create your [container groups](container-instances-container-groups.md).

Some features are currently restricted to Linux containers:

* Multiple containers per container group
* Volume mounting ([Azure Files](container-instances-volume-azure-files.md), [emptyDir](container-instances-volume-emptydir.md), [GitRepo](container-instances-volume-gitrepo.md), [secret](container-instances-volume-secret.md))
* [Resource usage metrics](monitor-azure-container-instances.md#get-metrics) with Azure Monitor
* [GPU resources](container-instances-gpu.md) (preview)

For Windows container deployments, use images based on common [Windows base images](./container-instances-faq.yml#what-windows-base-os-images-are-supported-).

## Run multiple containers in a single container group

Azure Container Instances supports scheduling of [multiple containers within a single container group](container-instances-container-groups.md) that share the same container host, local network, storage, and lifecycle. This enables you to combine your main application container with other supporting role containers, such as logging sidecars.

## Virtual network deployment

> [!IMPORTANT]
> If you deploy your container group into a virtual network, you must use a NAT gateway for outbound connectivity. This is the only supported configuration for outbound connectivity from your container group in a virtual network. See [Configure a NAT gateway for static IP address for outbound traffic from a container group](./container-instances-nat-gateway.md) for more information on how to configure this.

Azure Container Instances enables [deployment of container instances into an Azure virtual network](container-instances-vnet.md). When deployed into a subnet within your virtual network, container instances can communicate securely with other resources in the virtual network, including those that are on premises (through [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) or [ExpressRoute](/azure/expressroute/expressroute-introduction)).

## Availability zones support

Azure Container Instances supports [zonal container group deployments](/azure/reliability/reliability-container-instances), meaning the instance is pinned to a specific, self-selected availability zone. The availability zone can be specified per container group.

## Managed identity

Azure Container Instances supports using [managed identity with your container group](container-instances-managed-identity.md), which enables your container group to authenticate to any service that supports Microsoft Entra authentication without managing credentials in your container code.

## Managed identity authenticated image pull

Azure Container Instances can authenticate with an Azure Container Registry (ACR) instance [using a managed identity](/azure/container-instances/using-azure-container-registry-mi), allowing you to pull the image without having to include a username and password directly in your container group definition.

## Confidential container deployment

Confidential containers on ACI enable you to run containers in a trusted execution environment (TEE) that provides hardware-based confidentiality and integrity protections for your container workloads. Confidential containers on ACI can protect data-in-use and encrypts data being processed in memory. Confidential containers on ACI are supported as a SKU that you can select when deploying your workload. For more information, see [confidential container groups](./container-instances-confidential-overview.md).

## Spot container deployment

ACI Spot containers allow customers to run interruptible, containerized workloads on unused Azure capacity at discounted prices of up to 70% compared to regular-priority ACI containers. ACI spot containers may be preempted when Azure encounters a shortage of surplus capacity, and they're suitable for workloads without strict availability requirements. Customers are billed for per-second memory and core usage. To utilize ACI Spot containers, you can deploy your workload with a specific property flag indicating that you want to use Spot container groups and take advantage of the discounted pricing model.
For more information, see [Spot container groups](container-instances-spot-containers-overview.md).

## NGroups

NGroups provides advanced capabilities for managing multiple related container groups. NGroups provides support for maintaining a specified number of container groups, performing rolling upgrades, deploying across multiple availability zones, using load balancers for ingress, and deploying confidential containers. For more information, see [About NGroups](container-instance-ngroups/container-instances-about-ngroups.md).

## Virtual nodes on Azure Container Instances

[Virtual nodes on Azure Container Instances](container-instances-virtual-nodes.md) allow you to deploy pods in your Azure Kubernetes Service (AKS) cluster that run as container groups in ACI. This allows you to orchestrate your container groups using familiar Kubernetes constructs. Since virtual nodes are backed by ACI's serverless infrastructure, you can quickly scale up your workload without needing to wait for the Kubernetes cluster autoscaler to deploy VM compute nodes.

## Considerations

User’s credentials passed via command line interface (CLI) are stored as plain text in the backend. Storing credentials in plain text is a security risk; Microsoft advises customers to store user credentials in CLI environment variables to ensure they're encrypted/transformed when stored in the backend.

If your container group stops working, we suggest trying to restart your container, checking your application code, or your local network configuration before opening a [support request][azure-support].

Container Images can't be larger than 15 GB, any images above this size may cause unexpected behavior: [How large can my container image be?](./container-instances-faq.yml)

Some Windows Server base images are no longer compatible with Azure Container Instances:  
[What Windows base OS images are supported?](./container-instances-faq.yml)

If a container group restarts, the container group’s IP may change. We advise against using a hard coded IP address in your scenario. If you need a static public IP address, use Application Gateway: [Static IP address for container group - Azure Container Instances | Microsoft Learn](./container-instances-application-gateway.md)

There are ports that are reserved for service functionality. We advise you not to use these ports because using them leads to unexpected behavior: [Does the ACI service reserve ports for service functionality?](./container-instances-faq.yml)

 If you’re having trouble deploying or running your container, first check the [Troubleshooting Guide](./container-instances-troubleshooting.md) for common mistakes and issues 

Your container groups may restart due to platform maintenance events. These maintenance events are done to ensure the continuous improvement of the underlying infrastructure: [Container had an isolated restart without explicit user input](./container-instances-faq.yml)

ACI doesn't allow [privileged container operations](./container-instances-faq.yml). We advise you to not depend on using the root directory for your scenario.

## Next steps

Try deploying a container to Azure with a single command using our quickstart guide:

> [!div class="nextstepaction"]
> [Azure Container Instances Quickstart](container-instances-quickstart.md)

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[azure-support]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
