---
title: Scale to Zero - Disable and enable services in Azure Service Fabric
description: Learn how to disable and enable services in Azure Service Fabric to free cluster resources without deleting service metadata.
ms.topic: concept-article
ms.author: vpavlovic
author: vpavlovic
ms.service: azure-service-fabric
ms.date: 03/13/2026
---

# Scale to Zero - Disable and enable services in Azure Service Fabric

To increase resource utilization in a cluster, scale to zero feature can be used to free resources that are being held by services not currently in use. In this article, you learn how to disable and enable services in an Azure Service Fabric cluster. Disabling a service removes all of its replicas and instances while preserving the service definition. When you enable the service again, Service Fabric rebuilds the replicas without requiring a full re-creation.

Both stateful and stateless services are supported on Windows and Linux platforms.

## Prerequisites

- A Service Fabric cluster running version 11.5 or later.
- The `AllowDisableEnableService` configuration flag set to `true` in the [**FailoverManager**][fabric-settings-link] section:

  ```xml
  <Section Name="FailoverManager">
    <Parameter Name="AllowDisableEnableService" Value="true" />
  </Section>
  ```

## Disable a service

When you disable a service, Service Fabric marks it as **Disabling**, schedules replica removal, and returns immediately. The service transitions to **Disabled** asynchronously after all replicas are removed. Poll the service status to confirm completion.

#### [PowerShell](#tab/disable-ps)

```powershell
Disable-ServiceFabricService -ServiceName "fabric:/MyApp/MyService"
```

To force-disable a service and bypass graceful replica shutdown:

```powershell
Disable-ServiceFabricService -ServiceName "fabric:/MyApp/MyService" -ForceDisable
```

#### [C#](#tab/disable-csharp)

```csharp
var client = new FabricClient();
var description = new DisableServiceDescription(new Uri("fabric:/MyApp/MyService"))
{
    DisableServiceFlag = DisableServiceFlag.RemoveData
};

await client.ServiceManager.DisableServiceAsync(description);
```

To force-disable:

```csharp
var description = new DisableServiceDescription(new Uri("fabric:/MyApp/MyService"))
{
    DisableServiceFlag = DisableServiceFlag.RemoveData,
    ForceDisable = true
};

await client.ServiceManager.DisableServiceAsync(description);
```

#### [REST API](#tab/disable-rest)

```http
POST /Services/{serviceName}/$/Disable

{
    "DisableServiceFlag": "RemoveData"
}
```

To force-disable, include the `ForceDisable` property:

```http
POST /Services/{serviceName}/$/Disable

{
    "DisableServiceFlag": "RemoveData",
    "ForceDisable": true
}
```

---

> [!CAUTION]
> `ForceDisable` on stateful services can leave persisted state on disk that isn't properly cleaned up, because replicas are terminated without a graceful shutdown.

## Enable a service

When you enable a previously disabled service, Service Fabric builds replicas and instances according to the service's current configuration (target replica set size, instance count, placement constraints, and so on).

#### [PowerShell](#tab/enable-ps)

```powershell
Enable-ServiceFabricService -ServiceName "fabric:/MyApp/MyService"
```

#### [C#](#tab/enable-csharp)

```csharp
var client = new FabricClient();
await client.ServiceManager.EnableServiceAsync(new Uri("fabric:/MyApp/MyService"));
```

#### [REST API](#tab/enable-rest)

```http
POST /Services/{serviceName}/$/Enable
```

---

## Create a service in the disabled state

You can register a service in the cluster without placing any replicas or instances. The service stays disabled until you explicitly enable it.

#### [PowerShell](#tab/create-ps)

Create a stateless service as disabled:

```powershell
New-ServiceFabricService `
    -ApplicationName "fabric:/MyApp" `
    -ServiceName "fabric:/MyApp/MyService" `
    -ServiceTypeName "MyServiceType" `
    -Stateless `
    -InstanceCount 3 `
    -PartitionSchemeSingleton `
    -IsCreateAsDisabled
```

Create a stateful service as disabled:

```powershell
New-ServiceFabricService `
    -ApplicationName "fabric:/MyApp" `
    -ServiceName "fabric:/MyApp/MyStatefulService" `
    -ServiceTypeName "MyStatefulServiceType" `
    -Stateful `
    -HasPersistedState `
    -TargetReplicaSetSize 3 `
    -MinReplicaSetSize 2 `
    -PartitionSchemeSingleton `
    -IsCreateAsDisabled
```

#### [C#](#tab/create-csharp)

```csharp
var description = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/MyApp"),
    ServiceName = new Uri("fabric:/MyApp/MyService"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = 3,
    IsCreateAsDisabled = true
};

var client = new FabricClient();
await client.ServiceManager.CreateServiceAsync(description);
```

---

When you omit the `-IsCreateAsDisabled` flag, the default value is `false` and Service Fabric places replicas immediately upon creation.

## Check service status

Query the service to verify its current state.

#### [PowerShell](#tab/status-ps)

```powershell
Get-ServiceFabricService -ApplicationName "fabric:/MyApp" -ServiceName "fabric:/MyApp/MyService"
```

#### [C#](#tab/status-csharp)

```csharp
var client = new FabricClient();
var services = await client.QueryManager.GetServiceListAsync(
    new Uri("fabric:/MyApp"),
    new Uri("fabric:/MyApp/MyService"));

foreach (var svc in services)
{
    Console.WriteLine($"Service: {svc.ServiceName}, Status: {svc.ServiceStatus}");
}
```

---

The **ServiceStatus** field returns one of the following values:

| Status | Description |
|---|---|
| Active | The service is running normally. |
| Disabling | Replicas are being removed. The service transitions to Disabled when removal finishes. |
| Disabled | The service has no replicas or instances. |

## Delete a disabled service

You can delete a service while it's in the Disabled state. No extra steps are needed — the standard deletion flow applies.

```powershell
Remove-ServiceFabricService -ServiceName "fabric:/MyApp/MyService" -Force
```

## Understand upgrade behavior

### Application upgrades

Application upgrades proceed regardless of whether services in the application are disabled:

- Disabled services stay disabled after the upgrade completes.
- You can disable or enable services at any point during an upgrade.
- Service descriptions and configurations are preserved. Only the replica count is zero while the service is disabled.

### Fabric upgrades

Fabric (runtime) upgrades aren't blocked by disabled services. However, rolling back a fabric upgrade to a version that predates the disable/enable feature is rejected when disabled services exist in the cluster, because the rollback would cause inconsistent service states.

## Error codes

The following table lists error codes specific to the disable/enable feature:

| Error code | Description |
|---|---|
| `ServiceAlreadyInRequestedState` | The service is already in the requested state (already disabled or already enabled). |
| `ServiceDisableInProgress` | A disable operation is already in progress for this service. |
| `DisableEnableServiceFeatureDisabled` | The feature is turned off. Set `AllowDisableEnableService` to `true` in the [FailoverManager][fabric-settings-link] configuration. |
| `MaxAllowedDisabledServicesReached` | The cluster limit for disabled services is reached. |
| `OperationFailedServicePurged` | The service was purged after an FM data loss event. Re-create the service to restore it. |

## Configuration reference

Set the following values in the [**FailoverManager**][fabric-settings-link] section of the cluster configuration:

| Setting | Type | Default | Description |
|---|---|---|---|
| `AllowDisableEnableService` | bool | `false` | Turn the Scale-to-Zero feature on or off. This setting is dynamic and cluster-wide. |

## Known limitations

- System services can't be disabled.
- In an unlikely event where FM experiences data loss, disabled services will be permanently lost and require re-creation. After recovery, these services may still appear in Service Fabric Explorer (SFX) but in an **Unknown** state. They are cleaned up on the next `DeleteService` or `DisableService`/`EnableService` API call, which returns `OperationFailedServicePurged` meaning that the service was purged and must be re-created.

> [!IMPORTANT]
> The maximum number of services that can be in the disabled state simultaneously is 100,000.

> [!CAUTION]
> Disabled services are automatically deleted after a retention period of **90 days**. Once the timeout expires, the Service Fabric permanently removes the service. To keep a disabled service beyond this period, enable it before the timeout expires.

## Related content

- [Service Fabric application upgrade](application-upgrade-link)
- [Customize Service Fabric cluster settings](fabric-settings-link)

<!-- Links -->
[application-upgrade-link]: service-fabric-application-upgrade.md
[fabric-settings-link]: service-fabric-cluster-fabric-settings.md#failovermanager
