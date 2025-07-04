---
title: Scaling profile for Virtual Machine Scale Sets
description: The virtual machine scaling profile for Virtual Machine Scale Sets define the VM configuration you want to use when adding instances to the scale set via autoscaling 
author: fitzgeraldsteele 
ms.author: fisteele
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.date: 06/14/2024
ms.reviewer: jushiman
# Customer intent: "As a cloud administrator, I want to configure a Virtual Machine Scale Set with specific properties, so that I can efficiently manage instance scaling and deployment according to my application's requirements."
---

# Scaling profile

Virtual machine scale sets allow you to define a scaling profile or template, which specifies the properties of virtual machine instances. Examples of properties set in the virtual machine scaling profile include:

- VM Image
- Admin credentials
- Network interface settings
- Load balancer backend pool
- OS and Data disk settings

When you increase the capacity or instance count of the scale set, the scale set will add new virtual machines to the set based on the configuration defined in the profile. Scale sets with scaling profile are also eligible for orchestrations such as [reimaging](/rest/api/compute/virtual-machine-scale-sets/reimage), [rolling upgrades](/rest/api/compute/virtual-machine-scale-set-rolling-upgrades), [instance repair](virtual-machine-scale-sets-automatic-instance-repairs.md), and [automatic OS updates](virtual-machine-scale-sets-automatic-upgrade.md).

> [!IMPORTANT] 
> Virtual machine scaling profile settings are required for scale sets in Uniform Orchestration Mode, and optional for scale sets in Flexible Orchestration Mode.

## Create a scale set with a scaling profile
By default, scale sets are created with a virtual machine scaling profile. See [Create an Azure Virtual Machine](quick-create-portal.md) and [Create and manage Azure Virtual Machines](tutorial-create-and-manage-cli.md) for examples.

## Create a scale set without a scaling profile

> [!IMPORTANT]
>Starting November 2023, VM scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](
https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

Virtual machine scale sets in Flexible Orchestration Mode can optionally be created without a virtual machine scaling profile. This configuration is similar to creating and deploying an Availability Set in that you add to the set by manually creating virtual machine instances and adding them to the set. It's useful to create a scale set without a scaling profile when:

- You need complete control over all VM properties.
- You need to follow your own VM naming conventions.
- You want to add different types of VMs to the same scale set.
- You need to control the placement of virtual machines into a specific availability zone or fault domain.

| Feature                             | Virtual machine scale sets (no scaling profile) | Availability Sets |
| ----------------------------------- | :---------------------------------------------: | :---------------: |
| Maximum capacity                    |                      1000                       |        200        |
| Supports Availability Zones         |                       Yes                       |        No         |
| Maximum Aligned Fault Domains Count |                        3                        |         3         |
| Add new VM to set                   |                       Yes                       |        Yes        |
| Add VM to specific fault domain     |                       Yes                       |        No         |
| Maximum Update Domain count         |       N/A. Update domains are deprecated        |        20         |

Once you have created the virtual machine scale set, you can manually attach virtual machines.

### [Azure portal](#tab/portal)
1. Navigate to the **Create a Virtual Machine Scale Set (VMSS)** blade.
2. Set your resource group, scale set name, region, availability zone, and orchestration mode.
3. In the **Scaling** section, select **No scaling profile**.
4. Complete the **Advanced** and **Tags** section before selecting the **Review + create** button.

### [Azure CLI](#tab/cli)

By default, the Azure CLI will create a scale set with a scaling profile. Omit the scaling profile parameters to create a virtual machine scale set with no scaling profile.

```azurecli-interactive
az group create
 --name myResourceGroup
 --location westus3
az vmss create \
	--name myVmss \
	--resource-group myResourceGroup \
        --orchestration-mode flexible \
	--platform-fault-domain-count 3 
```

### [Azure PowerShell](#tab/powershell)

```azurepowershell-interactive
New-AzResourceGroup
 -Name myResourceGroup
 -Location westus3
$vmssConfig = New-AzVmssConfig 
	-Location 'westus3' 
	-PlatformFaultDomainCount 3 
$vmss = New-AzVmss `
	-ResourceGroupName 'myResourceGroup' `
	-Name 'myVmss' `
	-VirtualMachineScaleSet $vmssConfig
```

### [ARM template](#tab/arm)

Go to the Azure Quickstart Template [Manually add a VM to an existing Virtual Machine Scale Set](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vmss-flexible-orchestration-manual-add-vm) for a full example.

---


## Next steps
- Create a new Virtual Machine Scale Set with a scale profile using the [Azure portal](quick-create-portal.md).
- If you have created a virtual machine scale set without a scaling profile, you can [manually attach a virtual machine](virtual-machine-scale-sets-attach-detach-vm.md).
