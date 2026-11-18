---
title: "Tutorial: Save and restore a serverless sandbox (Preview)"
description: In this tutorial, you use the sandbox management REST API to save the state of a running serverless sandbox in Azure Container Instances in a snapshot, and then restore one and many sandboxes from it.
ms.topic: tutorial
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/04/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want to prepare a sandbox once and start many copies of it from a snapshot, so that my agents don't wait for setup every time they need a sandbox.
---

# Tutorial: Save and restore a serverless sandbox (preview)

A *snapshot* saves the state of a [serverless sandbox in Azure Container Instances](container-instances-ai-sandboxes.md), including the processes that run in it and their memory. The service stores snapshots in the storage account it provisions in the managed resource group of your AI agents group. When you restore a sandbox from a snapshot, the sandbox starts from the moment of the snapshot and doesn't repeat the setup that you did in the source sandbox.

Use snapshots to prepare a sandbox once, for example with installed packages and a warmed-up agent, and then quickly start copies of it.

In this tutorial, you start a process in a sandbox that writes the time to a log every second. You snapshot the sandbox, and then restore copies of it. The logs in the copies show that they start from the state of the snapshot.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Start a process in a source sandbox
> * Save the sandbox in a snapshot with the `saveSnapshot` action
> * Restore a sandbox from the snapshot, and check its state
> * Restore many sandboxes from the same snapshot

## Prerequisites

[!INCLUDE [container-instances-ai-sandboxes-prerequisites](./includes/container-instances-ai-sandboxes-prerequisites.md)]

* An AI agents group, and the `{endpoint}` and `{accessToken}` from the `connect` action. To create one, complete [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md).

## Start a process in a source sandbox

1. Create the source sandbox:

    ```http
    PUT {endpoint}/sandboxes/source1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "image": "mcr.microsoft.com/azurelinux/base/core:3.0",
        "resources": {
          "cpu": 1,
          "memory": 2
        }
      }
    }
    ```

    Wait until `properties.provisioningState` is `Succeeded`. Then copy `properties.connectionString` from the response, and wait until the guest is ready (`WaitForReady`). Use the [sample C# CLI](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto):

    ```bash
    dotnet run -- -cs "<source1-connection-string>" wait
    ```

1. In `source1`, start a background process that appends the current UTC time to `/tmp/clock.log` every second (`ExecSandbox`):

    ```bash
    dotnet run -- -cs "<source1-connection-string>" exec /bin/sh -c "nohup sh -c 'while true; do date -u +%H:%M:%S >> /tmp/clock.log; sleep 1; done' >/dev/null 2>&1 < /dev/null &"
    ```

1. Wait a few seconds, and then read `/tmp/clock.log` in `source1`:

    ```bash
    dotnet run -- -cs "<source1-connection-string>" exec cat /tmp/clock.log
    ```

    The log has one line for each second since you started the process. For example:

    ```output
    10:15:01
    10:15:02
    10:15:03
    10:15:04
    10:15:05
    ```

## Save the sandbox in a snapshot

1. Save a snapshot of the source sandbox by using the `saveSnapshot` action. When you save a snapshot, you pause the source sandbox and export its memory, filesystem, and runtime state. The snapshot name must be a valid DNS label (1-63 lowercase letters, digits, or hyphens, and it must start and end with a letter or digit).

    ```http
    POST {endpoint}/sandboxes/source1/saveSnapshot?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "snapshotName": "clock-snapshot"
    }
    ```

    The response includes an operation `endpoint` and `accessToken`.

    ```json
    {
      "endpoint": "<operation-gateway-host>",
      "accessToken": "<operation-token>"
    }
    ```

1. Call `aci.sandboxgateway.SandboxGatewayService/WaitForOperationComplete` on `<operation-gateway-host>:443` with `authorization: Bearer <operation-token>` to wait until the snapshot export finishes. By using the sample C# CLI, pass the operation `endpoint` and `accessToken` as a connection string:

    ```bash
    dotnet run -- -cs "address=<operation-gateway-host>;token=<operation-token>;token_type=bearer" wait-op 300
    ```

    gRPC request payload:

    ```json
    {
      "timeoutSeconds": 300
    }
    ```

    The response includes `"complete": true` and `"terminalState": "succeeded"`. Note the time when the snapshot finishes.

1. List the snapshots in the AI agents group.

    ```http
    GET {endpoint}/snapshots?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

    The `value` array in the response includes `clock-snapshot`.

## Restore a sandbox from the snapshot

1. Wait at least 10 seconds, so that the current time moves past the time of the snapshot.

1. Create a sandbox from the snapshot. Set the `snapshot` property instead of `image`:

    ```http
    PUT {endpoint}/sandboxes/restored1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "snapshot": "clock-snapshot"
      }
    }
    ```

    Wait until `properties.provisioningState` is `Succeeded`, copy `properties.connectionString` from the response, and wait until the guest is ready (`WaitForReady`).

1. In `restored1`, print the current UTC time and the contents of `/tmp/clock.log`:

    ```bash
    dotnet run -- -cs "<restored1-connection-string>" exec /bin/sh -c "date -u +%H:%M:%S; cat /tmp/clock.log"
    ```

    The log contains the timestamps from the source sandbox up to the moment of the snapshot, with a gap between the snapshot time and the time when the restored sandbox resumed. The restored sandbox started from the captured state of the snapshot rather than from a fresh image.

## Restore many sandboxes from the same snapshot

You can create many sandboxes from the same snapshot in parallel by sending concurrent `PUT` requests with `"snapshot": "clock-snapshot"`:

```http
PUT {endpoint}/sandboxes/{sandboxName}?api-version=2026-08-01-preview
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "location": "centralus",
  "properties": {
    "snapshot": "clock-snapshot"
  }
}
```

Each copy starts from the same snapshot state, including any environment variables captured in the snapshot's memory, and then runs independently. When you restore a sandbox, don't specify `image` or `environmentVariables` with `snapshot`. You can set other properties, such as `resources` and an [agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md) in `agentIdentities`.

## Clean up resources

1. Delete the sandboxes that you created in this tutorial, including any sandboxes that you restored in parallel.

    ```http
    DELETE {endpoint}/sandboxes/source1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

    ```http
    DELETE {endpoint}/sandboxes/restored1?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

    ```http
    DELETE {endpoint}/sandboxes/{sandboxName}?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

1. The snapshot stays in the AI agents group after you delete the source sandbox. If you no longer need it, delete it.

    ```http
    DELETE {endpoint}/snapshots/clock-snapshot?api-version=2026-08-01-preview
    Authorization: Bearer {accessToken}
    ```

If you don't plan to do the other tutorials, delete the AI agents group and the resource group.

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Connect sandboxes to a virtual network](container-instances-ai-sandboxes-tutorial-virtual-network.md)
