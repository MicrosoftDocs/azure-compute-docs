---
title: How to configure Inbound NAT Rules V2 for Azure Service Fabric managed clusters
description: Learn how to configure an Azure Service Fabric managed cluster to use Inbound NAT Rules V2
ms.topic: how-to
ms.author: davidzhang
author: davidzhang
ms.service: azure-service-fabric
ms.custom: devx-track-arm-template
services: service-fabric
ms.date: 07/07/2026
# Customer intent: "As a cloud architect, I want to configure a Service Fabric managed cluster to use Inbound NAT Rules V2.
---

# Inbound NAT Rules V2


> [!NOTE]
> This feature is still in preview and may not be fully stable, it is recommended to attempt it in a test cluster first before production.

## Recommended Reading

Readers should first review and understand the difference between [V1 and V2 rules](https://learn.microsoft.com/en-us/azure/load-balancer/inbound-nat-rules). Also read [Azure Load Balancer NAT pool migration guide](https://learn.microsoft.com/azure/load-balancer/load-balancer-nat-pool-migration) before proceeding; note there is an error in that document, the section "Manual Migration" suggests that the load balancer is updated *before* the VMSS. This is incorrect as NAT pool deletion will fail as the pool is still actively referenced by a VMSS.

## Overview

Azure Load Balancer provides inbound NAT connectivity to individual virtual machine instances in a Service Fabric managed cluster. There are two generations of this mechanism:

- **V1 (Inbound NAT Pools):** Each virtual machine network interface is individually mapped to a NAT pool on the load balancer. Azure is deprecating this mechanism.
- **V2 (Inbound NAT Rules):** Inbound NAT rules target a backend address pool rather than mapping individual instances through a NAT pool.

> [!IMPORTANT] As of early 2026, all newly created Service Fabric managed clusters use V2 rules by default. This can be verified by checking what kind of Inbound NAT rules are being used by the managed load balancer.

## How Service Fabric managed clusters implement V1 and V2

Service Fabric managed clusters use the managed load balancer to provide inbound NAT connectivity (e.g., RDP access) to individual virtual machine instances across node types.

### V1 (Legacy)

In V1 mode, SFMC creates **inbound NAT pools** on the managed load balancer; one per node type for the default RDP/SSH port (named `LBBackendNatPool{nodeTypeName}`), plus an additional pool for each `natConfigurations` entry (named `LBBackendNatPool{backendPort}{nodeTypeName}`). Each pool has a unique, non-overlapping frontend port range. The node type's virtual machine scale set references these pools directly, and Azure automatically maps a unique frontend port to each VM instance's backend port.

### V2 (New)

In V2 mode, SFMC creates a **dedicated backend address pool per node type** on the managed load balancer, named `LoadBalancerNatPool-{nodeTypeName}`. Each node type's virtual machine instances are added to their corresponding pool. Unlike V1, where each port requires a separate NAT pool, V2 uses individual **inbound NAT rules** that all target the same per-node-type backend pool; one rule for the default RDP/SSH port, plus one for each `natConfigurations` entry.

This means the VMSS references a single backend pool rather than multiple NAT pools, and port-level configuration is handled entirely through NAT rules on the load balancer. Each node type's instances also remain in the shared `LoadBalancerBEAddressPool` used by load balancer rules (e.g., Service Fabric gateway endpoints) and outbound rules.

## Migrating from V1 to V2 (Preview)

To migrate an existing V1 cluster to V2, add the `SFRP.UseInboundNatRulesV2` tag with value `"true"` to the managed cluster resource and submit a cluster update. The migration runs automatically.

Once the migration is complete, the tag can be safely removed (set to `null` or omit it entirely). Do not set the tag back to `"false"`.

> [!IMPORTANT] Do not batch other changes with the migration. The cluster update that adds the `SFRP.UseInboundNatRulesV2` tag should be the **only** change in the deployment. Do not combine the migration tag with other cluster property changes or node type modifications in the same update. The migration is a multi-step operation that involves rolling VMSS redeployments across all node types; batching additional unrelated changes increases the likelihood of failures and triggering prolonged NAT connectivity outages. It also makes it harder to diagnose the issue and recover NAT connectivity if the migration fails. A failed update can leave the cluster in a partially migrated state where some nodes have lost inbound NAT connectivity. Submit the migration tag as an isolated update, and make any other changes in a separate deployment before or after the migration completes.

> [!NOTE]
> The migration is blocked while any node type is mid-creation. New node type creation is also blocked while the migration is running. Wait for one to complete before starting the other.

## Opting Out of V2 for New Clusters (Not Recommended)

All new Service Fabric managed clusters use V2 rules by default. To create a new cluster using the legacy V1 pools, set the `SFRP.UseInboundNatRulesV2` tag to `"false"` on the cluster resource at creation time. This opt-out only applies during initial cluster creation; it has no effect on existing clusters. In other words, you cannot rollback a cluster from V2 back to V1.

## What Happens During Migration

The migration is a fully automated process. Customers only need to set the tag and submit the cluster update; Service Fabric Resource Provider handles the rest.

### Step 1: Create per-node-type NAT backend pools

The new V2 backend address pools are added to the load balancer alongside the existing V1 pools. No connectivity changes occur at this point; existing inbound NAT access continues to work through the V1 configuration.

### Step 2: Migrate VMSS to V2 backend pools

Each node type's virtual machine scale set is updated to remove the V1 pool references and add the per-node-type V2 NAT backend pool (`LoadBalancerNatPool-{nodeTypeName}`). This involves a rolling redeploy of each VMSS. Node types that are already on V2 rules are automatically skipped.

Inbound NAT connectivity is lost for each node type as it is updated; once the V1 pool references are removed from a VMSS, connectivity is not restored until V2 rules are added to the load balancer in Step 3.

Because Step 3 only runs after all node types have completed Step 2, there is a window where **all** node types have lost inbound NAT connectivity simultaneously. This window lasts from the completion of the last VMSS redeploy in Step 2 until Step 3 finishes adding V2 rules to the load balancer. Application traffic is **not** affected during this window; only management access (RDP/SSH) through inbound NAT is interrupted. Load balancer rules for Service Fabric gateway endpoints and outbound rules continue to function normally.

### Step 3: Replace V1 pools with V2 rules

The V1 pool definitions are removed from the load balancer and replaced with V2 rules targeting the per-node-type backend pools created in Step 1. This restores inbound NAT connectivity for all migrated node types.

After this step, the migration is complete and the cluster is fully running on V2 rules.

## Rollback

Rollback from V2 to V1 is **not supported**. The migration is designed to move forward only. If the migration encounters a transient failure partway through, the recommended action is to retry, not to attempt a manual rollback.

## Failure Handling

The migration does **not** automatically revert on failure. Instead, it preserves the progress made so far and expects customers to re-submit the same cluster update to resume. Each step is designed to be safely retried.

### What to Do If Migration Fails

| Failure Point | Impact |
|---------------|--------|
| Step 1 failed (creating per-node-type NAT backend pools) | No impact; V1 pools remain intact and no VMSS changes were made. |
| Step 2 failed (migrating VMSS to V2 backend pools) | Inbound NAT connectivity is lost on nodes mid-migration or that are already migrated; their V1 pool references have been removed but V2 rules do not exist on the load balancer yet. Nodes that haven't been updated yet still have inbound NAT connectivity through V1. Re-submit the cluster update to resume; SFMC tracks which nodes have already been migrated and skips them on retry. |
| Step 3 failed (replacing V1 pools with V2 rules) | Inbound NAT connectivity is lost on all migrated node types (expected state after Step 2); V1 pool references have been removed but V2 rules were not successfully added to the load balancer. This step should not fail. |

## Bring-Your-Own Load Balancer (BYOLB) and Additional NIC Considerations

The SFMC NAT V2 migration only applies to the **managed** load balancer. SFMC does not migrate custom load balancers. Customers using custom load balancers are recommended to migrate those **first** before migrating the managed load balancer.

### Migration Scope

SFMC does not modify custom load balancer resources during migration. This is true for both external and internal load balancers.

> [!IMPORTANT] Step 3 of the migration process will fail if a VMSS still references a V1 pool on the managed load balancer. This can occur on node types specifying `frontendConfigurations` or `additionalNetworkInterfaceConfigurations`. To avoid this, ensure the `loadBalancerInboundNatPools` property is not specified on any node type before performing the SFMC NAT V2 migration.

> [!IMPORTANT] Inbound NAT pools (V1) cannot be deleted if a VMSS still references the pool; this is why inbound NAT pool references are removed from the VMSS before updating the load balancer.

### Node type using custom load balancer only (no managed load balancer)

If a node type is only associated with a custom load balancer, the migration skips patching the node type. Since the node type is not using any inbound NAT pools from the managed load balancer, there is no migration needed.

### Node type using both custom and managed load balancer

A node type may be associated with both a custom load balancer and the managed load balancer (in such case, it must be an internal load balancer). In this scenario, the node type can hold references to V1 pools from **both** the custom and managed load balancer. In order for the node type to only use V2 rules, both load balancer's inbound NAT pools must be migrated. It is strongly recommended that you perform the inbound NAT rules migration for the custom load balancers **first**; *then* then the SFMC NAT V2 migration (managed load balancer). In the scenario where one load balancer is migrated and the other is not, the node type will be accessible with V1 pools from the unmigrated load balancer, and with V2 rules from the migrated load balancer.

### Node type using additional NICs

The migration only modifies the primary network interface on each node type. Additional NICs are not affected during migration; their IP configurations, including any backend address pool or V1 pool references, are preserved exactly as specified by customers. If your node type ARM template specifies the `loadBalancerInboundNatPoolId` property anywhere inside the `additionalNetworkInterfaceConfigurations` property, then you should migrate those referenced load balancers to use V2 rules before performing the SFMC NAT V2 migration.

## Custom Load Balancer References

Both `frontendConfigurations` (BYOLB) and `additionalNetworkInterfaceConfigurations` node type properties allow customers to specify custom load balancer references on a node type, including V1 pool references. The migration does **not** modify any of these references. Customers *can* perform the SFMC NAT V2 migration first and migrate those later, but this is discouraged.

## Recommended Action for Customers Using BYOLB or Additional NICs

If any node types reference V1 pools from a custom load balancer, customers should migrate them to V2 rules before the SFMC NAT V2 migration.

Example: node type references a V1 pool from a custom load balancer

```json
{
  "type": "Microsoft.ServiceFabric/managedClusters/nodeTypes",
  "properties": {
    "frontendConfigurations": [
      {
        "loadBalancerInboundNatPoolId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sfmc/providers/Microsoft.Network/loadBalancers/customLB/inboundNatPools/LoadBalancerNATPool",
        "loadBalancerBackendAddressPoolId": "...",
      }
    ]
  }
}
```

If the node type specifies `loadBalancerInboundNatPoolId`, customers must migrate the custom load balancer to use V2 rules:

1. Remove the node type `loadBalancerInboundNatPoolId` reference(s).

```json
{
  "type": "Microsoft.ServiceFabric/managedClusters/nodeTypes",
  "properties": {
    "frontendConfigurations": [
      {
        "loadBalancerBackendAddressPoolId": "..."
      }
    ]
  }
}
```

After this step, there should not be any references to `loadBalancerInboundNatPoolId` on the node type.

2. Verify V1 pools absent on the custom load balancer

Check the custom load balancer and ensure that the inbound NAT pools/rules do not reference any virtual machine instance. In other words, the load balancer NAT pools should be fully disconnected from any VMSS.

3. Swap `inboundNatPools` for `inboundNatRules` on the custom load balancer.

4. Verify inbound NAT connectivity restored with V2

After this step, customers may begin the SFMC NAT V2 migration.

### Migration summary for BYOLB and additional NIC scenario's

1. **Custom NAT pool references.** For each node type, review the `loadBalancerInboundNatPools` entries on both `frontendConfigurations` and `additionalNetworkInterfaceConfigurations`. Note which load balancers and NAT pools are referenced.
2. **Migrate custom load balancers first.** For any referenced load balancer that still uses V1 pools, migrate it to use V2 rules.
3. **Trigger the SFMC NAT V2 migration.** Once all custom load balancers and associated node types have been migrated to use V2 rules, it is safe to set the `SFRP.UseInboundNatRulesV2` tag and submit the cluster update.
