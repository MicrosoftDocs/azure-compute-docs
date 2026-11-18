---
title: "Quickstart: Create your first serverless sandbox in Azure Container Instances (Public Preview)"
description: In this quickstart, you use the Azure Resource Manager REST API to create an AI agents group, the sandbox management REST API to create a serverless sandbox in Azure Container Instances, and the sandbox gateway to run a command and copy a file into the sandbox.
ms.topic: quickstart
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/04/2026
ms.update-cycle: 180-days
# Customer intent: As a developer of AI agents, I want to create a serverless sandbox and run a command in it in a few minutes, so that I can evaluate serverless sandboxes in Azure Container Instances for my agents.
---

# Quickstart: Create your first serverless sandbox in Azure Container Instances (Public Preview)

In this quickstart, you create a [serverless sandbox](container-instances-ai-sandboxes.md) in Azure Container Instances with the three [sandbox APIs](container-instances-ai-sandboxes.md#sandbox-apis):

1. The **AI agents group API** (Azure Resource Manager REST), to create an AI agents group and connect to it.
1. The **sandbox management API** (REST), to create a sandbox in the group.
1. The **sandbox gateway API** (gRPC), to run a command, copy a file, and view logs in the sandbox.

Then you delete the resources.

## Prerequisites

[!INCLUDE [container-instances-ai-sandboxes-prerequisites](./includes/container-instances-ai-sandboxes-prerequisites.md)]

## Send the requests

The examples in this quickstart show the method, URL, headers, and body of each HTTP request. You can send them with any HTTP client. For example, in a Bash shell:

* For the **AI agents group API**, use `az rest`, which authenticates to Azure Resource Manager with your Azure CLI sign-in. For example, this command sends the request that [creates the AI agents group](#create-an-ai-agents-group):

    ```azurecli
    subscriptionId=$(az account show --query id --output tsv)
    groupUrl="https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup"
    apiVersion="api-version=2026-08-01-preview"

    az rest --method put --url "$groupUrl?$apiVersion" --body '{ "location": "centralus", "properties": {} }'
    ```

* For the **sandbox management API**, save the `endpoint` and `accessToken` that the [`connect` action](#connect-to-the-ai-agents-group) returns, and use `curl`. For example, these commands connect to the group and send the request that [creates a sandbox](#create-a-sandbox):

    ```bash
    { read -r endpoint; read -r accessToken; } < <(az rest --method post --url "$groupUrl/connect?$apiVersion" --query "[endpoint, accessToken]" --output tsv)

    curl -X PUT "$endpoint/sandboxes/sandbox1?$apiVersion" \
      -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" \
      -d '{ "location": "centralus", "properties": { "image": "mcr.microsoft.com/azurelinux/base/core:3.0", "resources": { "cpu": 1, "memory": 2 } } }'
    ```

## Create an AI agents group

An AI agents group is an Azure Resource Manager resource of type `Microsoft.ContainerInstance/AIAgentsGroups`. It holds your sandboxes and the settings that they share. When you create an AI agents group, the service also creates a managed resource group in your subscription containing a storage account used to save and restore snapshots.

1. Create a resource group:

    ```azurecli
    az group create --name myResourceGroup --location centralus
    ```

1. Create the AI agents group:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview

    {
      "location": "centralus",
      "properties": {}
    }
    ```

1. Creation of an AI agents group is a long-running operation. Check the group until `properties.provisioningState` is `Succeeded`:

    ```http
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview
    ```

## Connect to the AI agents group

To manage sandboxes in the AI agents group, call the `connect` action of the group:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup/connect?api-version=2026-08-01-preview
```

The response includes the group's sandbox management `endpoint` and a short-lived `accessToken`:

```json
{
  "endpoint": "https://<group-endpoint-host>/subscriptions/{subscriptionId}/resourcegroups/myResourceGroup/providers/Microsoft.ContainerInstance/<group-path>",
  "accessToken": "<access-token>",
  "notAfter": "2026-11-04T10:30:00Z"
}
```

The `endpoint` value already includes the resource path for the group. Use the returned `{endpoint}` URL directly as the base URL for sandbox management requests (for example, `{endpoint}/sandboxes/sandbox1`), and pass `{accessToken}` in the `Authorization: Bearer {accessToken}` header. When the access token expires at the `notAfter` time, call `connect` again.

## Create a sandbox

1. Create a sandbox in the AI agents group:

    ```http
    PUT {endpoint}/sandboxes/sandbox1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "image": "mcr.microsoft.com/azurelinux/base/core:3.0",
        "guestOs": "linux",
        "architecture": "amd64",
        "resources": {
          "cpu": 1,
          "memory": 2
        }
      }
    }
    ```

1. Sandbox creation is asynchronous. Get the sandbox until `properties.provisioningState` is `Succeeded`, which typically takes a few seconds:

    ```http
    GET {endpoint}/sandboxes/sandbox1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

    The response includes `properties.connectionString`, which contains the gateway `address` and bearer `token` for the sandbox:

    ```json
    {
      "id": "/subscriptions/{subscriptionId}/resourcegroups/myResourceGroup/providers/Microsoft.ContainerInstance/<group-path>/sandboxes/sandbox1",
      "name": "<group-name>/sandbox1",
      "type": "Microsoft.ContainerInstance/sandboxes",
      "location": "centralus",
      "tags": {},
      "properties": {
        "resources": {
          "cpu": 1.0,
          "memory": 2.0,
          "disk": 0.0
        },
        "guestOs": "linux",
        "architecture": "amd64",
        "provisioningState": "Succeeded",
        "targetState": "Active",
        "connectionString": "address=<gateway-host>;token=<gateway-token>;token_type=bearer",
        "attachToAgentFabric": false
      }
    }
    ```

    Treat the connection string as a secret. Anyone with the token can run commands in the sandbox until the token expires.

1. List the sandboxes in the group:

    ```http
    GET {endpoint}/sandboxes?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

## Connect to the sandbox gateway

After you create the sandbox, use a gRPC client to connect to the sandbox gateway (`aci.sandboxgateway.SandboxGatewayService`) and run operations inside the sandbox. Generate a gRPC client from [`sandboxgateway.proto`](container-instances-ai-sandboxes-gateway-proto.md) and connect by using the values in `properties.connectionString`:

* **Connect to the gateway endpoint:** Open a TLS gRPC connection to `https://<gateway-host>:443` using the `address` from `properties.connectionString`.
* **Authenticate and route requests with the token:** Pass `authorization: Bearer <gateway-token>` in the gRPC request headers (metadata). The gateway uses this token both to authorize the request and to identify the target sandbox.

To build the sample C# CLI tool (`SandboxGatewayCli`) used in the following steps, install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) and follow [Build a C# CLI from sandboxgateway.proto](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto).

Call `aci.sandboxgateway.SandboxGatewayService/WaitForReady` to wait until the guest inside the sandbox is running.

By using the [sample C# CLI](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto):

```bash
dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" wait
```

gRPC request payload:

```json
{
  "timeoutSeconds": 120
}
```

Response:

```json
{
  "ready": true,
  "currentState": "running"
}
```

## Run a command in the sandbox

Call `aci.sandboxgateway.SandboxGatewayService/ExecSandbox` to run a command in the sandbox. `command` is the executable to run, and `args` are its arguments.

Use the sample C# CLI:

```bash
dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" exec /bin/sh -c "cat /etc/os-release; nproc"
```

gRPC request payload:

```json
{
  "command": "/bin/sh",
  "args": ["-c", "cat /etc/os-release; nproc"],
  "timeoutSeconds": 60
}
```

The response includes `stdout` with the operating system of the image (Azure Linux 3.0) and the CPU count of the sandbox (`1`), `stderr`, and `exitCode`.

## Copy a file into and out of the sandbox

1. Write a file to the sandbox with `aci.sandboxgateway.SandboxGatewayService/WriteFile` (`content` is raw bytes, or Base64-encoded in JSON).

    By using the sample C# CLI:

    ```bash
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" write-file /tmp/hello.txt $'hello from my computer\n'
    ```

    gRPC request payload:

    ```json
    {
      "path": "/tmp/hello.txt",
      "content": "aGVsbG8gZnJvbSBteSBjb21wdXRlcgo="
    }
    ```

1. Append a line to the file inside the sandbox with `aci.sandboxgateway.SandboxGatewayService/ExecSandbox`.

    By using the sample C# CLI:

    ```bash
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" exec /bin/sh -c "echo 'hello from the sandbox' >> /tmp/hello.txt"
    ```

    gRPC request payload:

    ```json
    {
      "command": "/bin/sh",
      "args": ["-c", "echo 'hello from the sandbox' >> /tmp/hello.txt"]
    }
    ```

1. Read the file with `aci.sandboxgateway.SandboxGatewayService/ReadFile`.

    By using the sample C# CLI:

    ```bash
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" read-file /tmp/hello.txt
    ```

    gRPC request payload:

    ```json
    {
      "path": "/tmp/hello.txt"
    }
    ```

    The streamed response returns the file bytes in `content` and `"eof": true`:

    ```output
    hello from my computer
    hello from the sandbox
    ```

## View sandbox logs

Fetch the sandbox logs with `aci.sandboxgateway.SandboxGatewayService/GetLogs`.

By using the sample C# CLI:

```bash
dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" logs 20
```

gRPC request payload:

```json
{
  "tail": 20,
  "timestamps": true
}
```

For all the gateway methods, see [Sandbox gateway API](container-instances-ai-sandboxes-api-reference.md#sandbox-gateway-api).

## Clean up resources

1. Delete the sandbox:

    ```http
    DELETE {endpoint}/sandboxes/sandbox1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

1. If you don't plan to do the tutorials, delete the AI agents group and the resource group. Deletion of an AI agents group is a long-running operation and deletes all sandboxes in the group.

    ```http
    DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview
    ```

    ```azurecli
    az group delete --name myResourceGroup
    ```

The tutorials use the AI agents group from this quickstart. To do them, keep the AI agents group.

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Save and restore a sandbox](container-instances-ai-sandboxes-tutorial-save-restore.md)
