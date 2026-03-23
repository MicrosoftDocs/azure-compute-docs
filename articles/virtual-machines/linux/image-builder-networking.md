---
title: Azure VM Image Builder networking options
description: Understand the networking options available to you when you deploy the Azure VM Image Builder service.
author: kof-f
ms.author: kofiforson
ms.reviewer: jushiman
ms.date: 07/25/2023
ms.topic: concept-article
ms.service: azure-virtual-machines
ms.subservice: image-builder
ms.custom: linux-related-content
# Customer intent: "As a cloud engineer, I want to utilize Azure VM Image Builder with my existing virtual network, so that I can efficiently manage image builds while ensuring secure and private connectivity to my resources."
---

# Azure VM Image Builder networking options

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

Azure VM Image Builder (AIB) deploys an [Azure Container Instance (ACI)](../../container-instances/container-instances-overview.md) in the staging resource group in your subscription. The ACI must be associated with a subnet in an [Azure virtual network (VNet)](/azure/virtual-network/virtual-networks-overview). The AIB service code running in the ACI deploys a build virtual machine (build VM) in the staging resource group to customize your image, and the build VM must be placed on a separate subnet. To customize and validate your image, the ACI must have network connectivity to the build VM. Based on your networking requirements and organizational policies, you can configure AIB to use different network topologies. You can choose to bring both subnets, one subnet, or none. For recommendations, see [Best practices for Azure VM Image Builder](../image-builder-best-practices.md#bring-your-own-subnets).

## Network topologies

### Don't bring your own Build VM subnet or the ACI subnet

- You can select this topology by not specifying the `vnetConfig` field in the Image Template or by specifying the field but without `subnetId` and `containerInstanceSubnetId` subfields.
- If you don't specify any subnets, AIB deploys an Azure virtual network (VNet) in the staging resource group with two subnets: one for the Azure Container Instance and one for the build VM. Both subnets are associated with a [network security group (NSG)](/azure/virtual-network/network-security-groups-overview) that includes default rules, while still allowing direct line-of-sight connectivity required for customization. The build VM is also deployed with a [network interface resource](/azure/virtual-network/virtual-network-network-interface) (and other resources not directly related to networking, like Managed Disk). After the build completes, the build VM and networking resources are deleted.


### Bring your own Build VM subnet and bring your own ACI subnet

- You can select this topology by specifying the `vnetConfig` field with the `subnetId` and `containerInstanceSubnetId` subfields in the image template. This option, including the `containerInstanceSubnetId` subfield, is available starting with API version 2024-02-01. You can also update existing templates to use this topology.
- In this topology, AIB deploys the build VM to the specified build VM subnet and the ACI to the specified ACI subnet. Because the subnets are already provided, AIB does not deploy a virtual network, subnets, or a network security group. This topology is useful when quota restrictions or policies prevent deployment of these resources. The build VM can access resources that are reachable from your virtual network, and you can also create a siloed virtual network that is not connected to any other virtual network. The ACI subnet must satisfy the prerequisites for isolated image builds. For details about these fields, see the [template reference](image-builder-json.md#vnetconfig-optional).
- This topology is the recommended option for most scenarios because it gives you full control over both subnets, simplifies setup and network configuration, can reduce overall deployment costs, and helps align networking with your organization’s security and governance requirements.

### Bring your own build VM subnet but not your own ACI subnet

- You can select this topology by specifying the `vnetConfig` field with the `subnetId` subfield while omitting the `containerInstanceSubnetId` subfield in the image template.
- In this topology, AIB deploys a temporary virtual network in the staging resource group with two subnets, each associated with a network security group (NSG). One subnet hosts the ACI, and the other hosts the private endpoint resource. The build VM is deployed in your specified subnet. To enable communication between the ACI subnet and your build VM subnet, AIB also deploys a Private Link-based communication path in the staging resource group that includes a private endpoint, Private Link Service, Azure Load Balancer, network interfaces, and a proxy virtual machine. The exact resources and configurations vary slightly depending on whether you are customizing a Windows image or a Linux image.
- This topology is generally not recommended for most scenarios because it can increase deployment costs, require more setup and operational configuration, and introduce additional components that can make the end-to-end pipeline more sensitive to failures.

For examples of this topology, see the following articles:
* [Use Azure VM Image Builder for Windows VMs allowing access to an existing Azure virtual network](../windows/image-builder-vnet.md)
* [Use Azure VM Image Builder for Linux VMs allowing access to an existing Azure virtual network](image-builder-vnet.md)

#### What is Azure Private Link?

Azure Private Link provides private connectivity from a virtual network to Azure platform as a service (PaaS), or to customer-owned or Microsoft partner services. It simplifies network architecture and secures connectivity between Azure endpoints by eliminating data exposure to the public internet. Private Link requires an IP address from the specified virtual network and subnet. Azure currently does not support network policies on these IP addresses, so you must disable network policies on the subnet. For more information, see the [Private Link documentation](/azure/private-link/).

#### Why deploy a proxy VM?

When a VM without a public IP is behind an internal load balancer, it does not have internet access. The load balancer used for the virtual network is internal. The proxy VM allows internet access for the build VM during builds, and AIB uses the proxy VM to send commands between the service and the build VM. You can use the associated network security groups to restrict build VM access. Traffic from the ACI goes across the private link to the load balancer. The load balancer communicates with the proxy VM on port 60001 for Linux, or port 60000 for Windows. The proxy forwards commands to the build VM on port 22 for Linux, or port 5986 for Windows. By default, the deployed proxy VM size is *Standard A1_v2*, but you can change the size. For details, see the [template reference](image-builder-json.md#proxyvmsize-optional).

#### Checklist for using your virtual network

1. Allow Azure Load Balancer to communicate with the proxy VM in a network security group.
    * [Azure CLI example](image-builder-vnet.md#add-an-nsg-rule)
    * [PowerShell example](../windows/image-builder-vnet.md#add-an-nsg-rule)
2. Disable the private service policy on the subnet.
    * [Azure CLI example](image-builder-vnet.md#disable-private-service-policy-on-the-subnet)
    * [PowerShell example](../windows/image-builder-vnet.md#disable-private-service-policy-on-the-subnet)
3. Allow VM Image Builder to create a load balancer, and add VMs to the virtual network.
    * [Azure CLI example](image-builder-permissions-cli.md#existing-virtual-network-azure-role-example)
    * [PowerShell example](image-builder-permissions-powershell.md#permission-to-customize-images-on-your-virtual-networks)
4. Allow VM Image Builder to read and write source images, and create images.
    * [Azure CLI example](image-builder-permissions-cli.md#custom-image-azure-role-example)
    * [PowerShell example](image-builder-permissions-powershell.md#custom-image-azure-role-example)
5. Ensure that you're using a virtual network in the same region as the VM Image Builder service region.

## Additional considerations

### Required permissions for an existing virtual network
VM Image Builder requires specific permissions to use an existing virtual network. For more information, see [Configure Azure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure Azure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

> [!NOTE]
> The virtual network must be in the same region as the VM Image Builder service region.
> 

> [!IMPORTANT]
> The Azure VM Image Builder service modifies the WinRM connection configuration on all Windows builds to use HTTPS on port 5986 instead of the default HTTP port on 5985. This configuration change can impact workflows that rely on WinRM communication.

### Connectivity model
AIB does not deploy a public IP for direct connectivity, and the AIB service component that runs in the platform subscription does not have network connectivity to the ACI or the build VM. Only the AIB component that runs in the ACI has connectivity to the build VM to perform customization.

### Supported subnet combinations
You can configure both subnets (`subnetId` and `containerInstanceSubnetId`) or only the build VM subnet (`subnetId`). A configuration that specifies only the ACI subnet (`containerInstanceSubnetId`) is not supported.

## Next steps

[Azure VM Image Builder overview](../image-builder-overview.md)
