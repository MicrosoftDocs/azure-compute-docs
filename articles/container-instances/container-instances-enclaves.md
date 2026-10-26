---
title: Enclaves for confidential containers on Azure Container Instances (preview)
description: Learn how to run an enclave pod next to a confidential container group, and how to start containers in it and communicate with it.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.date: 10/26/2026
ms.update-cycle: 1095-days
ms.custom: devx-track-arm-template, devx-track-azurecli
# Customer intent: "As a developer of confidential applications, I want to run a separate, policy-enforced enclave next to my confidential container group, so that I can isolate my most sensitive code and data from the rest of my application."
---

# Enclaves for confidential containers on Azure Container Instances (preview)

[!INCLUDE [preview-callout](./includes/preview-callout.md)]

An *enclave* is a separate confidential environment that runs next to a [confidential container group](container-instances-confidential-overview.md). Your confidential container group is the *parent container group*. From the parent container group, you can start an *enclave pod* on demand. The enclave pod runs in its own hardware-based trusted execution environment (TEE), with its own confidential computing enforcement (CCE) policy.

Use an enclave to isolate the most sensitive part of your application from the rest of your application. For example, you can run code that handles keys or decrypted data in an enclave pod, and keep the rest of the application in the parent container group.

This article explains how enclaves work. It also shows how to request enclave resources, start an enclave pod, run containers in it, and communicate with it.

## How enclaves work

An enclave has these components:

| Component | Description |
|---|---|
| Parent container group | A container group with the `Confidential` SKU and an `enclaveProfile` property. The enclave profile reserves CPU and memory for the enclave pod. |
| Enclave pod | A separate AMD SEV-SNP utility VM that runs one or more Linux containers. The parent container group starts the enclave pod at run time. |
| `hpodctl` | A command-line tool that runs in a container of the parent container group. It manages the enclave pod, its containers, and its port forwards. |

The enclave flow has these steps:

1. You deploy a parent container group. The `enclaveProfile` property reserves CPU and memory for the enclave pod.
1. A container in the parent container group runs `hpodctl run-pod` with a CCE policy for the enclave pod.
1. Azure Container Instances checks that the request fits the reserved resources. Then it starts the enclave pod in a separate AMD SEV-SNP utility VM.
1. The parent container runs `hpodctl run-container` to pull an image and start a container in the enclave pod. The CCE policy of the enclave pod controls which containers and commands can run.
1. The parent container group and the enclave pod communicate only through port forwards.

### Isolation properties

* **Separate TEE.** Each enclave pod runs in its own AMD SEV-SNP utility VM. The enclave pod doesn't share memory with the parent container group or with other enclave pods.
* **Separate policy.** Each enclave pod has its own CCE policy. This policy is separate from the CCE policy of the parent container group.
* **No network access.** An enclave pod doesn't have a public IP address, an IP address in your virtual network, or outbound internet access. It can communicate only with its parent container group through port forwards.
* **Owner-only access.** Only the parent container group that created an enclave pod can manage it, run commands in it, and read its logs. Only the parent container group and its enclave pod can see and stop the port forwards between them.
* **Same lifetime as the parent.** When the parent container group stops or restarts, the platform removes its enclave pod.

## Prerequisites

[!INCLUDE [container-instances-tutorial-prerequisites-confidential-containers](./includes/container-instances-tutorial-prerequisites-confidential-containers.md)]

You also need:

* A container image for the parent container group that includes the `hpodctl` tool. For more information, see [Add hpodctl to the parent container image](#add-hpodctl-to-the-parent-container-image).
* A CCE policy for the enclave pod, separate from the policy of the parent container group. For more information, see [Confidential computing enforcement policies](container-instances-confidential-overview.md#confidential-computing-enforcement-policies).
* Container images for the enclave pod in a registry that allows anonymous pull. For more information, see [Limitations](#limitations).

## Add hpodctl to the parent container image

The parent container group uses `hpodctl` to manage the enclave pod. `hpodctl` is a single Linux x64 executable with no runtime dependencies.

Add `hpodctl` to the image of the container that manages the enclave pod. Put it on the `PATH` of the container.

You don't need to configure a connection for `hpodctl`.

Make sure that the CCE policy of the parent container group allows the commands that start `hpodctl`. For example, run `hpodctl` from the entry point script of your container.

## Deploy the parent container group with an enclave profile

To reserve resources for the enclave pod, add the `enclaveProfile` property to the container group. Set these values:

* `sku` to `Confidential`.
* `osType` to `Linux`.
* Use API version `2023-05-01` or later.

The following example shows the container group resource of an ARM template. The parent container group has 1 CPU and 1.5 GB of memory. The enclave profile reserves another 1 CPU and 2 GB of memory for the enclave pod.

```json
{
  "type": "Microsoft.ContainerInstance/containerGroups",
  "apiVersion": "2023-05-01",
  "name": "[parameters('name')]",
  "location": "[parameters('location')]",
  "properties": {
    "sku": "Confidential",
    "osType": "Linux",
    "confidentialComputeProperties": {
      "ccePolicy": ""
    },
    "containers": [
      {
        "name": "parent",
        "properties": {
          "image": "<your-registry>/<parent-image>:<tag>",
          "resources": {
            "requests": {
              "cpu": 1,
              "memoryInGB": 1.5
            }
          }
        }
      }
    ],
    "enclaveProfile": {
      "cpu": 1,
      "memoryInGB": 2
    }
  }
}
```

Generate the CCE policy of the parent container group, and then deploy the template. For the full steps, see [Tutorial: Prepare a deployment for a confidential container on Azure Container Instances](container-instances-tutorial-deploy-confidential-containers-cce-arm.md).

### Enclave profile rules

| Property | Rule |
|---|---|
| `cpu` | Required with `memoryInGB`. Minimum of 0.4. |
| `memoryInGB` | Required with `cpu`. Minimum of 0.4. |

The CPU in the enclave profile counts toward the core quota of your subscription, the same as the CPU of your containers. For more information, see [Resource availability and quota limits](container-instances-resource-and-quota-limits.md).

## Start an enclave pod

Run these commands in the container of the parent container group that includes `hpodctl`. Typically, your entry point script runs them.

1. Start an enclave pod with a CCE policy for the enclave pod. The command prints the ID of the enclave pod.

    ```bash
    POD=$(hpodctl run-pod --cce-policy-file ./enclave-policy.b64)
    ```

1. Start a container in the enclave pod. The command prints the ID of the container. Use the same image, command, and environment variables that the policy allows.

    ```bash
    CTR=$(hpodctl run-container "$POD" mcr.microsoft.com/azurelinux/base/core:3.0 -e GREETING=hello -- /bin/sh -c 'while true; do echo "$GREETING"; sleep 5; done')
    ```

1. Make sure that the container runs:

    ```bash
    hpodctl containers "$POD"
    hpodctl logs "$CTR" -n 20
    ```

## Communicate with an enclave pod

The parent container group and its enclave pod are separate VMs. They can't connect to each other directly. Instead, they communicate through *port forwards*.

Apps on both sides use vsock sockets (`AF_VSOCK`), not TCP sockets:

* An app in the enclave pod must listen on a vsock port. The enclave pod can't receive TCP connections from the parent container group.
* An app in the parent container group connects to vsock CID 2 on the listen port of the forward.

### Forward from the parent container group to an enclave pod

In the parent container, create a forward to vsock port 6666 in the enclave pod. Use `-l` to set the port that the parent app connects to:

```bash
hpodctl vsock-forward 6666 --pod "$POD" -l 7777
```

The command prints the forward ID and the listen port. The forward stays active after the command exits. The parent app can now connect to vsock CID 2, port 7777. Each connection goes to port 6666 in the enclave pod.

If you don't use `-l`, Azure Container Instances selects a random listen port between 49152 and 65535, and prints it.

### Forward from an enclave pod to the parent container group

An app in the enclave pod can also connect to the parent container group. In this case, the enclave pod creates the forward. The listen port is always the same as the target port:

```bash
hpodctl vsock-forward 8080 --parent
```

To use this mode, the enclave pod must include `hpodctl`, and its CCE policy must allow the command.

### Manage port forwards

```bash
# List the port forwards that start or end at this container group or enclave pod.
hpodctl list

# Stop a forward.
hpodctl stop-vsock-forward <forward-id>
```

A parent container group or an enclave pod can be the source of a maximum of 64 forwards.

## Run commands in an enclave pod

You can't use `az container exec` to connect to an enclave pod directly. Connect to the parent container first, and then use `hpodctl exec`:

1. Connect to the parent container:

    ```azurecli
    az container exec --resource-group <resource-group> --name <container-group> --container-name parent --exec-command "/bin/sh"
    ```

1. Start a shell in the enclave container:

    ```bash
    hpodctl exec -t <container-id> -- /bin/sh
    ```

The CCE policies of the parent container group and of the enclave pod must allow these commands. For more information, see the `--debug-mode` and exec options of the [Azure CLI confcom extension](https://github.com/Azure/azure-cli-extensions/blob/main/src/confcom/azext_confcom/README.md).

To read the logs of an enclave container, use `hpodctl logs <container-id>`. Add `-f` to follow new output.

## Stop and remove enclave containers

To stop and remove an enclave container, run:

```bash
hpodctl stop-container "$CTR" -t 10
hpodctl remove-container "$CTR"
```

When the parent container group stops or restarts, the platform removes its enclave pod automatically. When you delete the parent container group, the enclave pod is also removed.

To replace the enclave pod (for example, to use a new CCE policy), restart the parent container group. Then start a new enclave pod.

## Security recommendations

* **Always use a real CCE policy for the enclave pod.** `hpodctl run-pod` also has a `--dev-policy` option. This option allows any container and any command. Use it only for development. Don't use it for production workloads.
* **Protect data in transit.** Port forwards don't encrypt the traffic between the parent container group and the enclave pod. If your data must stay confidential in transit, encrypt it end to end, for example with TLS between the parent app and the enclave app.
* **Keep the enclave policy small.** Allow only the images, commands, and environment variables that the enclave pod needs. Don't allow interactive shells in production policies.

## Limitations

* Enclaves are available only for Linux container groups with the `Confidential` SKU.
* The system pulls enclave images without registry credentials. Use a registry that allows anonymous pulls. The system doesn't support private registries that need credentials.
* An enclave pod has no network access. It can communicate only with its parent container group through port forwards.
* Only the parent container group can run commands in an enclave pod. You can't use `az container exec` to connect to an enclave pod directly.
* `az container exec` limits the command to 5,000 characters. Don't pass a CCE policy inline in the command. Use `--cce-policy-file` instead.
* A parent container group can run only one enclave pod at a time. You can't remove the enclave pod without a restart of the parent container group.
* Port forwards can close, for example during platform maintenance. Make your app detect closed connections and create the forward again.
* A parent container group or an enclave pod can be the source of a maximum of 64 port forwards.

## hpodctl command reference

| Command | Description |
|---|---|
| `run-pod (--cce-policy <base64> \| --cce-policy-file <path> \| --dev-policy) [--cpu-count <n>] [--memory <GB>]` | Start an enclave pod, and print its ID. A container group can have only one active enclave pod. If you omit `--cpu-count` or `--memory`, the enclave pod uses all of that resource in the enclave profile. Use `--dev-policy` only for development. |
| `pods` | List the enclave pod of your container group. |
| `run-container <pod-id> <image> [-e KEY=VALUE ...] [-- <command> <args>]` | Start a container in an enclave pod, and print its ID. |
| `containers [<pod-id>]` | List your enclave containers. |
| `logs <container-id> [-f] [-n <lines>]` | Show or follow the logs of a container. |
| `exec [-t] <container-id> [-- <command> <args>]` | Run a command or a shell in a container. The default command is `/bin/sh`. |
| `stop-container <container-id> [-t <seconds>]` | Stop a container. |
| `remove-container <container-id>` | Remove a container. |
| `vsock-forward <port> --pod <pod-id> [-l <listen-port>]` | Forward from the parent container group to an enclave pod. |
| `vsock-forward <port> --parent` | Forward from an enclave pod to its parent container group. |
| `list` | List the port forwards that start or end at the caller. |
| `stop-vsock-forward <forward-id>` | Stop a port forward that starts or ends at the caller. |

To see all options of a command, run `hpodctl <command> --help`.

## Related content

* [Confidential containers on Azure Container Instances](container-instances-confidential-overview.md)
* [Tutorial: Prepare a deployment for a confidential container on Azure Container Instances](container-instances-tutorial-deploy-confidential-containers-cce-arm.md)
* [Attestation in confidential containers](confidential-containers-attestation-concepts.md)
* [Azure CLI confcom extension](https://github.com/Azure/azure-cli-extensions/blob/main/src/confcom/azext_confcom/README.md)
