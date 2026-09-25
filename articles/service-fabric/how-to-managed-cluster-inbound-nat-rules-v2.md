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
ai-usage: ai-assisted
# Customer intent: As a cloud architect, I want to configure a Service Fabric managed cluster to use Inbound NAT Rules V2.
---

# Inbound NAT rules V2

> [!NOTE]
> This feature is in preview and might not be fully stable. Try it in a test cluster before you use it in production.

## Recommended reading

Review and understand the difference between [V1 and V2 rules](/azure/load-balancer/inbound-nat-rules). Also read [Azure Load Balancer NAT pool migration guide](/azure/load-balancer/load-balancer-nat-pool-migration); note there's an error in that article. The **Manual Migration** section suggests updating the load balancer *before* the VMSS, but NAT pool deletion fails while the pool is still actively referenced by a VMSS.

## Overview

Azure Load Balancer provides inbound NAT connectivity to individual virtual machine instances in a Service Fabric managed cluster. There are two generations of this mechanism:

- **V1 (Inbound NAT pools):** Each virtual machine network interface is individually mapped to a NAT pool on the load balancer. Azure is deprecating this mechanism.
- **V2 (Inbound NAT rules):** Inbound NAT rules target a backend address pool rather than mapping individual instances through a NAT pool.

> [!IMPORTANT]
> As of early 2026, all newly created Service Fabric managed clusters use V2 rules by default. You can verify this by checking what kind of inbound NAT rules the managed load balancer uses.

## How Service Fabric managed clusters implement V1 and V2

Service Fabric managed clusters use the managed load balancer to provide inbound NAT connectivity (for example, RDP access) to individual virtual machine instances across node types.

### V1 (Legacy)

In V1 mode, SFMC creates **inbound NAT pools** on the managed load balancer. It creates one per node type for the default RDP/SSH port (named `LBBackendNatPool{nodeTypeName}`), plus an extra pool for each `natConfigurations` entry (named `LBBackendNatPool{backendPort}{nodeTypeName}`). Each pool has a unique, non-overlapping frontend port range. The node type's virtual machine scale set references these pools directly, and Azure automatically maps a unique frontend port to each VM instance's backend port.

### V2 (New)

In V2 mode, SFMC creates a **dedicated backend address pool for each node type** on the managed load balancer. These pools are named `LoadBalancerNatPool-{nodeTypeName}`. Each node type's virtual machine instances join their corresponding pool. Unlike V1, where each port requires a separate NAT pool, V2 uses individual **inbound NAT rules** that all target the same per-node-type backend pool. V2 has one rule for the default RDP/SSH port, plus one for each `natConfigurations` entry.

This setup means the VMSS references a single backend pool rather than multiple NAT pools. Port-level configuration is handled entirely through NAT rules on the load balancer. Each node type's instances also remain in the shared `LoadBalancerBEAddressPool` used by load balancer rules (such as Service Fabric gateway endpoints) and outbound rules.

## Migrating from V1 to V2 (Preview)

To migrate an existing V1 cluster to V2, add the `SFRP.UseInboundNatRulesV2` tag with the value `"true"` to the managed cluster resource and submit a cluster update. The migration runs automatically.

When the migration finishes, you can safely remove the tag (set it to `null` or omit it entirely). Don't set the tag back to `"false"`.

> [!IMPORTANT]
> Don't batch other changes with the migration. The cluster update that adds the `SFRP.UseInboundNatRulesV2` tag should be the **only** change in the deployment. Don't combine the migration tag with other cluster property changes or node type modifications in the same update. The migration is a multistep operation that involves rolling VMSS redeployments across all node types. Batching extra unrelated changes increases the likelihood of failures and triggers prolonged NAT connectivity outages. It also makes it harder to diagnose the issue and recover NAT connectivity if the migration fails. A failed update can leave the cluster in a partially migrated state where some nodes lose inbound NAT connectivity. Submit the migration tag as an isolated update, and make any other changes in a separate deployment before or after the migration completes.

> [!NOTE]
> The migration is blocked while any node type is mid-creation. New node type creation is also blocked while the migration is running. Wait for one to complete before starting the other.

## Opting out of V2 for new clusters (not recommended)

All new Service Fabric managed clusters use V2 rules by default. To create a new cluster by using the legacy V1 pools, set the `SFRP.UseInboundNatRulesV2` tag to `"false"` on the cluster resource at creation time. This opt-out only applies during initial cluster creation. It has no effect on existing clusters. In other words, you can't roll back a cluster from V2 to V1.

## What happens during migration

The migration is a fully automated process. You only need to set the tag and submit the cluster update. Service Fabric Resource Provider handles the rest.

### Step 1: Create per-node-type NAT backend pools

The new V2 backend address pools are added to the load balancer alongside the existing V1 pools. No connectivity changes occur at this point. Existing inbound NAT access continues to work through the V1 configuration.

### Step 2: Migrate VMSS to V2 backend pools

Update each node type's virtual machine scale set to remove the V1 pool references and add the per-node-type V2 NAT backend pool (`LoadBalancerNatPool-{nodeTypeName}`). This update requires a rolling redeploy of each VMSS. The migration process automatically skips node types that already use V2 rules.

You lose inbound NAT connectivity for each node type as you update it. When you remove the V1 pool references from a VMSS, you don't restore connectivity until you add V2 rules to the load balancer in Step 3.

Because Step 3 runs only after all node types complete Step 2, there's a window where **all** node types lose inbound NAT connectivity simultaneously. This window lasts from the completion of the last VMSS redeploy in Step 2 until Step 3 finishes adding V2 rules to the load balancer. Application traffic isn't affected during this window; only management access (RDP/SSH) through inbound NAT is interrupted. Load balancer rules for Service Fabric gateway endpoints and outbound rules continue to function normally.

### Step 3: Replace V1 pools with V2 rules

Remove the V1 pool definitions from the load balancer and replace them with V2 rules that target the per-node-type backend pools you created in Step 1. This step restores inbound NAT connectivity for all migrated node types.

After you complete this step, the migration is complete and the cluster is fully running on V2 rules.

## Rollback

Rollback from V2 to V1 is **not supported**. The migration is designed to move forward only. If the migration encounters a transient failure partway through, retry the migration. Don't attempt a manual rollback.

## Failure handling

The migration doesn't automatically revert on failure. Instead, it preserves the progress made so far and expects you to re-submit the same cluster update to resume. You can safely retry each step.

### What to do if migration fails

| Failure point | Impact |
| ------------- | ------ |
| Step 1 failed (creating per-node-type NAT backend pools) | No impact; V1 pools remain intact and no VMSS changes were made. |
| Step 2 failed (migrating VMSS to V2 backend pools) | Inbound NAT connectivity is lost on nodes mid-migration or that are already migrated. Their V1 pool references are removed but V2 rules don't exist on the load balancer yet. Nodes that you didn't update yet still have inbound NAT connectivity through V1. Re-submit the cluster update to resume. SFMC tracks which nodes are already migrated and skips them on retry. |
| Step 3 failed (replacing V1 pools with V2 rules) | Inbound NAT connectivity is lost on all migrated node types (expected state after Step 2). V1 pool references are removed but V2 rules weren't successfully added to the load balancer. This step shouldn't fail. |

## Bring-your-own load balancer (BYOLB) and additional NIC considerations

The SFMC NAT V2 migration only applies to the **managed** load balancer. SFMC doesn't migrate custom load balancers. If you use custom load balancers, migrate those **first** before migrating the managed load balancer.

### Migration scope

SFMC doesn't modify custom load balancer resources during migration. This behavior applies to both external and internal load balancers.

> [!IMPORTANT]
> Step 3 of the migration process fails if a VMSS still references a V1 pool on the managed load balancer. This condition can occur on node types that specify `frontendConfigurations` or `additionalNetworkInterfaceConfigurations`. To avoid this problem, ensure the `loadBalancerInboundNatPools` property isn't specified on any node type before you perform the SFMC NAT V2 migration.
>

> [!IMPORTANT]
> You can't delete inbound NAT pools (V1) if a VMSS still references the pool. The migration process removes inbound NAT pool references from the VMSS before updating the load balancer.

### Node type using custom load balancer only (no managed load balancer)

If a node type is only associated with a custom load balancer, the migration skips patching the node type. Since the node type isn't using any inbound NAT pools from the managed load balancer, no migration is needed.

### Node type using both custom and managed load balancer

A node type can be associated with both a custom load balancer and the managed load balancer (in this case, it must be an internal load balancer). In this scenario, the node type can hold references to V1 pools from **both** the custom and managed load balancers. To make the node type use only V2 rules, you must migrate both load balancers' inbound NAT pools. Migrate the inbound NAT rules for the custom load balancers **first**, then perform the SFMC NAT V2 migration for the managed load balancer. If you migrate one load balancer but not the other, the node type is accessible by using V1 pools from the unmigrated load balancer and by using V2 rules from the migrated load balancer.

### Node type using extra NICs

The migration only modifies the primary network interface on each node type. Additional NICs are not affected during migration; their IP configurations, including any backend address pool or V1 pool references, are preserved exactly as specified by customers. If your node type ARM template specifies the `loadBalancerInboundNatPoolId` property anywhere inside the `additionalNetworkInterfaceConfigurations` property, then you should migrate those referenced load balancers to use V2 rules before performing the SFMC NAT V2 migration.

## Custom load balancer references

You can specify custom load balancer references on a node type, including V1 pool references, by using both `frontendConfigurations` (BYOLB) and `additionalNetworkInterfaceConfigurations` node type properties. The migration process doesn't modify any of these references. You can perform the SFMC NAT V2 migration first and migrate those later, but this approach is discouraged.

## Recommended action for customers using BYOLB or additional NICs

If any node types reference V1 pools from a custom load balancer, migrate them to V2 rules before the SFMC NAT V2 migration.

Example: node type references a V1 pool from a custom load balancer

```json
{
  "type": "Microsoft.ServiceFabric/managedClusters/nodeTypes",
  "properties": {
    "frontendConfigurations": [
      {
        "loadBalancerInboundNatPoolId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sfmc/providers/Microsoft.Network/loadBalancers/customLB/inboundNatPools/LoadBalancerNATPool",
        "loadBalancerBackendAddressPoolId": "..."
      }
    ]
  }
}
```

If the node type specifies `loadBalancerInboundNatPoolId`, migrate the custom load balancer to use V2 rules:

1. Remove the node type `loadBalancerInboundNatPoolId` references.

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

   After this step, there shouldn't be any references to `loadBalancerInboundNatPoolId` on the node type.

1. Verify V1 pools are absent on the custom load balancer.

   Check the custom load balancer and ensure that the inbound NAT pools and rules don't reference any virtual machine instance. In other words, the load balancer NAT pools should be fully disconnected from any VMSS.

1. Swap `inboundNatPools` for `inboundNatRules` on the custom load balancer.

1. Verify inbound NAT connectivity is restored with V2.

   After this step, you can begin the SFMC NAT V2 migration.

### Migration summary for BYOLB and additional NIC scenarios

1. **Custom NAT pool references.** For each node type, review the `loadBalancerInboundNatPools` entries on both `frontendConfigurations` and `additionalNetworkInterfaceConfigurations`. Note which load balancers and NAT pools are referenced.
1. **Migrate custom load balancers first.** For any referenced load balancer that still uses V1 pools, migrate it to use V2 rules.
1. **Trigger the SFMC NAT V2 migration.** Once all custom load balancers and associated node types use V2 rules, set the `SFRP.UseInboundNatRulesV2` tag and submit the cluster update.
