---
title: Back up and restore Azure Service Fabric actors 
description: Learn how to implement back up and restore in your Azure Service Fabric actors.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/11/2022
# Customer intent: As a developer implementing Azure Service Fabric actors, I want to configure backup and restore methods for my actor services, so that I can ensure data integrity and quick recovery of stateful applications.
---


# Implement Reliable Actors back up and restore

> [!NOTE]
> Microsoft recommends to use [Periodic back up and restore](service-fabric-backuprestoreservice-quickstart-azurecluster.md) for configuring data backup of Reliable Stateful services and Reliable Actors. 
> 

In the following example, a custom actor service exposes a method to back up actor data by taking advantage of the remoting listener already present in `ActorService`:

```csharp
public interface IMyActorService : IService
{
    Task BackupActorsAsync();
}

class MyActorService : ActorService, IMyActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }

    public Task BackupActorsAsync()
    {
        return this.BackupAsync(new BackupDescription(PerformBackupAsync));
    }

    private async Task<bool> PerformBackupAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
    {
        try
        {
           // store the contents of backupInfo.Directory
           return true;
        }
        finally
        {
           Directory.Delete(backupInfo.Directory, recursive: true);
        }
    }
}
```
```Java
public interface MyActorService extends Service
{
    CompletableFuture<?> backupActorsAsync();
}

class MyActorServiceImpl extends ActorService implements MyActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<FabricActorService, ActorId, ActorBase> newActor)
    {
       super(context, typeInfo, newActor);
    }

    public CompletableFuture backupActorsAsync()
    {
        return this.backupAsync(new BackupDescription((backupInfo, cancellationToken) -> performBackupAsync(backupInfo, cancellationToken)));
    }

    private CompletableFuture<Boolean> performBackupAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
    {
        try
        {
           // store the contents of backupInfo.Directory
           return true;
        }
        finally
        {
           deleteDirectory(backupInfo.Directory)
        }
    }

    void deleteDirectory(File file) {
        File[] contents = file.listFiles();
        if (contents != null) {
            for (File f : contents) {
               deleteDirectory(f);
             }
        }
        file.delete();
    }
}
```

In this example, `IMyActorService` is a remoting contract that implements `IService` (C#) and `Service` (Java), and is then implemented by `MyActorService`. By adding this remoting contract, methods on `IMyActorService` are now also available to a client by creating a remoting proxy via `ActorServiceProxy`:

```csharp
IMyActorService myActorServiceProxy = ActorServiceProxy.Create<IMyActorService>(
    new Uri("fabric:/MyApp/MyService"), ActorId.CreateRandom());

await myActorServiceProxy.BackupActorsAsync();
```
```Java
MyActorService myActorServiceProxy = ActorServiceProxy.create(MyActorService.class,
    new URI("fabric:/MyApp/MyService"), actorId);

myActorServiceProxy.backupActorsAsync();
```

For more information on Reliable Actors, read the following articles:
* [Actor state management](service-fabric-reliable-actors-state-management.md)
* [Actor lifecycle and garbage collection](service-fabric-reliable-actors-lifecycle.md)
* [Actors API reference documentation](/previous-versions/azure/dn971626(v=azure.100))
* [.NET sample code](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Java sample code](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
