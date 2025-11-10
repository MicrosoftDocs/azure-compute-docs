---
title: Replica soft delete for enhanced data protection in Service Fabric 
description: Learn about the replica soft delete feature for Service Fabric's reliable services.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 11/07/2025
# Customer intent: As a developer, I want to understand how the soft deletion of a replica feature works and what impact it has on the existing replica development model.
---

# Replica soft delete for enhanced data protection in Service Fabric

A [replica of a stateful service](./service-fabric-concepts-replica-lifecycle.md#replicas-of-stateful-services) is a copy of the service logic running on one of the nodes of the cluster. This article introduces a new Service Fabric enhancement - soft delete for stateful replicas. Replica Soft Delete (RSD) provides an additional layer of data protection and operational safety for stateful workloads, helping ensure data integrity and reliability. This feature provides a safeguard against accidental or unintended data loss when using Remove Replica destructive operations on stateful persistent services. Instead of immediately and permanently deleting data, Service Fabric now retains a snapshot of the replica at the time of deletion until the partition regains quorum. RSD operates in the background for every deleted replica.  

RSD also incorporates smart deletion logic, which automatically identifies and permanently removes soft-deleted replicas that are stale if the partition is otherwise healthy. This ensures efficient resource management across the Service Fabric cluster.  

The key benefits of replica soft delete are:

* Operational safety & control: Prevents human error and safeguards against accidental data loss, with restore options and audit logs visibility.
* Data resiliency with minimal storage and compute overhead: Keeps your data protected while eliminating redundant replicas and preventing unnecessary data accumulation.

## Lifecycle of a soft-deleted replica

When stateful service replicas are removed either through the administrative PowerShell API (Remove-ServiceFabricReplica), or directly through the Fabric Client API, Service Fabric will now transition the replicas into a `ToBeRemoved` state. In this new state, the replicas are closed, releasing compute resources. However, SF continues to track these replicas and ensures that replica data on disk isn't cleaned up in case the partition is in quorum loss.  

If removing the replica causes the partition to go into quorum loss, Service Fabric would wait for manual intervention to bring the partition out of quorum loss by recovering the soft deleted replicas.  

Using a new restore replica API (Restore-ServiceFabricReplica), these soft deleted replicas can be recovered to regain quorum without data loss.

If the partition is healthy, the soft deleted replica data receives an automatic cleanup periodically.

Soft deleted replicas don't block repairs, upgrades, or other administrative operations on the service or the cluster. Repairs with data removal intents clean up the replica data as well.

The following diagram shows the flow for replica soft delete:

:::image type="content" source="media/service-fabric-replica-soft-delete/replica-soft-delete-lifecycle.png" alt-text="Diagram showing the process flow for a replica soft delete action." lightbox="media/service-fabric-replica-soft-delete/replica-soft-delete-lifecycle.png":::

## Opt-in process

Replica soft delete is available as an opt-in feature in Service Fabric 11.x releases, starting with SF 11.3. Beginning with SF runtime release 12.0, it's enabled by default for all customers.

Starting with SF 11.3, this behavior can be enabled by setting the `IsDelayedReplicaCleanupEnabled` configuration under the ReconfigurationAgent section in the cluster manifest to be “true”. For example, the following shows a configuration update using an ARM template:  

```json
{ 
  "name": "ReconfigurationAgent",
  "parameters": [
    {
     "name": "IsDelayedReplicaCleanupEnabled", 
     "value": "true" 
    }
  ] 
}
```

## Changes in user experience

Upon removal of stateful service replicas—whether initiated through the administrative PowerShell API (Remove-ServiceFabricReplica), or the Fabric Client API—Service Fabric transitions the affected replicas to the `ToBeRemoved` state.

`ToBeRemoved` replicas can be queried using the existing replica query API. To recover these replicas, a new Restore Replica API (Restore-ServiceFabricReplica) is introduced. This API can be used with the latest SF SDK version, either using PowerShell or FabricClient APIs directly. More details on the API behavior in the sections.  

Even without the new SDK, replicas are still soft-deleted once the feature is enabled, providing safeguards against data loss.

Usually, this behavior change won't impact existing workflows using the Remove Replica API, since SF automatically handles the cleanup within 10 minutes of deleting the replica if the partition is healthy. However, during this short window, you may notice a couple of temporary side effects. Specifically, placements for new replicas of the same partition will be blocked on nodes with soft-deleted replicas. Also, as the soft-deleted replica is cleaned up, the replica process comes back up momentarily to allow a graceful cleanup of disk resources.

> [!NOTE]
> RSD works regardless of whether [Backup and Restore Service (BRS)](service-fabric-reliable-services-backup-restore.md) is enabled or not.

### Newly introduced APIs

* Restore-ServiceFabricReplica:

  * Syntactically similar to [Restart-ServiceFabricReplica](/powershell/module/servicefabric/restart-servicefabricreplica). Recovers a `ToBeRemoved` replica by reopening the Replica object. If the partition was in quorum loss, customers should restore all soft deleted replicas using this API. Service Fabric automatically determines which replicas to retain to restore quorum.

### Changes to the behavior of existing APIs

* [Remove-ServiceFabricReplica](/powershell/module/servicefabric/remove-servicefabricreplica):

  * Without -ForceRemove parameter:

    * Rather than permanently deleting the replica, Remove-ServiceFabricReplica will now soft-delete a replica, provided the replica is responsive.

  * With -ForceRemove parameter:

    * Same as without the parameter, but we just don't wait for the replica to close gracefully.

* [Get-ServiceFabricReplica](/powershell/module/servicefabric/get-servicefabricreplica)

  * The output would indicate what replicas in the partition are `ToBeRemoved`.

:::image type="content" source="media/service-fabric-replica-soft-delete/to-be-removed-status.png" alt-text="Screenshot of sample command line output showing To Be Removed status." lightbox="media/service-fabric-replica-soft-delete/to-be-removed-status.png":::

### Service Fabric Explorer (SFX) changes

SFX now shows `ToBeRemoved` replicas, along with the time by which they get cleaned up permanently.

:::image type="content" source="media/service-fabric-replica-soft-delete/service-fabric-explorer-to-be-removed.png" alt-text="Screenshot of To Be Removed status in Service Fabric Explorer." lightbox="media/service-fabric-replica-soft-delete/service-fabric-explorer-to-be-removed.png":::

## Next steps

* [Reliable Services overview](service-fabric-reliable-services-introduction.md)
* [Reliable Services quickstart](service-fabric-reliable-services-quick-start.md)
* [Reliable collections](service-fabric-reliable-services-reliable-collections.md)
