---
title: Isolated Image Builds for Azure VM Image Builder
description: Isolated Image Builds is achieved by transitioning core process of VM image customization/validation from shared infrastructure to dedicated Azure Container Instances resources in your subscription providing compute and network isolation.
ms.date: 02/13/2024
ms.topic: sample
author: kof-f
ms.author: cynthn
ms.reviewer: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: image-builder
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to implement Isolated Image Builds using Azure Container Instances, so that I can achieve enhanced compute and network isolation while customizing and validating VM images in my subscription.
---

# Isolated Image Builds for Azure VM Image Builder

Isolated Image Builds is a feature of Azure VM Image Builder (AIB). It transitions the core process of VM image customization/validation from shared platform infrastructure to dedicated Azure Container Instances (ACI) resources in your subscription, providing compute and network isolation.

## Advantages of Isolated Image Builds

Isolated Image Builds enables defense-in-depth by limiting network access of your build VM to your subscription. Isolated Image Builds also provides more transparency by allowing you to inspect processing performed by AIB to customize and validate your VM image. Further, Isolated Image Builds simplifies viewing live build logs. Specifically:

1. **Compute Isolation:** Isolated Image Builds performs a major portion of image-building processing in ACI resources in your subscription instead of on AIB's shared platform resources. ACI provides hypervisor isolation for each container group to ensure containers run in isolation without sharing a kernel.
2. **Network Isolation:** Isolated Image Builds removes all direct network WinRM/SSH communication between your build VM and backend components of the AIB service.
    - If you're provisioning an AIB template without your own subnet for build VM, then a Public IP Address resource is no longer provisioned in your staging resource group at image build time.
    - If you're provisioning an AIB template with an existing subnet for build VM, then a Private Link-based communication channel is no longer set up between your build VM and AIB's backend platform resources. Instead, the communication channel is set up between the ACI and the build VM resources, both of which reside in the staging resource group in your subscription.
    - Starting with API version 2024-02-01, you can specify a second subnet for deploying the ACI in addition to the subnet for the build VM. If specified, AIB deploys the ACI on this subnet, and there is no need for AIB to set up the Private Link-based communication channel between the ACI and the build VM. For more information about the second subnet, see [Bring your own build VM subnet and bring your own ACI subnet](./security-isolated-image-builds-image-builder.md#bring-your-own-build-vm-subnet-and-bring-your-own-aci-subnet).

3. **Transparency:** AIB is built on HashiCorp [Packer](https://www.packer.io/). Isolated Image Builds executes Packer in the ACI in your subscription, which allows you to inspect the ACI resource and its containers. Similarly, having the entire network communication pipeline in your subscription allows you to inspect all the network resources, their settings, and their allowances.
4. **Better viewing of live logs:** AIB writes customization logs to a storage account in the staging resource group in your subscription. Isolated Image Builds provides another way to follow the same logs directly in the Azure portal by navigating to AIB's container in the ACI resource.

> [!NOTE]
> To access the live logs during the image build or the customization and validation log files after the build is complete, please refer to the [troubleshooting guide](./linux/image-builder-troubleshoot.md#access-live-logs-during-image-build).

For more information about networking topologies and connectivity behavior, see [Azure VM Image Builder networking options](./linux/image-builder-networking.md).

## Backward compatibility

Isolated Image Builds is a platform-level change and doesn't affect AIB's interfaces. So, your existing Image Template and Trigger resources continue to function and there's no change in the way you deploy new resources of these types. You need to create new templates or update existing templates if you want to use [the network topology that allows you to bring your own ACI subnet](./security-isolated-image-builds-image-builder.md#bring-your-own-build-vm-subnet-and-bring-your-own-aci-subnet).

Your image builds are automatically migrated to Isolated Image Builds and you need to take no action to opt in. Also, customization logs continue to be available in the storage account.

Depending on the network topology specified in the Image Template, you might observe a few new resources temporarily appear in the staging resource group (for example, ACI, Virtual Network, Network Security Group, and Private Endpoint), while some other resources no longer appear (for example, Public IP Address). As before, these temporary resources exist only during the build and AIB deletes them thereafter.

> [!IMPORTANT] 
> Make sure your subscription is registered for `Microsoft.ContainerInstance provider`: 
> - Azure CLI: `az provider register -n Microsoft.ContainerInstance`
> - PowerShell: `Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance`
>
> After successfully registering your subscription, make sure there are no Azure Policies in your subscription that deny deployment of ACI resources. Policies allowing only a restricted set of resource types not including ACI would cause Isolated Image Builds to fail.
>
> Ensure that your subscription also has a sufficient [quota of resources](../container-instances/container-instances-resource-and-quota-limits.md) required for deployment of ACI resources.
>

> [!IMPORTANT]
> Depending on the network topology specified in the Image Template, AIB may need to deploy temporary networking-related resources in the staging resource group in your subscription. Ensure that no Azure Policies deny the deployment of such resources (Virtual Network with Subnets, Network Security Group, Private Endpoint) in the resource group.
>
> If you have Azure Policies applying DDoS protection plans to any newly created Virtual Network, either relax the Policy for the resource group or ensure that the Template Managed Identity has permissions to join the plan. Alternatively, you can use the network topology that does not require deployment of a new Virtual Network by AIB.

> [!IMPORTANT]
> Make sure you follow all [best practices](image-builder-best-practices.md) while using AIB.

> [!NOTE]
> AIB is in the process of rolling this change out to all locations and customers. Some of these details (especially around deployment of new networking-related resources) might change as the process is fine-tuned based on service telemetry and feedback. For any errors, please refer to the [troubleshooting guide](./linux/image-builder-troubleshoot.md#troubleshoot-build-failures).

## Next steps

- [Azure VM Image Builder overview](./image-builder-overview.md)
- [Getting started with Azure Container Instances](../container-instances/container-instances-overview.md)
- [Securing your Azure resources](/azure/security/fundamentals/overview)
- [Troubleshooting guide for Azure VM Image Builder](./linux/image-builder-troubleshoot.md#troubleshoot-build-failures)
