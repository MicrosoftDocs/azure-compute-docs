---
title: Serverless sandboxes in Azure Container Instances (Public Preview)
description: Learn about serverless sandboxes in Azure Container Instances, which run AI agent workloads in fast, hardware-isolated micro VMs, and about their core concepts, APIs, and capabilities.
ms.topic: concept-article
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/03/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want to understand what serverless sandboxes in Azure Container Instances are and what they can do, so that I can decide whether to use them to run agent-generated code securely.
---

# Serverless sandboxes in Azure Container Instances (Public Preview)

[!INCLUDE [preview-callout](./includes/preview-callout.md)]

Serverless sandboxes in Azure Container Instances (ACI) provide purpose-built cloud compute for AI agents. Each *sandbox* is a lightweight micro VM with its own Hyper-V isolation boundary that starts quickly and gives an agent a stateful environment to execute code, run tools, manage files, call APIs, and maintain state across tasks.

## Why use sandboxes for AI agents

AI agents increasingly need to execute generated code, run CLI tools and browsers, install packages, read and write files, run temporary services, and preserve state across multi-step tasks. Sandboxes provide **hardware-based isolation**, **fast startup**, **stateful persistence**, and **high scale** for these workloads:

* **Fast startup.** Start isolated environments in seconds, or restore a prewarmed environment from a snapshot instead of reinstalling packages every time.
* **Stateful execution.** Install packages, write files, and keep background processes running across commands.
* **Pause, resume, stop, and start.** Pause or stop an idle sandbox and resume it when the agent needs it again.
* **Full-state snapshots.** Capture memory, filesystem, and runtime state in a snapshot, and restore one or many sandboxes from that exact point in time.
* **Direct, agent-friendly APIs.** Run commands, open interactive shells, read and write files, forward ports, and run containers and pods directly through the sandbox gateway instead of routing every command through the control plane.
* **High scale.** Run large numbers of sandboxes in an AI agents group, and create and delete them in parallel.
* **Azure Kubernetes Service (AKS) support.** If you run Kubernetes, you can also schedule and manage serverless sandboxes in ACI from your AKS cluster. For more information, see [AKS Agent Sandbox Extension](https://aka.ms/aks/agent-sandbox/serverless) and [AKS Pod Snapshot Restore Extension](https://aka.ms/aks/pod-snapshot/serverless).

A typical sandbox lifecycle for an agent looks like this:

`Create` → `Run code and tools` → `Save state (snapshot)` → `Pause or stop` → `Resume or restore` → `Delete`

### Isolation properties

* **Hardware-based isolation.** Each sandbox runs in its own Hyper-V micro VM and doesn't share a kernel with other sandboxes or with the host.
* **Per-sandbox resources.** Each sandbox has its own dedicated CPU, memory, and disk allocation, with support for in-place resizing.
* **Per-sandbox identity.** A sandbox can acquire Microsoft Entra tokens only if you assign it an agent identity.
* **Policy-controlled egress.** When you attach a sandbox to an [Agent Grid](/azure/agent-grid/overview), outbound traffic is denied by default unless a policy allows it.
* **Isolated enclaves.** A parent sandbox can start networkless enclave micro VMs reachable only over vsock for sensitive operations.

## Core concepts

```text
AI agents group (Microsoft.ContainerInstance/AIAgentsGroups)
├── Sandbox A (Hyper-V micro VM)
├── Sandbox B (Hyper-V micro VM)
├── Snapshot A (memory + filesystem + runtime state)
└── Shared identity, virtual network, and Agent Grid bindings
```

| Concept | Description |
|---|---|
| AI agents group | The top-level Azure Resource Manager resource (`Microsoft.ContainerInstance/AIAgentsGroups`) and tenant boundary. An AI agents group holds your sandboxes and snapshots, along with the shared identity, virtual network, and Agent Grid settings that apply to the group. When you create an AI agents group, the service provisions a managed resource group and a storage account in your subscription to store snapshots for save and restore. |
| Sandbox | A Hyper-V isolated micro VM in an AI agents group. Inside a sandbox, an agent can run code and shell commands, install packages, run browsers and temporary services, manage files, and launch containers and pods. |
| Snapshot | A saved copy of a sandbox that captures its **memory**, **filesystem**, and **runtime state**. You can restore one or many new sandboxes from a snapshot. |
| Enclave | A second micro VM that a parent sandbox starts from inside the guest. An enclave has no network access and is reachable only from its parent over vsock. |
| Sandbox gateway | A gRPC service that connects directly to a running sandbox to execute commands, open shells, manage files, forward ports, and run containers and pods. |

The basic flow has these steps:

1. You create an AI agents group in a region with the Azure Resource Manager REST API. The service provisions a managed resource group and storage account in your subscription for snapshot storage. Optionally, you connect the group to a virtual network, attach managed identities, or associate the group with an Agent Grid.
1. You create sandboxes in the group, or restore them from a snapshot, with the sandbox management API of the group. Sandbox creation is asynchronous.
1. Your agent connects directly to the sandbox gateway to run commands, manage files, and interact with the running environment.
1. You snapshot, pause, stop, or delete sandboxes as your agent workflow progresses. When you delete an AI agents group, its sandboxes are also deleted.

## Sandbox APIs

The service separates the management of the parent AI agents group from the operations on the sandboxes inside it. You use three APIs:

| API | Protocol and endpoint | Authorization | Operations |
|---|---|---|---|
| AI agents group API | Azure Resource Manager REST API at `https://management.azure.com` | Microsoft Entra ID and Azure role-based access control (RBAC) | Create, update, list, and delete AI agents groups. Get the group's sandbox management `endpoint` and `accessToken` with the `connect` action. |
| Sandbox management API | REST API at the `endpoint` that the `connect` action returns | The short-lived `accessToken` that the `connect` action returns | Create, list, get, and delete sandboxes. Start, stop, restart, and resize sandboxes. Save, list, and delete snapshots. |
| Sandbox gateway API | gRPC service `aci.sandboxgateway.SandboxGatewayService` at the `address` in the `connectionString` of a sandbox | The `token` in the `connectionString` of the sandbox | Wait until a sandbox is ready. Run commands, open shells, and read and write files in a sandbox. Forward ports to a sandbox. Run containers and pods in a sandbox with the Kubernetes Container Runtime Interface (CRI). |

Because you manage sandboxes through the group's endpoint rather than as individual Azure Resource Manager resources, you can create and delete them quickly and in large numbers without Azure Resource Manager deployment limits. Azure RBAC on the AI agents group controls who can get an access token from `connect`.

To build a client or CLI tool for the sandbox gateway, generate it from the gateway protobuf file, [`sandboxgateway.proto`](container-instances-ai-sandboxes-gateway-proto.md), with the gRPC tools for your language. For a step-by-step C# walkthrough, see [Build a C# CLI from sandboxgateway.proto](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto).

For the full list of operations, see [Serverless sandboxes REST and gateway API reference](container-instances-ai-sandboxes-api-reference.md).

## Capabilities

| Capability | What it does | Get started |
|---|---|---|
| Linux sandboxes | Create an isolated Linux micro VM. Run commands, open shells, and move files in and out. | [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md) |
| Save and restore | Save the state of a sandbox in a snapshot, and then create one or many sandboxes from it. | [Tutorial: Save and restore a sandbox](container-instances-ai-sandboxes-tutorial-save-restore.md) |
| Enclave | From a parent sandbox, start enclave micro VMs that have no network access, within a reserved CPU and memory budget. | [Tutorial: Run workloads in an enclave](container-instances-ai-sandboxes-tutorial-enclave.md) |
| Agent identity | Give a sandbox a Microsoft Entra Agent ID, so that code in the sandbox can access Azure resources as that agent. | [Tutorial: Access Azure resources with an agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md) |
| Stretchable sandboxes | Set an initial, minimum, and maximum size for CPU and memory, and then resize a running sandbox in place. | [Stretchable sandboxes](#stretchable-sandboxes) |
| Virtual network integration | Connect an AI agents group to a delegated subnet, so that sandboxes can reach private resources. | [Tutorial: Connect sandboxes to a virtual network](container-instances-ai-sandboxes-tutorial-virtual-network.md) |
| Agent Grid | Give each sandbox a workload identity, and allow outbound traffic only to the destinations that policies allow. | [Tutorial: Control outbound traffic with Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md) |
| Confidential sandboxes | Run a sandbox in an AMD SEV-SNP trusted execution environment with a confidential computing enforcement (CCE) policy. | [Confidential sandboxes](#confidential-sandboxes) |
| Sandbox gateway | Run commands, open shells, read and write files, and forward ports in a running sandbox. | [Sandbox gateway API](container-instances-ai-sandboxes-api-reference.md#sandbox-gateway-api) |
| Containers and pods in a sandbox | Pull images, and run pods and containers inside a sandbox with the Kubernetes Container Runtime Interface (CRI). Each sandbox keeps its own micro VM isolation boundary around the pods and containers that run in it. | [Run containers and pods in a sandbox](container-instances-ai-sandboxes-api-reference.md#run-containers-and-pods-in-a-sandbox) |

For step-by-step walkthroughs of [saving and restoring snapshots](container-instances-ai-sandboxes-tutorial-save-restore.md), [virtual network integration](container-instances-ai-sandboxes-tutorial-virtual-network.md), [Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md), [agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md), and [enclaves](container-instances-ai-sandboxes-tutorial-enclave.md), follow the linked tutorials in the preceding table. The following sections describe stretchable sandboxes and confidential sandboxes.

### Stretchable sandboxes

A *stretchable* sandbox has an `initial`, `min`, and `max` value (`min <= initial <= max`) for both `resources.cpu` (cores) and `resources.memory` (GiB), with support for fractional values. The sandbox starts with the `initial` values, and its resource bounds are immutable after creation.

To resize a running stretchable sandbox in place without restarting its processes, call `POST {endpoint}/sandboxes/{sandbox}/resize?api-version=2026-08-01-preview` with `{ "cpu": <cores>, "memoryInGB": <GiB> }` within the `[min, max]` range, and wait for the operation to complete with `WaitForOperationComplete`. Use `GET {endpoint}/sandboxes/{sandbox}` (`resources.cpu.current` and `resources.memory.current`) to check the active allocation.

### Confidential sandboxes

A *confidential* sandbox runs in a hardware-based trusted execution environment (TEE) that uses AMD SEV-SNP. The sandbox's memory is encrypted, and a confidential computing enforcement (CCE) policy controls which containers and commands can run in it.

To create a confidential sandbox, set `properties.confidentialComputeProperties.ccePolicy` to a Base64-encoded CCE policy when you create the sandbox. Generate the policy by using the [Azure CLI confcom extension](https://github.com/Azure/azure-cli-extensions/blob/main/src/confcom/azext_confcom/README.md). The policy must allow every command that you run in the sandbox through the sandbox gateway. In the sandbox, the `/dev/sev-guest` device is available for attestation. For more information, see [Confidential containers on Azure Container Instances](container-instances-confidential-overview.md) and [Attestation in confidential containers](confidential-containers-attestation-concepts.md).

Currently, a CCE policy isn't supported with an [enclave budget](container-instances-ai-sandboxes-tutorial-enclave.md), [Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md), or [virtual network integration](container-instances-ai-sandboxes-tutorial-virtual-network.md).

## Related content

* [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md)
* [Tutorial: Save and restore a sandbox](container-instances-ai-sandboxes-tutorial-save-restore.md)
* [Tutorial: Connect sandboxes to a virtual network](container-instances-ai-sandboxes-tutorial-virtual-network.md)
* [Tutorial: Control outbound traffic with Agent Grid](container-instances-ai-sandboxes-tutorial-agent-grid.md)
* [Tutorial: Access Azure resources with an agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md)
* [Tutorial: Run workloads in an enclave](container-instances-ai-sandboxes-tutorial-enclave.md)
* [Serverless sandboxes REST and gateway API reference](container-instances-ai-sandboxes-api-reference.md)
* [Sandbox gateway protobuf reference (`sandboxgateway.proto`)](container-instances-ai-sandboxes-gateway-proto.md)
