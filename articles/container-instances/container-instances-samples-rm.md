---
title: Azure Resource Manager template samples
description: Find Azure Resource Manager template samples to deploy Azure Container Instances in different configurations
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
ms.custom: devx-track-arm-template
services: container-instances
ms.topic: sample
ms.date: 11/17/2025
# Customer intent: As a cloud engineer, I want to access sample Resource Manager templates for deploying container instances, so that I can efficiently create various configurations tailored to my project requirements.
---

# Azure Resource Manager templates for Azure Container Instances

The following sample templates deploy container instances in various configurations.

For deployment options, see the [Deployment](#deployment) section. If you'd like to create your own templates, the Azure Container Instances [Resource Manager template reference][ref] details template format and available properties.

## Sample templates

[!INCLUDE [network profile callout](./includes/network-profile-callout.md)]

| Template | Description |
|-|-|
| **Applications** ||
| [WordPress][app-wp] | Creates a WordPress website and its MySQL database in a container group. The WordPress site content and MySQL database are persisted to an Azure Files share. Also creates an application gateway to expose public network access to WordPress. |
| [MS NAV with SQL Server and IIS][app-nav] | Deploys a single Windows container with a fully featured self-contained Dynamics NAV / Dynamics 365 Business Central environment. |
| **Volumes** ||
| [emptyDir][vol-emptydir] | Deploys two Linux containers that share an emptyDir volume. |
| [gitRepo][vol-gitrepo] | Deploys a Linux container that clones a GitHub repo and mounts it as a volume. |
| [secret][vol-secret] | Deploys a Linux container with a PFX cert mounted as a secret volume. |
| **Networking** ||
| [UDP-exposed container][net-udp] | Deploys a Windows or Linux container that exposes a UDP port. |
| [Linux container with public IP][net-publicip] | Deploys a single Linux container accessible via a public IP. |
| [Deploy a container group with a virtual network][net-vnet] | Deploys a new virtual network, subnet, network profile, and container group. |
| **Azure resources** ||
| [Create Azure Storage account and Files share][az-files] | Uses the Azure CLI in a container instance to create a storage account and an Azure Files share.

## Deployment

You have several options for deploying resources with Resource Manager templates:

[Azure CLI][deploy-cli]

[Azure PowerShell][deploy-powershell]

[Azure portal][deploy-portal]

[REST API][deploy-rest]

<!-- LINKS - External -->
[app-nav]: https://github.com/Azure/azure-quickstart-templates/tree/master/demos/
[app-wp]: https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/wordpress/aci-wordpress
[az-files]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-storage-file-share
[net-publicip]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-public-ip
[net-udp]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-udp
[net-vnet]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-vnet
[repo]: https://github.com/Azure/azure-quickstart-templates
[vol-emptydir]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-emptydir
[vol-gitrepo]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-gitrepo
[vol-secret]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-secret

<!-- LINKS - Internal -->
[deploy-cli]: /azure/azure-resource-manager/templates/deploy-cli
[deploy-portal]: /azure/azure-resource-manager/templates/deploy-portal
[deploy-powershell]: /azure/azure-resource-manager/templates/deploy-powershell
[deploy-rest]: /azure/azure-resource-manager/templates/deploy-rest
[ref]: /azure/templates/microsoft.containerinstance/containergroups
