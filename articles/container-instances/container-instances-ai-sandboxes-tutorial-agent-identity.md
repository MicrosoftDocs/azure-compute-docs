---
title: "Tutorial: Access Azure resources from a serverless sandbox with an agent identity (Preview)"
description: In this tutorial, you configure a Microsoft Entra Agent ID on a serverless sandbox in Azure Container Instances so that code inside the sandbox can authenticate to Azure Blob Storage without secrets.
ms.topic: tutorial
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/13/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want to assign a dedicated Microsoft Entra Agent ID to each serverless sandbox, so that code in the sandbox can access only the Azure resources authorized for that agent without storing credentials.
---

# Tutorial: Access Azure resources from a serverless sandbox with an agent identity (preview)

By using agent identity, you can assign a [Microsoft Entra Agent ID](/entra/agent-id/) to an individual [serverless sandbox in Azure Container Instances](container-instances-ai-sandboxes.md) so that code running in the sandbox can acquire Microsoft Entra tokens and access Azure resources without storing secrets in the sandbox. Sandboxes created without an agent identity can't acquire tokens from the instance metadata endpoint, even when they run in the same AI agents group.

In this tutorial, you configure an AI agents group with a user-assigned managed identity for federated token exchange, grant an agent identity access to an Azure Blob Storage container, verify that a sandbox without an agent identity is denied access, and then create a sandbox with the agent identity that writes, reads, and deletes a blob.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Configure a user-assigned managed identity and federated identity credential for an agent identity blueprint
> * Configure `identityProfile.federatedIdentityCredential` on an AI agents group
> * Verify that a sandbox without an agent identity can't acquire tokens
> * Create a sandbox with `agentIdentities` and access Azure Blob Storage from inside the sandbox

## Prerequisites

[!INCLUDE [container-instances-ai-sandboxes-prerequisites](./includes/container-instances-ai-sandboxes-prerequisites.md)]

* Permissions in Microsoft Entra ID to [create an agent identity blueprint](/entra/agent-id/create-blueprint) and an agent identity.

## Prepare the managed identity, agent identity, and storage account

1. Create a resource group and a user-assigned managed identity. Record its resource ID, principal ID, and tenant ID.

    ```azurecli
    az group create --name myResourceGroup --location centralus

    az identity create --resource-group myResourceGroup --name myGroupIdentity --location centralus

    identityResourceId=$(az identity show --resource-group myResourceGroup --name myGroupIdentity --query id --output tsv)
    identityPrincipalId=$(az identity show --resource-group myResourceGroup --name myGroupIdentity --query principalId --output tsv)
    tenantId=$(az account show --query tenantId --output tsv)
    ```

1. In Microsoft Entra ID, [create an agent identity blueprint](/entra/agent-id/create-blueprint) and configure a federated identity credential on the blueprint that trusts the user-assigned managed identity of the AI agents group:

    * **Issuer:** `https://login.microsoftonline.com/<tenant-id>/v2.0`
    * **Subject identifier:** `<identityPrincipalId>` (the `principalId` of `myGroupIdentity`)
    * **Audience:** `api://AzureADTokenExchange`

    Then create an agent identity from the blueprint. Record the blueprint's application (client) ID (`<blueprint-client-id>`) and the agent identity's application (client) ID (`<agent-client-id>`) and object ID (`<agent-object-id>`).

1. Create an Azure Storage account and a blob container. Assign the **Storage Blob Data Contributor** role on the storage account to the **agent identity** (not to `myGroupIdentity`).

    ```azurecli
    storageAccount="aisandboxstore$RANDOM"

    az storage account create --resource-group myResourceGroup --name $storageAccount --location centralus --sku Standard_LRS
    az storage container create --account-name $storageAccount --name agentdemo --auth-mode login

    storageAccountId=$(az storage account show --resource-group myResourceGroup --name $storageAccount --query id --output tsv)

    az role assignment create --assignee-object-id <agent-object-id> --assignee-principal-type ServicePrincipal \
      --role "Storage Blob Data Contributor" --scope $storageAccountId
    ```

## Configure the AI agents group for federated identity credentials

1. Create or update the AI agents group with the user-assigned managed identity attached in `identity.userAssignedIdentities` and its `principalId` set in `properties.identityProfile.federatedIdentityCredential`.

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myGroupIdentity": {}
        }
      },
      "properties": {
        "identityProfile": {
          "federatedIdentityCredential": "<identity-principal-id>"
        }
      }
    }
    ```

1. Check the AI agents group until `properties.provisioningState` is `Succeeded`, and then call `connect` to get its `{endpoint}` and `{accessToken}`:

    ```http
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview
    ```

    ```http
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup/connect?api-version=2026-08-01-preview
    ```

## Verify that a sandbox without an agent identity is denied

1. Create a sandbox in the group without setting `properties.agentIdentities`:

    ```http
    PUT {endpoint}/sandboxes/sandbox-no-id?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "image": "mcr.microsoft.com/mirror/docker/library/nginx:1.25",
        "resources": {
          "cpu": 1,
          "memory": 2
        }
      }
    }
    ```

    Wait until `properties.provisioningState` is `Succeeded`, copy `properties.connectionString`, and wait until the guest is ready (`WaitForReady`). By using the [sample C# CLI](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto):

    ```bash
    dotnet run -- -cs "<sandbox-no-id-connection-string>" wait
    ```

1. From inside `sandbox-no-id`, attempt to request an access token for Azure Storage from the instance metadata endpoint (`http://169.254.169.254/metadata/identity/oauth2/token`). Use the sample C# CLI:

    ```bash
    dotnet run -- -cs "<sandbox-no-id-connection-string>" exec /bin/sh -c '
    resp=$(curl -s -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/")
    token=$(printf "%s" "$resp" | sed -n "s/.*\"access_token\" *: *\"\([^\"]*\)\".*/\1/p")
    if [ -z "$token" ]; then echo "DENIED: no identity token available for this sandbox"; fi'
    ```

    Because no agent identity is assigned to `sandbox-no-id`, no `access_token` is returned:

    ```output
    DENIED: no identity token available for this sandbox
    ```

## Create a sandbox with an agent identity and access Azure Blob Storage

1. Create a sandbox with `properties.agentIdentities` set to the `agentId` (`<agent-client-id>`) and `agentBlueprintId` (`<blueprint-client-id>`):

    ```http
    PUT {endpoint}/sandboxes/sandbox-with-id?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "image": "mcr.microsoft.com/mirror/docker/library/nginx:1.25",
        "resources": {
          "cpu": 1,
          "memory": 2
        },
        "agentIdentities": [
          {
            "agentId": "<agent-client-id>",
            "agentBlueprintId": "<blueprint-client-id>"
          }
        ]
      }
    }
    ```

    Wait until `properties.provisioningState` is `Succeeded`, copy `properties.connectionString`, and wait until the guest is ready (`WaitForReady`). By using the sample C# CLI:

    ```bash
    dotnet run -- -cs "<sandbox-with-id-connection-string>" wait
    ```

1. Inside `sandbox-with-id`, request a token from `http://169.254.169.254/metadata/identity/oauth2/token` (or use `DefaultAzureCredential` in the Azure SDKs) and write, read, and delete a blob in the storage container. Replace `<storage-account-name>` with your storage account name. Use the sample C# CLI:

    ```bash
    dotnet run -- -cs "<sandbox-with-id-connection-string>" exec /bin/sh -c '
    resp=$(curl -s -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/")
    token=$(printf "%s" "$resp" | sed -n "s/.*\"access_token\" *: *\"\([^\"]*\)\".*/\1/p")
    blob="https://<storage-account-name>.blob.core.windows.net/agentdemo/agent-demo.txt"
    put=$(curl -s -o /dev/null -w "%{http_code}" -X PUT -H "Authorization: Bearer $token" -H "x-ms-version: 2021-08-06" -H "x-ms-blob-type: BlockBlob" -H "Content-Type: text/plain" --data "hello from the agent-identity sandbox" "$blob")
    echo "write  (PUT)    : HTTP $put"
    get=$(curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer $token" -H "x-ms-version: 2021-08-06" "$blob")
    echo "read   (GET)    : HTTP $get"
    del=$(curl -s -o /dev/null -w "%{http_code}" -X DELETE -H "Authorization: Bearer $token" -H "x-ms-version: 2021-08-06" "$blob")
    echo "delete (DELETE) : HTTP $del"'
    ```

    The token issued to the sandbox carries the `appid` and `oid` claims of the agent identity, and all three storage operations succeed:

    ```output
    write  (PUT)    : HTTP 201
    read   (GET)    : HTTP 200
    delete (DELETE) : HTTP 202
    ```

1. Decode the access token payload inside `sandbox-with-id` to confirm that the token belongs to the agent identity (`<agent-client-id>`) rather than to the user-assigned managed identity (`myGroupIdentity`). Use the sample C# CLI:

    ```bash
    dotnet run -- -cs "<sandbox-with-id-connection-string>" exec /bin/sh -c '
    resp=$(curl -s -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/")
    token=$(printf "%s" "$resp" | sed -n "s/.*\"access_token\" *: *\"\([^\"]*\)\".*/\1/p")
    payload=$(printf "%s" "$token" | cut -d. -f2 | tr "_-" "/+")
    case $(( ${#payload} % 4 )) in 2) payload="${payload}==";; 3) payload="${payload}=";; esac
    printf "%s" "$payload" | base64 -d 2>/dev/null | tr "," "\n" | grep -E "\"(appid|oid|azp|sub|xms_act_fct)\""'
    ```

    The decoded claims show `appid` set to `<agent-client-id>` (the agent identity that was granted access to the storage account) and `xms_act_fct` indicating the acting agent:

    ```output
    "appid":"<agent-client-id>"
    "oid":"<agent-object-id>"
    "sub":"<agent-object-id>"
    "xms_act_fct":"11 3"
    ```

## Clean up resources

Delete the sandboxes that you created in this tutorial:

```http
DELETE {endpoint}/sandboxes/sandbox-no-id?api-version=2026-08-01-preview
Authorization: Bearer {accessToken}
```

```http
DELETE {endpoint}/sandboxes/sandbox-with-id?api-version=2026-08-01-preview
Authorization: Bearer {accessToken}
```

If you don't plan to do the other tutorials, delete the AI agents group and the resource group.

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Run sensitive workloads in a serverless sandbox enclave](container-instances-ai-sandboxes-tutorial-enclave.md)
