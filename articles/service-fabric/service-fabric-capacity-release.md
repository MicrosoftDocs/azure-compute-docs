---
title: Release Service Fabric capacity during severe capacity loss
description: Learn how to reduce selected Service Fabric services so critical workloads can use surviving cluster capacity during a severe capacity loss.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 08/13/2026
ai-usage: ai-assisted
# Customer intent: As a Service Fabric cluster operator, I want to release capacity from selected lower-priority services during a severe capacity loss, so that critical workloads can use the surviving capacity.
---

# Release Service Fabric capacity during severe capacity loss

Capacity release is a last-resort, operator-controlled mitigation for severe capacity loss, such as a zonal outage. It reduces selected lower-priority services so that critical workloads can use the surviving cluster capacity. Service Fabric never activates capacity release automatically.

> [!WARNING]
> Capacity release can reduce redundancy, make services unavailable, and cause permanent data loss. Use it only after other mitigations can't provide enough capacity for critical workloads.

This article explains how to prepare services for capacity release, select a cluster-wide capacity-release level, verify the active level, and return the cluster to normal operation.

## Capacity-release actions and levels

Each service has a `CapacityReleaseAction` setting. The action determines how Service Fabric changes the service when you select a cluster-wide capacity-release level:

- `None` is the default. Service Fabric keeps the service at its configured target at every capacity-release level.
- `DropToMin` keeps the service at its configured target at the `None` and `Minor` levels, and reduces it to its configured minimum at the `Major` level.
- `DropToZero` keeps the service at its configured target at the `None` level, reduces it to its configured minimum at the `Minor` level, and reduces it to zero at the `Major` level.

For a stateful service, the configured target is `TargetReplicaSetSize`, and the configured minimum is `MinReplicaSetSize`. For a stateless service, the configured target is `InstanceCount`, and the configured minimum is `MinInstanceCount`.

The following table summarizes the behavior.

| `CapacityReleaseAction` | `None` level | `Minor` level | `Major` level |
|---|---:|---:|---:|
| `None` | Configured target | Configured target | Configured target |
| `DropToMin` | Configured target | Configured target | Configured minimum |
| `DropToZero` | Configured target | Configured minimum | Zero |

The cluster-wide levels are:

- `None`: Normal operation. Services run at their configured targets.
- `Minor`: Releases capacity from services configured with `DropToZero` by reducing them to their configured minimum.
- `Major`: Releases the most capacity. Services configured with `DropToMin` run at their configured minimum, and services configured with `DropToZero` run at zero.

You can transition directly between any levels. For example, you can change from `None` directly to `Major`, or from `Major` directly to `None`.

> [!WARNING]
> Reducing a stateful service to its minimum increases the risk of quorum loss. Reducing a stateful service to zero permanently removes its state. Returning the cluster to `None` doesn't restore lost state.

For example, consider a stateful service with `TargetReplicaSetSize=5`, `MinReplicaSetSize=3`, and `CapacityReleaseAction=DropToZero`. Its target is 5 replicas at `None`, 3 replicas at `Minor`, and 0 replicas at `Major`.

## Prepare the cluster and services

Before an outage, decide which lower-priority services can release capacity and how far each service can safely reduce. Keep critical services configured with `CapacityReleaseAction=None`.

Enable capacity-release operations by setting the following dynamic cluster setting:

```xml
<Section Name="FailoverManager">
  <Parameter Name="EnableCapacityReleaseReplicaDrop" Value="true" />
</Section>
```

The setting is off by default. If `EnableCapacityReleaseReplicaDrop` isn't `true`, capacity-release operations return `OperationNotSupported`.

Configure `CapacityReleaseAction` for each service that can release capacity:

- Use `DropToMin` when the service can run at its configured minimum during a major capacity loss.
- Use `DropToZero` only when the service can run at its configured minimum during a minor capacity loss and can be removed during a major capacity loss.
- Keep `None` for services that shouldn't release capacity.

For stateful services, choose `MinReplicaSetSize` with quorum and availability requirements in mind. Back up stateful services according to your recovery requirements. Capacity release isn't a replacement for backup or disaster recovery.

## Compare projected capacity

Capacity-release estimation is planned for the target release but isn't available until the estimation interfaces are implemented and ready. Don't depend on estimation until your Service Fabric release documentation identifies it as available.

When available, use estimation before changing the level to compare projected capacity for `None`, `Minor`, and `Major`. Each eligible unique metric has exactly three entries, one for each level.

Use one of these interfaces to request an estimate:

### PowerShell

```powershell
Get-ServiceFabricCapacityReleaseEstimation
```

### .NET

Call `FabricClient.QueryManager.GetCapacityReleaseEstimationAsync`.

### REST

```http
GET https://{cluster-endpoint}:19080/$/GetCapacityReleaseEstimation?api-version={api-version}
```

## Set the capacity-release level

Select the least disruptive level that releases enough capacity for critical workloads.

### PowerShell

Set the level to `Minor`:

```powershell
Set-ServiceFabricCapacityReleaseLevel -Level Minor
```

Set the level to `Major`:

```powershell
Set-ServiceFabricCapacityReleaseLevel -Level Major
```

Return to normal operation:

```powershell
Set-ServiceFabricCapacityReleaseLevel -Level None
```

### .NET

Call `FabricClient.FaultManager.SetCapacityReleaseLevelAsync` with `Minor`, `Major`, or `None`.

### REST

Set the level by using the `Level` query parameter:

```http
POST https://{cluster-endpoint}:19080/$/SetCapacityReleaseLevel?Level=Minor&api-version={api-version}
```

Replace `Minor` with `Major` or `None` as needed.

Services created while a capacity-release level is active are affected by the current level according to their configured `CapacityReleaseAction`.

## Verify the active level

Verify the active cluster capacity-release level.

### PowerShell

```powershell
Get-ServiceFabricCapacityReleaseLevel
```

### .NET

Call `FabricClient.QueryManager.GetCapacityReleaseLevelAsync`.

### REST

```http
GET https://{cluster-endpoint}:19080/$/GetCapacityReleaseLevel?api-version={api-version}
```

## Return to normal operation

Set the capacity-release level to `None`. Service Fabric then works toward each service's configured target, subject to available cluster capacity.

State that was permanently removed when a stateful service reached zero isn't restored. Recover or recreate the service according to your application's recovery plan.

## Respond to a Failover Manager partition repair

A Failover Manager partition repair resets the capacity-release level to `None`. After a repair, query the active level and reapply `Minor` or `Major` if the cluster still needs capacity release.

A service that had zero replicas during the repair might need to be deleted and recreated.

## Limitations

Capacity release:

- Doesn't activate automatically.
- Doesn't determine the business priority of services.
- Doesn't guarantee placement priority for critical workloads.
- Doesn't guarantee recovery after all replicas of a stateful service are removed.
- Isn't a replacement for backup or disaster recovery.
- Doesn't provide additional outage mitigation beyond releasing capacity from selected services.
