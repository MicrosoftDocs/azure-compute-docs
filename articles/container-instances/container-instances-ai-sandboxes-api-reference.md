---
title: Serverless sandboxes REST and gateway API reference (Preview)
description: Reference for the serverless sandboxes APIs in Azure Container Instances, including the AI agents group API, the sandbox management API, and the sandbox gateway gRPC API.
ms.topic: reference
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/04/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want a complete list of the serverless sandbox operations, request properties, and gateway methods, so that I can build my own client for serverless sandboxes in Azure Container Instances.
---

# Serverless sandboxes REST and gateway API reference (preview)

[Serverless sandboxes in Azure Container Instances](container-instances-ai-sandboxes.md) have three APIs. The parent AI agents group is an Azure Resource Manager resource, and you manage the sandboxes inside it through the group's endpoint.

| API | Endpoint | Authorization |
|---|---|---|
| [AI agents group API](#ai-agents-group-api) | `https://management.azure.com` | Microsoft Entra ID token for Azure Resource Manager, and Azure RBAC on the AI agents group |
| [Sandbox management API](#sandbox-management-api) | The `endpoint` that the `connect` action returns | `Authorization: Bearer <accessToken>`, with the `accessToken` that the `connect` action returns |
| [Sandbox gateway API](#sandbox-gateway-api) | The `address` in the `connectionString` of a sandbox, on port 443 | `authorization: Bearer <token>`, with the `token` in the `connectionString` of the sandbox |

All requests use API version `2026-08-01-preview`.

## AI agents group API

The AI agents group API is the Azure Resource Manager REST API for the `Microsoft.ContainerInstance/AIAgentsGroups` resource type. The URLs in this table are relative to `https://management.azure.com/subscriptions/{subscriptionId}`.

| Operation | Method and URL |
|---|---|
| Create or update an AI agents group | `PUT /resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/AIAgentsGroups/{agentsGroup}` |
| Get an AI agents group | `GET /resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/AIAgentsGroups/{agentsGroup}` |
| List AI agents groups in a resource group | `GET /resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/AIAgentsGroups` |
| List AI agents groups in a subscription | `GET /providers/Microsoft.ContainerInstance/AIAgentsGroups` |
| Delete an AI agents group | `DELETE /resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/AIAgentsGroups/{agentsGroup}` |
| Connect to an AI agents group | `POST /resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/AIAgentsGroups/{agentsGroup}/connect` |

Creation, update, and deletion of an AI agents group are long-running operations. Send `GET` requests until `properties.provisioningState` is `Succeeded`. Deletion of an AI agents group also deletes its sandboxes.

### AI agents group properties

| Property | Description |
|---|---|
| `location` | The Azure region of the AI agents group. |
| `tags` | Resource tags. |
| `identity` | The managed identities of the AI agents group, in the standard Azure Resource Manager `identity` format, with `type` and `userAssignedIdentities`. |
| `properties.networkProfile.subnets[].id` | The resource ID of a subnet that's delegated to `Microsoft.ContainerInstance/AIAgentsGroups`. Set it when you create the group. You can't change it later. |
| `properties.identityProfile.federatedIdentityCredential` | The principal ID of the managed identity that [agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md) blueprints trust, or `SystemIdentity`. |
| `properties.identityProfile.imageRegistryCredential` | The principal ID of the managed identity that pulls images, or `SystemIdentity`. |
| `properties.identityProfile.sandboxCredential` | An array of principal IDs of managed identities that sandboxes can use, or `SystemIdentity`. |
| `properties.agentFabricResourceId` | The resource ID of an [Azure Agent Grid](/azure/agent-grid/overview) resource (`Microsoft.NetworkSecurity/agentFabrics`) in the same subscription and region (see [Agent Grid region availability](/azure/agent-grid/overview#region-availability)). Set it when you create the group. You can't change it later, and you can't combine it with `networkProfile`. |
| `properties.provisioningState` | Read-only. The provisioning state of the AI agents group. |
| `properties.sandboxGroupUniqueId` | Read-only. The stable unique identifier generated for the AI agents group. |
| `properties.managedResourceGroupId` | Read-only. The resource ID of the managed resource group provisioned for the AI agents group. |
| `properties.storageAccountResourceId` | Read-only. The resource ID of the managed storage account provisioned for snapshot save and restore. |

Example request:

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myAgentsGroup?api-version=2026-08-01-preview

{
  "location": "centralus",
  "identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity": {}
    }
  },
  "properties": {
    "networkProfile": {
      "subnets": [
        { "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/sandbox-subnet" }
      ]
    },
    "identityProfile": {
      "federatedIdentityCredential": "<identity-principal-id>"
    }
  }
}
```

### Connect response

The `connect` action returns the sandbox management endpoint of the AI agents group, and a short-lived access token for it:

```json
{
  "endpoint": "https://<group-endpoint-host>/subscriptions/{subscriptionId}/resourcegroups/myResourceGroup/providers/Microsoft.ContainerInstance/<group-path>",
  "accessToken": "<access-token>",
  "notAfter": "2026-11-04T10:30:00Z"
}
```

The `endpoint` value already includes the resource path for the group. Use the returned `endpoint` URL directly. When the access token expires at the `notAfter` time, call `connect` again.

## Sandbox management API

The sandbox management API is the REST API of an AI agents group. The URLs in this table are relative to the `endpoint` that the `connect` action returns. Add `?api-version=2026-08-01-preview` to each request.

| Operation | Method and URL | Request body |
|---|---|---|
| Create a sandbox | `PUT /sandboxes/{sandbox}` | [Sandbox properties](#sandbox-properties) |
| Get a sandbox | `GET /sandboxes/{sandbox}` | None |
| List sandboxes | `GET /sandboxes` | None |
| Delete a sandbox | `DELETE /sandboxes/{sandbox}` | None |
| Stop a sandbox | `POST /sandboxes/{sandbox}/stop` | None |
| Start a stopped sandbox | `POST /sandboxes/{sandbox}/start` | None |
| Restart a sandbox | `POST /sandboxes/{sandbox}/restart` | None |
| Resize a sandbox | `POST /sandboxes/{sandbox}/resize` | `{ "cpu": <cores>, "memoryInGB": <GiB> }` |
| Save a snapshot of a sandbox | `POST /sandboxes/{sandbox}/saveSnapshot` | `{ "snapshotName": "<name>" }` |
| List snapshots | `GET /snapshots` | None |
| Delete a snapshot | `DELETE /snapshots/{snapshot}` | None |

Sandbox creation is asynchronous. Send `GET` requests until `properties.provisioningState` is `Succeeded`, and then call the [`WaitForReady`](#sandbox-gateway-api) method of the gateway to wait until the guest is running.

List operations return a `value` array, and a `nextLink` URL when there are more results.

The `stop` action is asynchronous. The sandbox moves to `Stopping` and then to `Stopped` in `properties.provisioningState`. Call `start` only after the sandbox reaches `Stopped`. A stopped sandbox has no `connectionString`, so call `GET /sandboxes/{sandbox}` again after `start` to retrieve the new connection string.

The `resize` and `saveSnapshot` actions require a running sandbox and return an operation `endpoint` and `accessToken`. To wait until the operation finishes, call the `WaitForOperationComplete` method of the gateway at that endpoint, with that token. Saving a snapshot pauses the source sandbox and exports its memory, filesystem, and runtime state.

### Sandbox properties

| Property | Description |
|---|---|
| `location` | The region of the AI agents group. |
| `tags` | Tags for the sandbox. |
| `properties.image` | Optional OCI image reference or public image alias to initialize the sandbox root filesystem from. Omit to use the default guest environment. Mutually exclusive with `snapshot`. |
| `properties.snapshot` | The name of a snapshot in the same AI agents group to [restore](container-instances-ai-sandboxes-tutorial-save-restore.md) the sandbox from. Mutually exclusive with `image` and `environmentVariables`. |
| `properties.guestOs` | `linux` (default). |
| `properties.architecture` | `amd64` (default). |
| `properties.resources.cpu` | The CPU cores (fractional values supported). Use a scalar number for fixed sizing, or an object with `initial`, `min`, and `max` values (`min <= initial <= max`) for a [stretchable sandbox](container-instances-ai-sandboxes.md#stretchable-sandboxes). Both `cpu` and `memory` are required when bounds are specified. |
| `properties.resources.memory` | The memory in GiB (fractional values supported). Use a scalar number for fixed sizing, or an object with `initial`, `min`, and `max` values (`min <= initial <= max`) for a stretchable sandbox. |
| `properties.resources.disk` | The disk size in GB. |
| `properties.environmentVariables` | Optional array of default environment variables (`{ "name": "<name>", "value": "<value>" }` or `{ "name": "<name>", "secureValue": "<secret>" }`) applied to every `exec` and `shell` process in the sandbox. Supports up to 100 entries and 64 KB total. Create-only; `secureValue` is write-only and omitted on `GET`; cannot be combined with `snapshot`. |
| `properties.dataDisks` | Optional array of managed data disks attached to the sandbox (`{ "id": "<managed-disk-resource-id>" }`). |
| `properties.trustedCaCertificates` | Optional array of root or intermediate CA certificates (`{ "format": "pem", "value": "<base64-pem>" }`) to add to the sandbox trust store. |
| `properties.diagnostics` | Optional diagnostics settings (`{ "logAnalytics": { "workspaceId": "<guid>", "workspaceKey": "<key>" } }`) to send container `stdout`/`stderr` to a Log Analytics workspace. Create-only; `workspaceKey` is write-only. |
| `properties.confidentialComputeProperties.ccePolicy` | The Base64-encoded CCE policy of a [confidential sandbox](container-instances-ai-sandboxes.md#confidential-sandboxes). Currently not supported with `peerSandboxProfile`, `attachToAgentFabric`, or virtual network integration (`networkProfile`). |
| `properties.peerSandboxProfile` | The [enclave](container-instances-ai-sandboxes-tutorial-enclave.md) budget, with `cpu` (1-64) and `memoryInGB` (1-256) as whole numbers. Create-only; cannot be combined with `confidentialComputeProperties` or `resources` `min`/`max` bounds. |
| `properties.agentIdentities` | An array with one [agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md) entry containing both `agentId` and `agentBlueprintId`. |
| `properties.attachToAgentFabric` | `true` to attach the sandbox to the [Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md) of the AI agents group. Cannot be combined with `confidentialComputeProperties`, and cannot be changed from `true` to `false` after creation. |
| `properties.harvest` | Optional boolean (`true` or `false`) indicating whether the sandbox runs on harvest capacity. |
| `properties.dnsConfig` | DNS settings (`servers`, `searches`, `options`) for the sandbox. Defaults to Azure DNS (`168.63.129.16`). |
| `properties.provisioningState` | Read-only. The provisioning state of the sandbox. |
| `properties.targetState` | Read-only. `Active`, or `Stopped` after a `stop` action. |
| `properties.resources.cpu.current`, `properties.resources.memory.current` | Read-only. The current resource allocation of a stretchable sandbox. |
| `properties.connectionString` | Read-only. The `address` and `token` of the sandbox gateway for the sandbox, in the form `address=<host>;token=<token>;token_type=bearer`. Treat it as a secret. |
| `properties.instanceView.errorMessage` | Read-only. Why the sandbox failed, when it failed. |

Example request:

```http
PUT {endpoint}/sandboxes/sandbox1?api-version=2026-08-01-preview
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

### Snapshot names

A snapshot name must be a DNS label: 1-63 lowercase letters, digits, or hyphens, that starts and ends with a letter or digit.

## Sandbox gateway API

The sandbox gateway API is the gRPC service `aci.sandboxgateway.SandboxGatewayService`. Use it to work inside a running sandbox.

To call the gateway:

1. Get the `connectionString` of the sandbox from the sandbox management API (`GET {endpoint}/sandboxes/{sandbox}`). It has the form `address=<host>;token=<token>;token_type=bearer`.
1. Connect to `<host>` on port 443 with TLS.
1. Send the token in the `authorization` metadata of each call, as `Bearer <token>`. The gateway identifies the sandbox from the token, so requests don't have a sandbox name.

Generate a client for your language from [`sandboxgateway.proto`](container-instances-ai-sandboxes-gateway-proto.md). For a C# CLI walkthrough and usage examples, see [Build a C# CLI from sandboxgateway.proto](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto) and [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md).

| Method | Type | Request fields | Description |
|---|---|---|---|
| `WaitForReady` | Unary | `timeout_seconds` | Wait until the sandbox is ready. Returns `ready`, `current_state`, and, when the host reports them, startup times. |
| `GetSandbox` | Unary | None | Get the definition and current state of the sandbox. |
| `GetLogs` | Unary | `tail`, `level`, `timestamps`, `cursor` | Read the running sandbox's log buffer. Returns `entries`, `next_cursor`, `gap_detected`, and `has_more`. |
| `ExecSandbox` | Unary | `command`, `args`, `working_directory`, `environment`, `timeout_seconds`, `user` | Run a command in the sandbox. Returns `exit_code`, `stdout`, `stderr`, and `timed_out`. |
| `InteractiveShell` | Bidirectional stream | `start` (`shell`, `args`, `environment`, `working_directory`, `initial_rows`, `initial_cols`, `user`, `non_terminal`), then `input`, `window_size`, or `signal` | Open an interactive or streaming shell session in the sandbox. |
| `ExecContainer` | Bidirectional stream | `start` (`container_id`, `cmd`, `tty`, `stdin`, `working_directory`, `environment`, `initial_rows`, `initial_cols`), then `input`, `window_size`, or `signal` | Run an interactive command in a container in the sandbox. |
| `PortForward` | Bidirectional stream | `start` (`port`, `host`), then `data` or `close` | Forward a TCP connection to a port in the sandbox. |
| `WriteFile` | Unary | `path`, `content`, `mode`, `append`, `container_id` | Write a file in the sandbox or in a container (`container_id`). |
| `WriteFileStream` | Client stream | `start` (`path`, `mode`, `append`, `container_id`), then `content` | Stream bytes to a file without buffering the entire file in memory. |
| `ReadFile` | Server stream | `path`, `offset`, `max_chunk_size`, `container_id` | Read a file in chunks. Each response has `content`, `next_offset`, and `eof`. |
| `DownloadFromBlobstore` | Unary | `source_sas_url`, `path`, `mode`, `container_id` | Download a blob from Azure Blob Storage directly into the sandbox or a container using a SAS URL. |
| `DownloadFileToUrl` | Unary | `path`, `destination_sas_url`, `container_id` | Upload a file from the sandbox or a container directly to Azure Blob Storage using a SAS URL. |
| `CreateDirectory` | Unary | `path`, `mode`, `container_id` | Create a directory and its parent directories. |
| `ListDirectory` | Unary | `path`, `container_id` | List a directory. |
| `Move` | Unary | `source`, `destination`, `overwrite`, `container_id` | Move or rename a file or directory. |
| `DeleteFile` | Unary | `path`, `recursive`, `container_id` | Delete a file or directory. |
| `SearchContent` | Unary | `path`, `pattern`, `is_regex`, `case_sensitive`, `max_results`, `container_id` | Search files for text. |
| `ReplaceContent` | Unary | `path`, `pattern`, `replacement`, `is_regex`, `case_sensitive`, `container_id` | Replace text in a file. |
| `BindMount` | Unary | `source`, `target`, `readonly`, `recursive` | Bind-mount a path in the sandbox to another path. |
| `ConnectIscsi` | Unary | `target_iqn`, `portal`, `port`, `sessions`, `timeout_seconds`, `format_if_empty` | Connect the sandbox guest to an iSCSI target. Returns `session_ids`, `devices`, `portal_address`, and `multipath_devices`. |
| `WatchChanges` | Server stream | `path`, `recursive` | Stream changes to a path until the call ends. |
| `PauseSandbox`, `ResumeSandbox` | Unary | None | Pause or resume the sandbox. |
| `WaitForOperationComplete` | Unary | `timeout_seconds` | Wait until a long-running operation, such as a snapshot or a resize, completes. Call it with the operation endpoint and token. Returns `complete` and `terminal_state`. |

In JSON representations of protobuf messages, field names use lower camel case, such as `timeoutSeconds`, and `bytes` fields, such as `content`, are Base64-encoded.

## Enclave host bridge API

From inside a parent sandbox that has an enclave budget (`properties.peerSandboxProfile`), the guest communicates with the host's gRPC service `aci.hostbridge.peersandbox.v1.PeerSandboxService` over vsock (`AF_VSOCK`, host `CID 2`, port `5090`) to manage enclaves and vsock port forwards. For the complete `peersandbox.proto` definition and `sandboxctl` CLI walkthrough, see [Tutorial: Run sensitive workloads in a serverless sandbox enclave](container-instances-ai-sandboxes-tutorial-enclave.md#enclave-protobuf-definition-peersandboxproto).

| Method | Type | Request fields | Description |
|---|---|---|---|
| `CreatePeer` | Unary | `name`, `image`, `cores`, `memory_in_gb` | Create an enclave micro VM within the parent sandbox's `peerSandboxProfile` budget. |
| `GetPeer` | Unary | `name` | Get the current state and resource allocation of an enclave. |
| `ListPeers` | Unary | None | List all enclaves of the parent sandbox and the remaining budget (`cores`, `memory_in_gb`, `used_cores`, `used_memory_in_gb`). |
| `StartPeer`, `StopPeer` | Unary | `name` | Start a stopped enclave, or stop a running enclave. |
| `DeletePeer` | Unary | `name` | Delete an enclave. |
| `ExecPeer` | Unary | `name`, `command`, `args`, `working_directory`, `environment`, `timeout_seconds` | Run a command in an enclave. Returns `exit_code`, `stdout`, `stderr`, and `timed_out`. |
| `ShellPeer` | Bidirectional stream | `start` (`name`, `shell`, `environment`, `working_directory`, `rows`, `cols`), then `input`, `window_size`, or `signal` | Open an interactive TTY shell session in an enclave. |
| `CreateForward` | Unary | `peer`, `to_parent`, `target_port`, `listen_port` | Open a vsock port forward between the parent sandbox and an enclave. Returns `id` and `listen_port` on the host (`CID 2`). |
| `ListForwards` | Unary | None | List active vsock port forwards created by the caller. |
| `DeleteForward` | Unary | `id` | Delete a vsock port forward. |

## Run containers and pods in a sandbox

Each sandbox includes a container runtime. You can pull images and launch pods and containers inside a sandbox using the [Kubernetes Container Runtime Interface (CRI)](https://kubernetes.io/docs/concepts/architecture/cri/) through the sandbox gateway, at the same gateway `address` and with the same bearer `token` as the other gateway methods. The pods and containers run inside the micro VM of the sandbox, so the sandbox remains their hardware isolation boundary.

To run an interactive command in a container with a terminal, use the `ExecContainer` method of the sandbox gateway. When you delete the sandbox, any pods and containers running inside it are also deleted.

## Mount Azure Storage in a sandbox

You can mount Azure Files shares and Azure Blob Storage containers from inside a sandbox, for example with the `ExecSandbox` method. The sandbox needs network access to the storage account, and the image needs the mount tools.

| Storage | How to mount | Required in the image |
|---|---|---|
| Azure Files | Mount the share over SMB with `mount -t cifs`. For more information, see [Mount SMB Azure file share on Linux](/azure/storage/files/storage-how-to-use-files-linux). | `cifs-utils` |
| Azure Blob Storage | Mount the container with BlobFuse2. For more information, see [How to mount an Azure Blob Storage container on Linux with BlobFuse2](/azure/storage/blobs/blobfuse2-how-to-deploy). | `blobfuse2` and FUSE |

To avoid storage account keys in the sandbox, give the sandbox an [agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md), and grant that identity access to the storage account.

## Troubleshoot

| Symptom | Resolution |
|---|---|
| Creating an AI agents group fails with `404 Not Found` (`"code": "InvalidResourceType"`, `"message": "The resource type could not be found in the namespace 'Microsoft.ContainerInstance' for api version '2026-08-01-preview'"`), or the group's `provisioningState` is `Failed` | Quota for serverless sandboxes isn't provisioned on your subscription yet. To resolve this error, submit a [Container Instances quota support request in the Azure portal](https://portal.azure.com/#view/Microsoft_Azure_Support/NewSupportRequestV4Blade/topicId/a11041c8-bf9f-820b-f1ef-c859995c6040/productId/06bfd9d3-516b-d5c6-5802-169c800dec89/issueType/quota) with **Quota type** in **Quota details** set to **AIAgentsGroups - StandardCores (Preview)**. After your quota request is approved, re-register the `Microsoft.ContainerInstance` resource provider (`az provider register --namespace Microsoft.ContainerInstance --wait`). |
| AI agents group creation or requests fail with authorization errors | Sign in with an account that has access to the AI agents group, and make sure that the **Compute Sandbox Groups** service principal (`1eeacc2b-ec4a-4b1e-977a-2b5fef2131c6`) exists in your tenant (`az ad sp create --id 1eeacc2b-ec4a-4b1e-977a-2b5fef2131c6`) and has the **Contributor**, **Azure Deployment Stack Owner**, and **Storage Blob Data Contributor** roles on your subscription so it can provision the managed resource group and storage account. If your quota request was recently approved, re-register the `Microsoft.ContainerInstance` resource provider, and wait for role assignments to propagate. |
| Sandbox management requests fail with `401 Unauthorized` | The access token expired. Call the `connect` action again. |
| Gateway calls fail with an authorization error | Get the `connectionString` of the sandbox again, and use its token. |
| Sandbox creation fails with `Allocation failed permanently` | No host capacity is left for the request. To request more capacity, submit a [Container Instances quota support request in the Azure portal](https://portal.azure.com/#view/Microsoft_Azure_Support/NewSupportRequestV4Blade/topicId/a11041c8-bf9f-820b-f1ef-c859995c6040/productId/06bfd9d3-516b-d5c6-5802-169c800dec89/issueType/quota) (select **AIAgentsGroups - StandardCores (Preview)** for **Quota type** in **Quota details**) or [create an Azure support request](container-instances-support-help.md). |
| `resize` fails with `BadRequest` | The target is outside the `min` and `max` values of the sandbox, or the sandbox has fixed resources. |
| `saveSnapshot` fails with `BadRequest` | The sandbox isn't running, the snapshot name isn't a valid DNS label, or the sandbox has an enclave budget (`peerSandboxProfile`). |
| A sandbox can't get a token for Azure resources | Make sure that you created the sandbox with `agentIdentities`, and that the blueprint trusts the managed identity of the AI agents group. |

## Related content

* [Serverless sandboxes in Azure Container Instances](container-instances-ai-sandboxes.md)
* [Sandbox gateway protobuf reference (`sandboxgateway.proto`)](container-instances-ai-sandboxes-gateway-proto.md)
* [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md)
