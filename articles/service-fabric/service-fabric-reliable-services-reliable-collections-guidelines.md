---
title: Guidelines for Reliable Collections
description: Guidelines and recommendations for using Service Fabric Reliable Collections in an Azure Service Fabric application.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 05/21/2026
ms.update-cycle: 1095-days
# Customer intent: "As a developer using Azure Service Fabric, I want to implement Reliable Collections best practices, so that I can ensure data consistency and avoid common pitfalls in my application."
---

# Guidelines and recommendations for Reliable Collections in Azure Service Fabric
This section provides guidelines for using Reliable State Manager and Reliable Collections. The goal is to help users avoid common pitfalls.

## Reliable Collection guidelines

The guidelines are organized as simple recommendations prefixed with the terms *Do*, *Consider*, *Avoid*, and *Do not*. Each recommendation that has an accompanying code sample links to the [Examples](#examples) section.

### Do Not

* **Don't** dispose or cancel a committing transaction. This isn't supported and could crash the host process.
* **Don't** use a transaction after it has been committed, aborted, or disposed.
* **Don't** modify an object of custom type returned by read operations (for example, `TryPeekAsync` or `TryGetValueAsync`). Reliable Collections, just like Concurrent Collections, return a reference to the objects and not a copy. Instead deep copy the returned object of a custom type before modifying it. Since structs and built-in types are pass-by-value, you don't need to do a deep copy on them unless they contain reference-typed fields or properties that you intend to modify. See [Example: Don't modify objects returned by read operations](#example-dont-modify-objects-returned-by-read-operations).
* **Don't** create reliable state with `IReliableStateManager.GetOrAddAsync` and use the reliable state in the same transaction. This results in an InvalidOperationException. See [Example: Create state and use it in separate transactions](#example-create-state-and-use-it-in-separate-transactions).
* **Don't** apply [parallel](/dotnet/standard/parallel-programming/how-to-use-parallel-invoke-to-execute-parallel-operations) 
  or [concurrent](/dotnet/api/system.threading.tasks.task.whenall) operations within a transaction. 
  Only one user thread operation is supported within a transaction. Otherwise, it will cause memory leaks and lock issues. See [Example: Avoid parallel or concurrent operations in a transaction](#example-avoid-parallel-or-concurrent-operations-in-a-transaction).
* **Don't** create a transaction within another transaction's `using` statement because it can cause deadlocks. See [Example: Avoid nested transactions](#example-avoid-nested-transactions).
* **Don't** use TimeSpan.MaxValue for timeouts. Timeouts are used to detect stalled replication and deadlocks. Excessively large values can hide system issues and delay failure detection. The default timeout is 4 seconds for all the Reliable Collection APIs. Most users should use the default timeout.
* **Don't** use an enumeration outside of the transaction scope it was created in. See [Example: Enumerate within the transaction scope](#example-enumerate-within-the-transaction-scope).

### Do

* **Do** ensure all transactions are disposed after their use. Enumerable transactions which are not disposed can prevent internal checkpoint cleanup logic and bloat the Reliable Collections size due to inability to clean up old data. See [Example: Dispose enumerable transactions](#example-dispose-enumerable-transactions).
* **Do** perform a multi-phase upgrade to your code when adding or changing state providers or associated serializers, due to the nature of rolling upgrades in Service Fabric. Please refer to [Reliable Collection object serialization in Azure Service Fabric - Upgradability](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-serialization#upgradability) for more details.
* **Do** ensure that your `IComparable<TKey>` implementation is correct. The system takes dependency on `IComparable<TKey>` for merging checkpoints and rows.
* **Do** ensure the operation order of different dictionaries stays the same for all concurrent transactions when reading or writing multiple dictionaries to avoid deadlock. See [Example: Consistent operation order across dictionaries](#example-consistent-operation-order-across-dictionaries).
* **Do** use an Update lock when reading an item that you intend to modify within the same transaction. This prevents deadlocks that occur when multiple transactions read the same key with a shared lock and later attempt to upgrade to an exclusive lock. See [Example: Use an Update lock for read-then-write](#example-use-an-update-lock-for-read-then-write).
* **Do** handle InvalidOperationException. User transactions can be aborted by the system for a variety of reasons. For example, when the Reliable State Manager is changing its role out of Primary or when a long-running transaction is blocking truncation of the transactional log. In such cases, the user may receive an InvalidOperationException indicating that their transaction has already been terminated. Assuming the termination of the transaction wasn't requested by the user, the best way to handle this exception is to dispose the transaction, check if the cancellation token has been signaled (or the role of the replica has been changed), and if not create a new transaction and retry.
* **Do** implement bounded retries with exponential backoff.
* **Do** measure and validate serialization cost before production use. This cost should be minimized as much as possible as all state must be serialized for replication and persistence. Inefficient serializers can significantly increase CPU usage and latency.
* **Do** adjust the CheckpointThresholdInMB setting in relation to the write rate and typical transaction size of your Reliable Collection. This value should be at least double your largest transaction size and such that this setting is exceeded every 30-120 seconds under steady state write load.

### Avoid

* **Avoid** hot keys. Distribute writes evenly across keys to prevent creation of long queues all waiting for a single key to be locked.
* **Avoid** long-running transactions with blocking code. Transactions should be short-lived and blocking code within a transaction can result in blocking log truncation, lead to lock contention and subsequent timeouts. See [Example: Avoid long-running work inside a transaction](#example-avoid-long-running-work-inside-a-transaction).
* **Avoid** mixing single entity operations and multi-entity operations (e.g. `GetCountAsync`, `CreateEnumerableAsync`) in the same transaction due to the different isolation levels. For more information, you can refer to [Transactions And Lock Modes in Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-transactions-locks).
* **Avoid** creating multi-operation transactions with the intention of not committing the transaction. These transactions will still replicate the earlier operations to secondaries and on disposal without commit, will require reverting the progress on each secondary in addition to Aborting the transaction. 

### Consider

* **Consider** disposing transactions as soon as possible after commit completes (especially if using ConcurrentQueue).
* **Consider** keeping your items (for example, TKey + TValue for Reliable Dictionary) below 80 KBytes: the smaller the better. This reduces the amount of Large Object Heap usage as well as disk and network IO requirements. Often, it reduces replicating duplicate data when only one small part of the value is being updated. A common way to achieve this in a Reliable Dictionary is to break your rows into multiple rows.
* **Consider** keeping the number of Reliable Collections per partition to be less than 1000. Prefer Reliable Collections with more items over more Reliable Collections with fewer items.
* **Consider** using backup and restore functionality to have disaster recovery.

Some things to keep in mind:
* When [string](/dotnet/api/system.string) is used as the key for a reliable dictionary, the sorting order uses [default string comparer CurrentCulture](/dotnet/api/system.string.compare#system-string-compare(system-string-system-string)). Note that the CurrentCulture sorting order is different from [Ordinal string comparer](/dotnet/api/system.stringcomparer.ordinal).
* The default cancellation token is `CancellationToken.None` in all Reliable Collections APIs.
* The key type parameter (*TKey*) for a Reliable Dictionary must correctly implement `GetHashCode()` and `Equals()`. Keys must be immutable.
* To achieve high availability for the Reliable Collections, each service should have at least a target and minimum replica set size of 3.
* Read operations on the secondary may read versions that aren't quorum committed.
  This means that a version of data that is read from a single secondary might be false progressed.
  Reads from Primary are always stable: they can never be false progressed.
* Security/Privacy of the data persisted by your application in a reliable collection is your decision and subject to the protections provided by your storage management; i.e., Operating System disk encryption could be used to protect your data at rest.
* `ReliableDictionary` enumeration uses a sorted data structure ordered by key. To make enumeration efficient, commits are added to a temporary hashtable and later moved into the main sorted data structure post checkpoint. Adds/Updates/Deletes have best case runtime of O(1) and worst case runtime of O(log n), in the case of validation checks on the presence of the key. Gets might be O(1) or O(log n) depending on whether you're reading from a recent commit or from an older commit.

## Additional guidelines for volatile Reliable Collections

When deciding to use volatile reliable collections, consider the following:

* ```ReliableDictionary``` does have volatile support
* ```ReliableQueue``` does have volatile support
* ```ReliableConcurrentQueue``` does NOT have volatile support
* Persisted services CANNOT be made volatile. Changing the ```HasPersistedState``` flag to ```false``` requires recreating the entire service from scratch
* Volatile services CANNOT be made persisted. Changing the ```HasPersistedState``` flag to ```true``` requires recreating the entire service from scratch
* ```HasPersistedState``` is a service level config. This means that **ALL** collections will either be persisted or volatile. You can't mix volatile and persisted collections
* Quorum loss of a volatile partition results in complete data loss
* Backup and restore is NOT available for volatile services

## Examples

The examples below illustrate the recommendations from the [Reliable Collection guidelines](#reliable-collection-guidelines) section.

### Example: Don't modify objects returned by read operations

Applies to: **Don't** modify an object of custom type returned by read operations.

```csharp
// Suppose Order is a reference type stored in the dictionary:
//     class Order { public int Quantity { get; set; } public List<string> Tags { get; set; } }

// INCORRECT: mutating the reference returned by the dictionary directly.
// The in-memory instance is shared with the Reliable Collection, so this
// change is visible without going through a transaction and is never
// replicated or persisted. Concurrent readers may also observe partial
// mutations.
using (var tx = this.StateManager.CreateTransaction())
{
    var result = await orders.TryGetValueAsync(tx, "order-1");
    if (result.HasValue)
    {
        result.Value.Quantity += 1;          // mutating the shared reference
        result.Value.Tags.Add("expedited");  // mutating a reference-typed field
    }
    await tx.CommitAsync();
}

// CORRECT: deep copy the value, modify the copy, then write it back via
// the dictionary so the update is transacted, replicated, and persisted.
using (var tx = this.StateManager.CreateTransaction())
{
    var result = await orders.TryGetValueAsync(tx, "order-1");
    if (result.HasValue)
    {
        var updated = new Order
        {
            Quantity = result.Value.Quantity + 1,
            Tags = new List<string>(result.Value.Tags) { "expedited" },
        };

        await orders.SetAsync(tx, "order-1", updated);
    }
    await tx.CommitAsync();
}
```

### Example: Create state and use it in separate transactions

Applies to: **Don't** create reliable state with `IReliableStateManager.GetOrAddAsync` and use the reliable state in the same transaction.

```csharp
// INCORRECT: creating reliable state and using it within the same transaction.
using (var tx = this.StateManager.CreateTransaction())
{
    var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, int>>(
        tx, "myDictionary");

    // Throws InvalidOperationException when myDictionary was newly created above,
    // because the new state isn't registered until the creating transaction commits.
    await myDictionary.AddAsync(tx, "key1", 42);

    await tx.CommitAsync();
}

// CORRECT: create (or get) the reliable state in one transaction,
// then use it in a separate transaction.
IReliableDictionary<string, int> myDictionary;
using (var tx = this.StateManager.CreateTransaction())
{
    myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, int>>(
        tx, "myDictionary");
    await tx.CommitAsync();
}

using (var tx = this.StateManager.CreateTransaction())
{
    await myDictionary.AddAsync(tx, "key1", 42);
    await tx.CommitAsync();
}
```

### Example: Avoid parallel or concurrent operations in a transaction

Applies to: **Don't** apply parallel or concurrent operations within a transaction.

```csharp
// INCORRECT: fanning out concurrent operations on the same transaction.
using (var tx = this.StateManager.CreateTransaction())
{
    var writes = keys.Select(k => myDictionary.AddAsync(tx, k, 0));

    // Multiple operations on `tx` run concurrently — causes lock/memory issues.
    await Task.WhenAll(writes);

    await tx.CommitAsync();
}

// ALSO INCORRECT: Parallel.ForEachAsync sharing a single transaction across threads.
using (var tx = this.StateManager.CreateTransaction())
{
    await Parallel.ForEachAsync(keys, async (k, _) =>
    {
        await myDictionary.AddAsync(tx, k, 0);
    });

    await tx.CommitAsync();
}

// CORRECT: issue operations on the transaction one at a time (sequentially awaited).
using (var tx = this.StateManager.CreateTransaction())
{
    foreach (var k in keys)
    {
        await myDictionary.AddAsync(tx, k, 0);
    }

    await tx.CommitAsync();
}
```

### Example: Avoid nested transactions

Applies to: **Don't** create a transaction within another transaction's `using` statement.

```csharp
// INCORRECT: an inner transaction is created while the outer transaction
// is still open. The inner tx tries to acquire locks on keys the outer tx
// already holds, but the outer can't release them until it commits — and
// it can't commit until the inner finishes. The two transactions deadlock.
using (var outerTx = this.StateManager.CreateTransaction())
{
    await myDictionary.SetAsync(outerTx, "key1", 1);

    using (var innerTx = this.StateManager.CreateTransaction())
    {
        // Blocks waiting for the write lock on "key1" held by outerTx.
        await myDictionary.SetAsync(innerTx, "key1", 2);
        await innerTx.CommitAsync();
    }

    await outerTx.CommitAsync();
}

// CORRECT: commit and dispose each transaction before starting the next one.
using (var tx = this.StateManager.CreateTransaction())
{
    await myDictionary.SetAsync(tx, "key1", 1);
    await tx.CommitAsync();
}

using (var tx = this.StateManager.CreateTransaction())
{
    await myDictionary.SetAsync(tx, "key1", 2);
    await tx.CommitAsync();
}
```

### Example: Enumerate within the transaction scope

Applies to: **Don't** use an enumeration outside of the transaction scope it was created in.

```csharp
// INCORRECT: the enumerable is captured but enumerated after its
// transaction is disposed. The enumeration is tied to the tx's snapshot
// and becomes invalid as soon as the tx is committed and disposed.
IAsyncEnumerable<KeyValuePair<string, int>> enumerable;
using (var tx = this.StateManager.CreateTransaction())
{
    enumerable = await myDictionary.CreateEnumerableAsync(tx);
    await tx.CommitAsync();
} // tx is disposed here — the enumeration is no longer valid.

var enumerator = enumerable.GetAsyncEnumerator();
while (await enumerator.MoveNextAsync(CancellationToken.None))
{
    var current = enumerator.Current;
}

// CORRECT: enumerate within the same transaction scope that created the enumerable.
using (var tx = this.StateManager.CreateTransaction())
{
    var enumerable = await myDictionary.CreateEnumerableAsync(tx);
    var enumerator = enumerable.GetAsyncEnumerator();
    while (await enumerator.MoveNextAsync(CancellationToken.None))
    {
        var current = enumerator.Current;
        // process current
    }
    await tx.CommitAsync();
}
```

### Example: Dispose enumerable transactions

Applies to: **Do** ensure all transactions are disposed after their use.

```csharp
// INCORRECT: an enumerable is created on a transaction that lives outside
// a `using` block. If iteration throws (or a later code path returns
// early), the transaction is never disposed. Leaked enumerable
// transactions hold their snapshot and prevent internal checkpoint cleanup logic,
// causing the Reliable Collection to grow over time.
var tx = this.StateManager.CreateTransaction();
var enumerable = await myDictionary.CreateEnumerableAsync(tx);
var enumerator = enumerable.GetAsyncEnumerator();
while (await enumerator.MoveNextAsync(CancellationToken.None))
{
    var current = enumerator.Current;
    // process current
}
await tx.CommitAsync();
// tx.Dispose() is never called.
```

### Example: Consistent operation order across dictionaries

Applies to: **Do** ensure the operation order of different dictionaries stays the same for all concurrent transactions.

```csharp
// INCORRECT: TransferAtoB locks dictA before dictB, while TransferBtoA locks
// dictB before dictA. If they run concurrently, each transaction can hold
// one dictionary's lock and block waiting for the other — they deadlock.
async Task TransferAtoB(long amount)
{
    using (var tx = this.StateManager.CreateTransaction())
    {
        await dictA.AddOrUpdateAsync(tx, "balance", k => -amount, (k, v) => v - amount);
        await dictB.AddOrUpdateAsync(tx, "balance", k => amount,  (k, v) => v + amount);
        await tx.CommitAsync();
    }
}

async Task TransferBtoA(long amount)
{
    using (var tx = this.StateManager.CreateTransaction())
    {
        await dictB.AddOrUpdateAsync(tx, "balance", k => -amount, (k, v) => v - amount);
        await dictA.AddOrUpdateAsync(tx, "balance", k => amount,  (k, v) => v + amount);
        await tx.CommitAsync();
    }
}

// CORRECT: both methods touch dictA before dictB. Any number of concurrent
// transfers acquire the locks in the same global order and cannot deadlock.
async Task TransferAtoB(long amount)
{
    using (var tx = this.StateManager.CreateTransaction())
    {
        await dictA.AddOrUpdateAsync(tx, "balance", k => -amount, (k, v) => v - amount);
        await dictB.AddOrUpdateAsync(tx, "balance", k => amount,  (k, v) => v + amount);
        await tx.CommitAsync();
    }
}

async Task TransferBtoA(long amount)
{
    using (var tx = this.StateManager.CreateTransaction())
    {
        await dictA.AddOrUpdateAsync(tx, "balance", k => amount,  (k, v) => v + amount);
        await dictB.AddOrUpdateAsync(tx, "balance", k => -amount, (k, v) => v - amount);
        await tx.CommitAsync();
    }
}
```

### Example: Use an Update lock for read-then-write

Applies to: **Do** use an Update lock when reading an item that you intend to modify within the same transaction.

```csharp
// INCORRECT: read uses the default (shared) lock, then the same transaction
// tries to upgrade to an exclusive lock to write. Two concurrent transactions
// running this pattern both hold a shared read lock on "key1" and each waits
// for the other to release before they can upgrade — they deadlock.
using (var tx = this.StateManager.CreateTransaction())
{
    var current = await myDictionary.TryGetValueAsync(tx, "key1");

    if (current.HasValue)
    {
        await myDictionary.SetAsync(tx, "key1", current.Value + 1);
    }

    await tx.CommitAsync();
}

// CORRECT: read with LockMode.Update. Only one transaction at a time can
// hold the Update lock on "key1", so the later upgrade to an exclusive
// write lock cannot deadlock with another transaction following the same
// read-then-write pattern.
using (var tx = this.StateManager.CreateTransaction())
{
    var current = await myDictionary.TryGetValueAsync(tx, "key1", LockMode.Update);

    if (current.HasValue)
    {
        await myDictionary.SetAsync(tx, "key1", current.Value + 1);
    }

    await tx.CommitAsync();
}
```

### Example: Avoid long-running work inside a transaction

Applies to: **Avoid** long-running transactions with blocking code.

```csharp
// INCORRECT: a long-running call inside the transaction keeps the write
// lock on "key1" held for the duration of the call.
using (var tx = this.StateManager.CreateTransaction())
{
    // AddOrUpdateAsync acquires a write lock on "key1".
    await myDictionary.AddOrUpdateAsync(tx, "key1", k => 0, (k, existing) => existing + 1);

    // This HTTP call may run for a long time. The write lock on "key1"
    // is held for the entire duration, blocking other writers to the
    // same key.
    var payload = await httpClient.GetStringAsync(uri);

    await tx.CommitAsync();
}
```

## Next steps
* [Working with Reliable Collections](service-fabric-work-with-reliable-collections.md)
* [Transactions and Locks](service-fabric-reliable-services-reliable-collections-transactions-locks.md)
* Managing Data
  * [Backup and Restore](service-fabric-reliable-services-backup-restore.md)
  * [Notifications](service-fabric-reliable-services-notifications.md)
  * [Serialization and Upgrade](service-fabric-application-upgrade-data-serialization.md)
  * [Reliable State Manager configuration](service-fabric-reliable-services-configuration.md)
* Others
  * [Reliable Services quickstart](service-fabric-reliable-services-quick-start.md)
  * [Developer reference for Reliable Collections](/dotnet/api/microsoft.servicefabric.data.collections#microsoft_servicefabric_data_collections)
