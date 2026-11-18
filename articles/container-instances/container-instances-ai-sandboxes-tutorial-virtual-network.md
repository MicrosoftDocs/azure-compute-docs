---
title: "Tutorial: Connect serverless sandboxes to a virtual network (Public Preview)"
description: In this tutorial, you use the Azure Resource Manager REST API to connect an AI agents group in Azure Container Instances to a delegated subnet, and reach a private web app from a serverless sandbox.
ms.topic: tutorial
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/04/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want my sandboxes to reach private resources in my virtual network, so that agents can use internal services and data without exposing them to the internet.
---

# Tutorial: Connect serverless sandboxes to a virtual network (Public Preview)

You can connect an AI agents group for [serverless sandboxes in Azure Container Instances (ACI)](container-instances-ai-sandboxes.md) to a subnet in your Azure virtual network. Each sandbox in the group gets a private IP address from the subnet, so sandboxes can reach private resources in the virtual network, such as internal services, databases, and private endpoints.

In this tutorial, you deploy a web app with only a private IP address. You then create an AI agents group in the same virtual network, and reach the web app from a sandbox.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Create a virtual network with a delegated subnet for sandboxes
> * Deploy a private web app in the virtual network
> * Create an AI agents group that's connected to the subnet
> * Reach the private web app from a sandbox

## Prerequisites

[!INCLUDE [container-instances-ai-sandboxes-prerequisites](./includes/container-instances-ai-sandboxes-prerequisites.md)]

## Create a virtual network

Create a virtual network with two subnets:

| Subnet | Delegation | Used by |
|---|---|---|
| `sandbox-subnet` | `Microsoft.ContainerInstance/AIAgentsGroups` | The sandboxes. Each sandbox uses one IP address. |
| `app-subnet` | `Microsoft.ContainerInstance/containerGroups` | The private web app in this tutorial. |

```azurecli
az group create --name myResourceGroup --location centralus

az network vnet create --resource-group myResourceGroup --name myVnet --location centralus --address-prefix 10.10.0.0/16

az network vnet subnet create --resource-group myResourceGroup --vnet-name myVnet --name sandbox-subnet \
  --address-prefixes 10.10.0.0/21 --delegations Microsoft.ContainerInstance/AIAgentsGroups

az network vnet subnet create --resource-group myResourceGroup --vnet-name myVnet --name app-subnet \
  --address-prefixes 10.10.8.0/24 --delegations Microsoft.ContainerInstance/containerGroups
```

Size the sandbox subnet for the number of sandboxes that you plan to run at the same time. Each sandbox uses one IP address from the subnet. The `/21` subnet in this example has room for about 2,000 sandboxes.

> [!IMPORTANT]
> For outbound internet access (egress), associate a NAT gateway with the sandbox subnet in your virtual network and set the subnet's default outbound access setting (`defaultOutboundAccess`) to `true`. For NAT gateway configuration guidance, see [Configure container group egress with a NAT gateway](container-instances-nat-gateway.md).

## Deploy a private web app

Deploy a container group in `app-subnet`. The container group has only a private IP address, so it isn't reachable from the internet:

```azurecli
az container create --resource-group myResourceGroup --name private-web \
  --image mcr.microsoft.com/azuredocs/aci-helloworld --vnet myVnet --subnet app-subnet --os-type Linux --cpu 1 --memory 1.5

az container show --resource-group myResourceGroup --name private-web --query ipAddress.ip -o tsv
```

The output is a private IP address from `app-subnet`, such as `10.10.8.4`.

## Create an AI agents group in the virtual network

1. Create the AI agents group with the resource ID of `sandbox-subnet` in `properties.networkProfile.subnets`:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myVnetAgentsGroup?api-version=2026-08-01-preview

    {
      "location": "centralus",
      "properties": {
        "networkProfile": {
          "subnets": [
            {
              "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/sandbox-subnet"
            }
          ]
        }
      }
    }
    ```

1. Check the AI agents group until `properties.provisioningState` is `Succeeded`, and then call `connect` to get its `{endpoint}` and `{accessToken}`:

    ```http
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myVnetAgentsGroup?api-version=2026-08-01-preview
    ```

    ```http
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myVnetAgentsGroup/connect?api-version=2026-08-01-preview
    ```

All the sandboxes in the group use the subnet. You can't change the network profile of an AI agents group after you create it.

## Reach the private web app from a sandbox

1. Create a sandbox in the group. The `azure-cli` image includes `curl`:

    ```http
    PUT {endpoint}/sandboxes/vnet1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "image": "mcr.microsoft.com/azure-cli"
      }
    }
    ```

    Wait until `properties.provisioningState` is `Succeeded`. Then copy `properties.connectionString` from the response, and wait until the guest is ready (`WaitForReady`). By using the [sample C# CLI](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto):

    ```bash
    dotnet run -- -cs "<vnet1-connection-string>" wait
    ```

1. In `vnet1`, request the private web app (`ExecSandbox`). Replace `<private-web-ip>` with the private IP address of `private-web`:

    ```bash
    dotnet run -- -cs "<vnet1-connection-string>" exec curl -s http://<private-web-ip>
    ```

    The response contains the HTML of the web app, which includes this text:

    ```output
    Welcome to Azure Container Instances!
    ```

The sandbox reached the web app through the virtual network. A sandbox in an AI agents group that isn't connected to the virtual network can't reach this private IP address.

> [!NOTE]
> For ACI serverless sandboxes connected to a virtual network, network isolation depends on the delegated subnet. To block or restrict traffic in any direction (inbound or outbound), configure a [network security group (NSG)](/azure/virtual-network/network-security-groups-overview) on the sandbox subnet.

## Clean up resources

1. Delete the AI agents group. Deleting an AI agents group is a long-running operation. It also deletes the sandboxes in the group:

    ```http
    DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myVnetAgentsGroup?api-version=2026-08-01-preview
    ```

1. Delete the web app and the virtual network. Delete the virtual network after the AI agents group deletion finishes. You can't delete a subnet while an AI agents group uses it.

    ```azurecli
    az container delete --resource-group myResourceGroup --name private-web --yes
    az network vnet delete --resource-group myResourceGroup --name myVnet
    ```

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Control sandbox outbound traffic with Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md)
