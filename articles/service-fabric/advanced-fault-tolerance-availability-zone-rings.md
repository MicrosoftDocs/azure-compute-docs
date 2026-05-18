---
title: Advanced Fault Tolerance for Cross-Availability-Zone Rings (On-Demand Auxiliary Replicas)
description: Learn how on-demand auxiliary replicas reduce quorum-loss risk in two-availability-zone Service Fabric deployments during planned maintenance by utilizing a third availability zone.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 05/18/2026
---

# Advanced Fault Tolerance for Cross-Availability-Zone Rings (On-Demand Auxiliary Replicas)

This article describes an advanced fault tolerance capability for full-service applications in two availability zones (AZs) by utilizing a third AZ.

> [!IMPORTANT]
> This feature requires a three-zone cluster layout:
>   * Two large zones for regular service workload.
>   * One small zone for seed-node quorum support.

On-demand auxiliary replicas help protect write availability during planned maintenance in two-zone application deployments by temporarily preserving safer replica distribution.

## Overview

In 2-AZ deployments, planned operations such as application upgrades, cluster upgrades, and node deactivations can temporarily remove one replica from a partition.

If a zone outage happens during that window, the partition can drop below write quorum and enter quorum loss.

The availability zone ring feature reduces this risk by creating a temporary auxiliary replica during planned maintenance and removing it after the original replica returns.

## What is an auxiliary replica?

Auxiliary replicas are specialized replicas intended for advanced replicator-level scenarios that are outside the normal roles of `Primary`, `IdleSecondary`, and `ActiveSecondary`.

Key behavior:

* Auxiliary replicas are not granted read or write access.
* Service Fabric does not consider auxiliary replicas as primary-election targets during balancing or failover.
* They can observe replication for scenarios such as auditing, monitoring, or transient replication-side processing.

This capability is intended for advanced workloads and can require corresponding application replicator changes.

## Why use auxiliary replicas?

Without this feature, a planned replica-down event plus a zone failure can force a partition into quorum loss.

With this feature enabled, Service Fabric creates a temporary auxiliary replica in the same availability zone as the replica that's planned to be down before the replica-down event. This auxiliary replica can preserve quorum in that same failure window.

Benefits:

* Improves availability during planned maintenance in 2-AZ application deployments.
* Helps maintain safer cross-zone write quorum distribution.
* Enables a more capacity-optimized three-zone topology for customers who do not want full workload capacity in all zones.
* Adds no steady-state replica overhead because auxiliary replicas are temporary.

## Example failure scenario

Scenario: Availability Zone 1 becomes unavailable while a planned replica-down event is in progress in Availability Zone 2.

Without on-demand auxiliary replicas:

* One regular replica is already offline for maintenance.
* A zone outage removes additional replicas.
* The partition can enter quorum loss.

With on-demand auxiliary replicas:

* Service Fabric creates an auxiliary replica before the planned replica-down event.
* If the zone outage occurs in the same window, an additional replica remains available.
* The partition is more likely to preserve write quorum and recover automatically.

> [!IMPORTANT]
> Protection is expected only when replicas are healthy before the outage window. If a replica is already unexpectedly down before a zone failure, quorum loss can still occur.

## Prerequisites

* Your application code *must* support auxiliary replicas (custom `IReplicator` scenarios).
* Service Fabric 11.5 or later.
* Service Fabric classic (SFRP) cluster topology across three zones:
    * Two large zones (for example, Zone 1 and Zone 2) host regular service replicas and seed nodes.
    * One small zone (for example, Zone 3) hosts seed nodes only.

    :::image type="content" source="media/advanced-fault-tolerance-availability-zone-rings/topology.png" alt-text="Diagram of the cluster topology required to utilize the advanced fault tolerance on-demand auxiliary replicas feature.":::

## Configure on-demand auxiliary replicas

### Step 1: Enable cluster-level fabric settings

Add these settings to the cluster fabric configuration:

| Section | Parameter | Value |
| --- | --- | --- |
| `Common` | `EnableAuxiliaryReplicas` | `true` |
| `PlacementAndLoadBalancing` | `OnDemandReplicaConstraintPriority` | `0` |

Example snippet:

```json
"fabricSettings": [
  {
    "name": "Common",
    "parameters": [
      {
        "name": "EnableAuxiliaryReplicas",
        "value": "true"
      }
    ]
  },
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
        "name": "OnDemandReplicaConstraintPriority",
        "value": "0"
      }
    ]
  }
]
```

### Step 2: Configure each target stateful service

Use the following service settings:

| Setting | Value |
| --- | --- |
| Target replica set size | Minimum `4` |
| Minimum replica set size | `3` |
| Enable on-demand auxiliary replica | `true` |
| Has persisted state | `true` |

To keep user services off the seed-only zone, use a placement constraint such as `NodeTypeName != ntSeed`.

### Step 3: Verify runtime behavior

After deployment:
1. Start a rolling upgrade or node deactivation.
1. Observe partition replica set changes.
1. Confirm an auxiliary replica appears before the planned replica-down event.
1. Confirm the auxiliary replica is removed after maintenance completes.

## Next steps

* Review [Service Fabric cluster reliability characteristics](service-fabric-cluster-capacity.md).
* Review [Service Fabric best practices](./service-fabric-best-practices-security.md).
