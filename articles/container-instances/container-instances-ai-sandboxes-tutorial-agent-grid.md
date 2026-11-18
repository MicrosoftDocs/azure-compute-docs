---
title: "Tutorial: Control serverless sandbox outbound traffic with Agent Grid (Public Preview)"
description: In this tutorial, you associate an AI agents group in Azure Container Instances with an Agent Grid resource and apply default and per-sandbox outbound FQDN policies.
ms.topic: tutorial
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/13/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want to restrict outbound network traffic from my serverless sandboxes by using workload-identity-based FQDN policies, so that agents can reach only approved destinations.
---

# Tutorial: Control serverless sandbox outbound traffic with Agent Grid (Public Preview)

By using [Azure Agent Grid](/azure/agent-grid/overview), each [serverless sandbox in Azure Container Instances](container-instances-ai-sandboxes.md) that you enroll in a grid gets a SPIFFE workload identity, and the system denies outbound traffic from the sandbox by default unless a policy group that's bound to the grid allows it. You can apply default policies to all sandboxes in the grid and add policies for individual sandboxes while they run.

In this tutorial, you associate an AI agents group with an Agent Grid resource (`Microsoft.NetworkSecurity/agentFabrics`), apply a default policy that allows `*.bing.com` for all grid members, verify that other destinations are blocked, and then add a policy that allows `*.google.com` for one sandbox's SPIFFE identity.

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Create a default outbound policy group and bind it to an Agent Grid
> * Create an AI agents group associated with the Agent Grid
> * Create a sandbox enrolled in the Agent Grid and test outbound rules
> * Get the sandbox's SPIFFE identity and apply a per-sandbox policy

## Prerequisites

[!INCLUDE [container-instances-ai-sandboxes-prerequisites](./includes/container-instances-ai-sandboxes-prerequisites.md)]

* An [Azure Agent Grid](/azure/agent-grid/configure-agent-grid#create-an-agent-grid) resource (`Microsoft.NetworkSecurity/agentFabrics`) in the same subscription and region where you create your AI agents group, in a region where Agent Grid is available. For supported regions, see [Agent Grid region availability](/azure/agent-grid/overview#region-availability).

## Apply a default outbound policy to the Agent Grid

Policy groups (`Microsoft.NetworkSecurity/policyGroups`) define `FqdnFiltering` rules that match source SPIFFE workload identity patterns and allow outbound traffic to specified FQDNs and ports. The wildcard pattern `spiffe://microsoft.agentfabric/ns/*/sa/*` matches all sandboxes in the grid.

1. Create a default policy group that allows outbound traffic on ports `80` and `443` to `*.bing.com` for all grid members:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/policyGroups/default-egress-policy?api-version=2026-07-21-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "displayName": "Default corporate egress",
        "description": "Allow Bing subdomains for all current and future grid members.",
        "priority": 100,
        "networkSecurityPolicies": [
          {
            "type": "FqdnFiltering",
            "name": "allow-approved-corporate-egress",
            "priority": 10,
            "direction": "Outbound",
            "ports": [80, 443],
            "source": {
              "type": "WorkloadIdPatterns",
              "patterns": [
                "spiffe://microsoft.agentfabric/ns/*/sa/*"
              ]
            },
            "destination": {
              "type": "Fqdns",
              "fqdns": [
                "*.bing.com"
              ]
            }
          }
        ]
      }
    }
    ```

1. Bind the policy group to your Agent Grid resource in `properties.defaultPolicyGroups`:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/agentFabrics/myAgentGrid?api-version=2026-07-21-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "defaultPolicyGroups": [
          "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/policyGroups/default-egress-policy"
        ]
      }
    }
    ```

## Create an AI agents group associated with the Agent Grid

1. Create an AI agents group with `properties.agentFabricResourceId` set to the resource ID of the Agent Grid resource. You must set `agentFabricResourceId` when you create the group, and you can't combine it with `networkProfile`:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myGridAgentsGroup?api-version=2026-08-01-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "agentFabricResourceId": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/agentFabrics/myAgentGrid"
      }
    }
    ```

1. Check the AI agents group until `properties.provisioningState` is `Succeeded`, and then call `connect` to get its `{endpoint}` and `{accessToken}`:

    ```http
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myGridAgentsGroup?api-version=2026-08-01-preview
    ```

    ```http
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myGridAgentsGroup/connect?api-version=2026-08-01-preview
    ```

## Create a sandbox enrolled in the Agent Grid

Create a sandbox in the AI agents group with `properties.attachToAgentFabric` set to `true`. Sandboxes that you create without it don't join the grid, and you can't change `attachToAgentFabric` from `true` to `false` after creation or combine it with [confidential sandboxes](container-instances-ai-sandboxes.md#confidential-sandboxes):

```http
PUT {endpoint}/sandboxes/grid-sandbox1?api-version=2026-08-01-preview
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "location": "centralus",
  "properties": {
    "image": "mcr.microsoft.com/mirror/docker/library/alpine:3.16",
    "attachToAgentFabric": true,
    "resources": {
      "cpu": 1,
      "memory": 2
    }
  }
}
```

Wait until `properties.provisioningState` is `Succeeded`. Then copy `properties.connectionString` from the response, and wait until the guest is ready (`WaitForReady`). By using the [sample C# CLI](container-instances-ai-sandboxes-gateway-proto.md#build-a-c-cli-from-sandboxgatewayproto):

```bash
dotnet run -- -cs "<grid-sandbox1-connection-string>" wait
```

## Verify the default outbound policy

From inside `grid-sandbox1`, test outbound HTTP requests to `www.bing.com`, `www.google.com`, and `example.com`. Use the sample C# CLI:

```bash
dotnet run -- -cs "<grid-sandbox1-connection-string>" exec /bin/sh -c '
for url in http://www.bing.com http://www.google.com http://example.com; do
  out=$(if command -v curl >/dev/null 2>&1; then curl -sSD - -o /dev/null --max-time 15 "$url" 2>&1; else wget -S -O /dev/null -T 15 -t 1 "$url" 2>&1; fi)
  code=$(printf "%s\n" "$out" | sed -n "s/^[[:space:]]*HTTP\/[^[:space:]]*[[:space:]]*\([0-9][0-9][0-9]\).*/\1/p" | head -n 1)
  echo "$url -> HTTP ${code:-BLOCKED}"
done'
```

Because only `*.bing.com` is allowed by the default policy group, the request to `www.bing.com` succeeds with an HTTP `200` status code, while requests to `www.google.com` and `example.com` are blocked:

```output
http://www.bing.com -> HTTP 200
http://www.google.com -> HTTP BLOCKED
http://example.com -> HTTP BLOCKED
```

## Apply a per-sandbox policy by using the sandbox's SPIFFE ID

When a sandbox attaches to an Agent Grid, the grid registers it as a member and assigns it a unique SPIFFE workload identity (`properties.spiffeId`). You can target that exact SPIFFE ID in a policy group so that only that sandbox can reach extra destinations.

1. List the members of the Agent Grid resource and find the entry where `properties.workload.type` is `ACI`, `properties.workload.workloadId` is `grid-sandbox1`, and `properties.workload.resourceId` is the AI agents group resource ID:

    ```http
    GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/agentFabrics/myAgentGrid/members?api-version=2026-07-21-preview
    ```

    Copy the `properties.spiffeId` value for `grid-sandbox1` from the response (for example, `spiffe://microsoft.agentfabric/ns/<namespace>/sa/<service-account>`).

1. Create a policy group that allows `*.google.com` only for that sandbox's `spiffeId`:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/policyGroups/agent-specific-egress-policy?api-version=2026-07-21-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "displayName": "Agent-specific Google egress",
        "description": "Allow this sandbox member to access Google subdomains.",
        "priority": 90,
        "networkSecurityPolicies": [
          {
            "type": "FqdnFiltering",
            "name": "allow-google-egress",
            "priority": 10,
            "direction": "Outbound",
            "ports": [80, 443],
            "source": {
              "type": "WorkloadIdPatterns",
              "patterns": [
                "<sandbox-spiffe-id>"
              ]
            },
            "destination": {
              "type": "Fqdns",
              "fqdns": [
                "*.google.com"
              ]
            }
          }
        ]
      }
    }
    ```

1. Bind the new policy group alongside the default policy group on the Agent Grid resource:

    ```http
    PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/agentFabrics/myAgentGrid?api-version=2026-07-21-preview
    Content-Type: application/json

    {
      "location": "centralus",
      "properties": {
        "defaultPolicyGroups": [
          "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/policyGroups/default-egress-policy",
          "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.NetworkSecurity/policyGroups/agent-specific-egress-policy"
        ]
      }
    }
    ```

1. After the policy update propagates (which can take a few minutes), run the outbound test again inside the running sandbox without restarting or re-creating it. Use the sample C# CLI:

    ```bash
    dotnet run -- -cs "<grid-sandbox1-connection-string>" exec /bin/sh -c '
    for url in http://www.bing.com http://www.google.com http://example.com; do
      out=$(if command -v curl >/dev/null 2>&1; then curl -sSD - -o /dev/null --max-time 15 "$url" 2>&1; else wget -S -O /dev/null -T 15 -t 1 "$url" 2>&1; fi)
      code=$(printf "%s\n" "$out" | sed -n "s/^[[:space:]]*HTTP\/[^[:space:]]*[[:space:]]*\([0-9][0-9][0-9]\).*/\1/p" | head -n 1)
      echo "$url -> HTTP ${code:-BLOCKED}"
    done'
    ```

    Now both `www.bing.com` and `www.google.com` are allowed for `grid-sandbox1`, while `example.com` remains blocked:

    ```output
    http://www.bing.com -> HTTP 200
    http://www.google.com -> HTTP 200
    http://example.com -> HTTP BLOCKED
    ```

## Clean up resources

Delete the sandbox, or delete the AI agents group if you no longer need it:

```http
DELETE {endpoint}/sandboxes/grid-sandbox1?api-version=2026-08-01-preview
Authorization: Bearer {accessToken}
```

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/AIAgentsGroups/myGridAgentsGroup?api-version=2026-08-01-preview
```

For more information about configuring policies, TLS inspection, MCP policies, and flow logs in the Azure portal, see [Configure Azure Agent Grid](/azure/agent-grid/configure-agent-grid).

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Access Azure resources with an agent identity](container-instances-ai-sandboxes-tutorial-agent-identity.md)
