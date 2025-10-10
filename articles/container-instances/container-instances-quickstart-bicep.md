---
title: Quickstart - create a container instance - Bicep
description: In this quickstart, you use a Bicep file to quickly deploy a containerized web app that runs in an isolated Azure container instance.
author: tomvcassidy
services: container-instances
ms.author: tomcassidy
ms.date: 08/29/2024
ms.topic: quickstart
ms.service: azure-container-instances
ms.custom: subject-armqs, mode-arm, devx-track-bicep
# Customer intent: As a cloud developer, I want to deploy a containerized web application using a Bicep file, so that I can quickly launch isolated container instances without the complexity of full orchestration.
---

# Quickstart: Deploy a container instance in Azure using Bicep

Use Azure Container Instances to run serverless Docker containers in Azure with simplicity and speed. Deploy an application to a container instance on-demand when you don't need a full container orchestration platform like Azure Kubernetes Service. In this quickstart, you use a Bicep file to deploy an isolated Docker container and make its web application available with a public IP address.

[!INCLUDE [About Bicep](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-bicep-introduction.md)]

## Prerequisites

If you don't have an Azure subscription, create a [free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) account before you begin.

## Review the Bicep file

The Bicep file used in this quickstart is from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/aci-linuxcontainer-public-ip/).

The following resource is defined in the Bicep file:

* **[Microsoft.ContainerInstance/containerGroups](/azure/templates/microsoft.containerinstance/containergroups)**: create an Azure container group. This Bicep file defines a group consisting of a single container instance.

More Azure Container Instances template samples can be found in the [quickstart template gallery](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Containerinstance&pageNumber=1&sort=Popular).

To [deploy the container into a specific availability zone](/azure/reliability/reliability-container-instances#availability-zone-support), set the value of the `zone` parameter to the logical availability zone you want to deploy to.

> [!IMPORTANT]
> Zonal deployments are only available in regions that support availability zones. To see if your region supports availability zones, see [Azure Regions List](/azure/reliability/regions-list).

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file using either Azure CLI or Azure PowerShell.

    # [CLI](#tab/CLI)

    ```azurecli
    az group create --name exampleRG --location eastus
    az deployment group create --resource-group exampleRG --template-file main.bicep
    ```

    # [PowerShell](#tab/PowerShell)

    ```azurepowershell
    New-AzResourceGroup -Name exampleRG -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep
    ```

    ---

    When the deployment finishes, you should see a message indicating the deployment succeeded.

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

### View container logs

Viewing the logs for a container instance is helpful when troubleshooting issues with your container or the application it runs. Use the Azure portal, Azure CLI, or Azure PowerShell to view the container's logs.

# [CLI](#tab/CLI)

```azurecli-interactive
az container logs --resource-group exampleRG --name acilinuxpublicipcontainergroup
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzContainerInstanceLog -ResourceGroupName exampleRG -ContainerGroupName acilinuxpublicipcontainergroup -ContainerName acilinuxpublicipcontainergroup
```

---

> [!NOTE]
> It may take a few minutes for the HTTP GET request to generate.

## Clean up resources

When no longer needed, use the Azure portal, Azure CLI, or Azure PowerShell to delete the container and all of the resources in the resource group.

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

In this quickstart, you created an Azure container instance using Bicep. If you'd like to build a container image and deploy it from a private Azure container registry, continue to the Azure Container Instances tutorial.

> [!div class="nextstepaction"]
> [Tutorial: Create a container image for deployment to Azure Container Instances](./container-instances-tutorial-prepare-app.md)
