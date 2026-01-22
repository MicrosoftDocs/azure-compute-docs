---
title: 'Quickstart: Use a Bicep file to create a Windows VM'
description: In this quickstart, you learn how to use a Bicep file to create a Windows virtual machine
author: schaffererin
ms.author: schaffererin
ms.service: azure-virtual-machines
ms.collection: windows
ms.topic: quickstart
ms.date: 03/11/2022
ms.custom: subject-armqs, mode-arm, devx-track-bicep
# Customer intent: As a cloud engineer, I want to deploy a Windows virtual machine using a Bicep file, so that I can automate the setup and configuration of my virtual infrastructure in Azure.
---

# Quickstart: Create a Windows virtual machine using a Bicep file

**Applies to:** :heavy_check_mark: Windows VMs

This quickstart shows you how to use a Bicep file to deploy a Windows virtual machine (VM) in Azure.  Availability of [VM sizes](/azure/virtual-machines/sizes/overview), [images](/azure/virtual-machines/windows/cli-ps-findimage), and features can vary by Azure region and subscription SKU.

[!INCLUDE [About Bicep](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-bicep-introduction.md)]

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
* Install [Bicep tools](/azure/azure-resource-manager/bicep/install)
* Install [Azure CLI](/cli/azure/install-azure-cli) (version 2.20.0 or later) or [Azure PowerShell](/powershell/azure/install-azure-powershell) (Az module 5.6.0 or later).
* Signed in to Azure with the correct subscription selected. Use Azure CLI's [az login](/cli/azure/authenticate-azure-cli-interactively) command, or Azure PowerShell's [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

## Review the Bicep file

The Bicep file used in this quickstart is from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/vm-simple-windows/). The template relies on the following parameters:

| Parameter | Default value | Allowed values | Constraints |
|-|-|-|-|
| `adminUsername` | *No default (required)* | Any valid Windows admin username | Required. Cannot use reserved usernames such as `admin` or `administrator`. |
| `vmSize` | `Standard_DS1_v2` | Region-dependent [Azure VM sizes](/azure/virtual-machines/sizes/overview) | Size availability depends on the selected region and subscription quota. |
| `location` | Resource group location | Any Azure region | Some VM sizes and images are not available in all regions. |
| `windowsOSVersion` | `2019-Datacenter` | Supported Windows [Server images](/azure/virtual-machines/windows/cli-ps-findimage)  in Azure Marketplace | Image availability varies by region. |

:::code language="bicep" source="~/quickstart-templates/quickstarts/microsoft.compute/vm-simple-windows/main.bicep":::

Several resources are defined in the Bicep file:

- [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/Microsoft.Network/virtualNetworks/subnets): create a subnet.
- [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts): create a storage account.
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/Microsoft.Network/publicIPAddresses): create a public IP address.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/Microsoft.Network/networkSecurityGroups): create a network security group.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/Microsoft.Network/virtualNetworks): create a virtual network.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/Microsoft.Network/networkInterfaces): create a NIC.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/Microsoft.Compute/virtualMachines): create a virtual machine.

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file using either Azure CLI or Azure PowerShell.

    # [CLI](#tab/CLI)

    ```azurecli
    exampleRG=myResourceGroupName
    adminUserName=myAdminUserName

    az group create --name exampleRG --location eastus
    az deployment group create --resource-group exampleRG --template-file main.bicep --parameters adminUserName=<admin-username>
    ```

    # [Azure PowerShell](#tab/azurepowershell)

    ```azurepowershell
    $exampleRG = myResourceGroupName
    $adminUserName = myAdminUserName

    New-AzResourceGroup -Name exampleRG -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep -adminUserName "<admin-username>"
    ```

    ---

  > [!NOTE]
  > Replace **\<adminUserName\>** with a unique username. You'll also be prompted to enter adminPassword. The minimum password length is 12 characters.

  When the deployment finishes, you should see a messaged indicating the deployment succeeded. Use Azure CLI to verify that the virtual machine was created successfully by checking that the deployment provisioning state is `Succeeded`.

  ```azurecli
   az deployment group show --resource-group exampleRG --name main --query properties.provisioningState
  ```

Cost information isn't presented during the virtual machine creation process for Bicep like it is for the [Azure portal](quick-create-portal.md). If you want to learn more about how cost works for virtual machines, see the [Cost optimization Overview page](../cost-optimization-plan-to-manage-costs.md).

## Review deployed resources

Use the Azure portal, Azure CLI, or Azure PowerShell to list the deployed resources in the resource group.

# [CLI](#tab/CLI)

```azurecli-interactive
az resource list --resource-group exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzResource -ResourceGroupName exampleRG
```

---

## Clean up resources

When no longer needed, use the Azure portal, Azure CLI, or Azure PowerShell to delete the VM and all of the resources in the resource group.

# [CLI](#tab/CLI)

```azurecli-interactive
az group delete --name exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name exampleRG
```

---

## Next steps

In this quickstart, you deployed a simple virtual machine using a Bicep file. To learn more about Azure virtual machines, continue to the tutorial for Linux VMs.

> [!div class="nextstepaction"]
> [Azure Windows virtual machine tutorials](./tutorial-manage-vm.md)
