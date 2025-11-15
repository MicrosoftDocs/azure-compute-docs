---
title: Enable Managed Identity in a Container Group
description: Learn how to enable a managed identity in Azure Container Instances that can authenticate with other Azure services.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
ms.custom: devx-track-azurecli
services: container-instances
ms.date: 08/29/2024
# Customer intent: "As a cloud developer, I want to enable managed identities in Azure Container Instances so that I can authenticate to other Azure services without handling credentials in my code."
---

# Use managed identities with Azure Container Instances

Use [managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) to run code in Azure Container Instances that interacts with other Azure services. You don't have to maintain any secrets or credentials in code. The feature provides a Container Instances deployment with an automatically managed identity in Microsoft Entra ID.

In this article, you learn more about managed identities in Container Instances. You also:

> [!div class="checklist"]
> * Enable a user-assigned or system-assigned identity in a container group.
> * Grant the identity access to an Azure key vault.
> * Use the managed identity to access a key vault from a running container.

Adapt the examples to enable and use identities in Container Instances to access other Azure services. These examples are interactive. In practice, your container images would run code to access Azure services.

## Why use a managed identity?

Use a managed identity in a running container to authenticate to any [service that supports Microsoft Entra authentication](/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities#azure-services-that-support-azure-ad-authentication) without managing credentials in your container code. For services that don't support Microsoft Entra authentication, you can store secrets in an Azure key vault and use the managed identity to access the key vault to retrieve credentials. For more information about using a managed identity, see [What are managed identities for Azure resources?](/azure/active-directory/managed-identities-azure-resources/overview).

### Enable a managed identity

 When you create a container group, enable one or more managed identities by setting a [ContainerGroupIdentity](/rest/api/container-instances/2022-09-01/container-groups/create-or-update#containergroupidentity) property. You can also enable or update managed identities after a container group is running. Either action causes the container group to restart. To set the identities on a new or existing container group, use the Azure CLI, an Azure Resource Manager template, a YAML file, or another Azure tool.

Container Instances supports both types of managed Azure identities: user-assigned and system-assigned. On a container group, you can enable a system-assigned identity, one or more user-assigned identities, or both types of identities. If you're unfamiliar with managed identities for Azure resources, see the [overview](/azure/active-directory/managed-identities-azure-resources/overview).

### Use a managed identity

To use a managed identity, the identity must be granted access to one or more Azure service resources (such as a web app, a key vault, or a storage account) in the subscription. Using a managed identity in a running container is similar to using an identity in an Azure virtual machine (VM). For more information, see the VM guidance for using a [token](/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token), [Azure PowerShell or the Azure CLI](/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-sign-in), or the [Azure SDKs](/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-sdk).

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0.49 or later of the Azure CLI. If you use Azure Cloud Shell, the latest version is already installed.

## Create an Azure key vault

The examples in this article use a managed identity in Container Instances to access an Azure Key Vault secret.

First, create a resource group named *myResourceGroup* in the *eastus* location with the following [az group create](/cli/azure/group#az-group-create) command:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Use the [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command to create a key vault. Be sure to specify a unique key vault name.

```azurecli-interactive
az keyvault create \
  --name mykeyvault \
  --resource-group myResourceGroup \
  --location eastus
```

Store a sample secret in the key vault by using the [az keyvault secret set](/cli/azure/keyvault/secret#az-keyvault-secret-set) command:

```azurecli-interactive
az keyvault secret set \
  --name SampleSecret \
  --value "Hello Container Instances" \
  --description ACIsecret --vault-name mykeyvault
```

Continue with the following examples to access the key vault by using either a user-assigned or system-assigned managed identity in Container Instances.

## Example 1: Use a user-assigned identity to access the Azure key vault

### Create an identity

First create an identity in your subscription by using the [az identity create](/cli/azure/identity#az-identity-create) command. You can use the same resource group that was used to create the key vault. You can also use a different one.

```azurecli-interactive
az identity create \
  --resource-group myResourceGroup \
  --name myACIId
```

To use the identity in the following steps, use the [az identity show](/cli/azure/identity#az-identity-show) command to store the identity's service principal ID and resource ID in variables.

```azurecli-interactive
# Get service principal ID of the user-assigned identity
SP_ID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query principalId --output tsv)

# Get resource ID of the user-assigned identity
RESOURCE_ID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query id --output tsv)
```

### Grant user-assigned identity access to the key vault

Run the following [az keyvault set-policy](/cli/azure/keyvault) command to set an access policy on the key vault. The following example allows the user-assigned identity to get secrets from the key vault:

```azurecli-interactive
 az keyvault set-policy \
    --name mykeyvault \
    --resource-group myResourceGroup \
    --object-id $SP_ID \
    --secret-permissions get
```

### Enable user-assigned identity on a container group

Run the following [az container create](/cli/azure/container#az-container-create) command to create a container instance based on Microsoft's `azure-cli` image. This example provides a single container group that you can use interactively to run the Azure CLI to access other Azure services. In this section, only the base operating system is used. For an example to use the Azure CLI in the container, see [Enable system-assigned identity on a container group](#enable-system-assigned-identity-on-a-container-group).

The `--assign-identity` parameter passes your user-assigned managed identity to the group. The long-running command keeps the container running. This example uses the same resource group that was used to create the key vault, but you could specify a different one.

```azurecli-interactive
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity $RESOURCE_ID \
  --command-line "tail -f /dev/null"
```

Within a few seconds, you should get a response from the Azure CLI that indicates that the deployment finished. Check its status with the [az container show](/cli/azure/container#az-container-show) command.

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

The `identity` section in the output looks similar to the following example, which shows that the identity is set in the container group. The `principalID` under `userAssignedIdentities` is the service principal of the identity that you created in Microsoft Entra ID:

```output
[...]
"identity": {
    "principalId": "null",
    "tenantId": "aaaabbbb-0000-cccc-1111-dddd2222eeee",
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourcegroups/danlep1018/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACIId": {
        "clientId": "00001111-aaaa-2222-bbbb-3333cccc4444",
        "principalId": "aaaaaaaa-bbbb-cccc-1111-222222222222"
      }
    }
  },
[...]
```

### Use user-assigned identity to get a secret from the key vault

Now you can use the managed identity within the running container instance to access the key vault. First, open a Bash shell in the container:

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Run the following commands in the Bash shell in the container. To get an access token to use Microsoft Entra ID to authenticate to the key vault, run the following command:

```bash
client_id="00001111-aaaa-2222-bbbb-3333cccc4444"
curl "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net&client_id=$client_id" -H Metadata:true -s
```

Output:

```bash
{"access_token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9......xxxxxxxxxxxxxxxxx","refresh_token":"","expires_in":"28799","expires_on":"1539927532","not_before":"1539898432","resource":"https://vault.azure.net/","token_type":"Bearer"}
```

To store the access token in a variable to use in subsequent commands to authenticate, run the following command:

```bash
TOKEN=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true | jq -r '.access_token')

```

Now use the access token to authenticate to the key vault and read a secret. Be sure to substitute the name of your key vault in the URL (*https:\//mykeyvault.vault.azure.net/...*):

```bash
curl https://mykeyvault.vault.azure.net/secrets/SampleSecret/?api-version=7.4 -H "Authorization: Bearer $TOKEN"
```

The response looks similar to the following example that shows the secret. In your code, you parse this output to obtain the secret. Then, use the secret in a subsequent operation to access another Azure resource.

```bash
{"value":"Hello Container Instances","contentType":"ACIsecret","id":"https://mykeyvault.vault.azure.net/secrets/SampleSecret/xxxxxxxxxxxxxxxxxxxx","attributes":{"enabled":true,"created":1539965967,"updated":1539965967,"recoveryLevel":"Purgeable"},"tags":{"file-encoding":"utf-8"}}
```

## Example 2: Use a system-assigned identity to access an Azure key vault

### Enable system-assigned identity on a container group

Run the following [az container create](/cli/azure/container#az-container-create) command to create a container instance based on Microsoft's `azure-cli` image. This example provides a single container group that you can use interactively to run the Azure CLI to access other Azure services.

The `--assign-identity` parameter with no other value enables a system-assigned managed identity on the group. The identity is scoped to the resource group of the container group. The long-running command keeps the container running. This example uses the same resource group that was used to create the key vault, which is in the scope of the identity.

```azurecli-interactive
# Get the resource ID of the resource group
RG_ID=$(az group show --name myResourceGroup --query id --output tsv)

# Create container group with system-managed identity
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity --scope $RG_ID \
  --command-line "tail -f /dev/null"
```

Within a few seconds, you should get a response from the Azure CLI indicating that the deployment is finished. Check its status with the [az container show](/cli/azure/container#az-container-show) command.

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

The `identity` section in the output looks similar to the following example, which shows that a system-assigned identity is created in Microsoft Entra ID:

```output
[...]
"identity": {
    "principalId": "bbbbbbbb-cccc-dddd-2222-333333333333",
    "tenantId": "aaaabbbb-0000-cccc-1111-dddd2222eeee",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
},
[...]
```

Set a variable to the value of `principalId` (the service principal ID) of the identity to use in later steps.

```azurecli-interactive
SP_ID=$(az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query identity.principalId --out tsv)
```

### Grant container group access to the key vault

Run the following [az keyvault set-policy](/cli/azure/keyvault) command to set an access policy on the key vault. The following example allows the system-managed identity to get secrets from the key vault:

```azurecli-interactive
 az keyvault set-policy \
   --name mykeyvault \
   --resource-group myResourceGroup \
   --object-id $SP_ID \
   --secret-permissions get
```

### Use container group identity to get a secret from key vault

Now you can use the managed identity to access the key vault within the running container instance. First, open a Bash shell in the container:

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Run the following commands in the Bash shell in the container. First, sign in to the Azure CLI by using the managed identity:

```azurecli-interactive
az login --identity
```

From the running container, retrieve the secret from the key vault:

```azurecli-interactive
az keyvault secret show \
  --name SampleSecret \
  --vault-name mykeyvault --query value
```

The value of the secret is retrieved:

```output
"Hello Container Instances"
```

## Enable managed identity by using a Resource Manager template

To enable a managed identity in a container group by using a [Resource Manager template](container-instances-multi-container-group.md), set the `identity` property of the `Microsoft.ContainerInstance/containerGroups` object with a `ContainerGroupIdentity` object. The following snippets show the `identity` property configured for different scenarios. For more information, see the [Resource Manager template reference](/azure/templates/microsoft.containerinstance/containergroups). Specify a minimum `apiVersion` of `2018-10-01`.

### User-assigned identity

A user-assigned identity is a resource ID of the following form:

```
"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}"
```

You can enable one or more user-assigned identities.

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
```

### System-assigned identity

```json
"identity": {
    "type": "SystemAssigned"
    }
```

### System-assigned and user-assigned identities

On a container group, you can enable both a system-assigned identity and one or more user-assigned identities.

```json
"identity": {
    "type": "SystemAssigned, UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
...
```

## Enable managed identity by using a YAML file

To enable a managed identity in a container group deployed by using a [YAML file](container-instances-multi-container-yaml.md), include the following YAML.
Specify a minimum `apiVersion` of `2018-10-01`.

### User-assigned identity

A user-assigned identity is a resource ID of the following form:

```
'/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}'
```

You can enable one or more user-assigned identities.

```yaml
identity:
  type: UserAssigned
  userAssignedIdentities:
    {'myResourceID1':{}}
```

### System-assigned identity

```yaml
identity:
  type: SystemAssigned
```

### System-assigned and user-assigned identities

On a container group, you can enable both a system-assigned identity and one or more user-assigned identities.

```yml
identity:
  type: SystemAssigned, UserAssigned
  userAssignedIdentities:
   {'myResourceID1':{}}
```

## Managed identity on Windows containers

Managed identity on Windows container groups works differently than Linux container groups. For Windows containers, metadata server (169.254.169.254) isn't available for getting the Microsoft Entra ID token. Customers can follow a different pattern to get the access token in Windows containers. The pattern involves sending a token request to `IDENTITY_ENDPOINT` along with other information, such as the principal ID and the secret. The `IDENTITY_ENDPOINT` and `IDENTITY_HEADER` are injected as environmental variables in your container.

```console
curl -G -v %IDENTITY_ENDPOINT% --data-urlencode resource=https://vault.azure.net --data-urlencode principalId=<principal id> -H secret:%IDENTITY_HEADER%
```

A sample Azure PowerShell script:

```powershell
identityEndpoint = $env:IDENTITY_ENDPOINT
$identityHeader = $env:IDENTITY_HEADER
$resource = "https://vault.azure.net"
$principalId = "aaaaaaaa-bbbb-cccc-1111-222222222222"
 
Invoke-RestMethod -Uri "$identityEndpoint" `
    -Method Get `
    -Headers @{secret = $identityHeader} `
    -Body @{resource = $resource; principalId = $principalId} `
    -ContentType "application/x-www-form-urlencoded"
```

The `az login` module and other client libraries that depend on the metadata server (169.254.169.254) don't work in a Windows container. Windows containers in a virtual network can't connect to the endpoint. As a result, a managed identity token can't be generated in a Windows virtual network container.

## Related content

In this article, you learned about managed identities in Container Instances and how to:

> [!div class="checklist"]
> * Enable a user-assigned or system-assigned identity in a container group.
> * Grant the identity access to an Azure key vault.
> * Use the managed identity to access a key vault from a running container.

* Learn more about [managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/).
* See an [Azure Go SDK example](https://medium.com/@samkreter/c98911206328) of using a managed identity to access a key vault from Container Instances.
