---
title: Throttling Guidelines for Reliable Collections
description: Learn how throttling manifests when you use Azure Service Fabric Reliable Collections and how to prevent scenarios that cause throttling in a Service Fabric replication.
ms.topic: concept-article
ms.author: justincurley
author: curleyfries
ms.service: azure-service-fabric
services: service-fabric
ms.date: 06/25/2026
ms.update-cycle: 1095-days
ai-usage: ai-assisted
# Customer intent: "As a developer who uses Azure Service Fabric, I want to understand how throttling works in Reliable Collections and implement best practices, so that I can reduce the possibility of throttling as a result of my application code."
---

# Throttling in Reliable Collections

This article provides a practical guide to the types of throttling that you might encounter when you write to Reliable Collections in Azure Service Fabric. It also offers best practices for avoiding and handling throttling.

## Background

When you commit a change to a Reliable Collection, the write operation is appended to a transactional log. The process persists the log to disk and replicates it to a quorum of secondary replicas before the commit finishes. This commit pipeline drains at a finite *rate*, determined by how fast it can flush to disk and how quickly secondaries acknowledge.

The pipeline absorbs short bursts in bounded in-memory buffers and queues, but those buffers are finite. When the sustained rate of incoming work exceeds the rate at which the pipeline drains, the buffers fill. To protect durability and consistency, the system slows or pauses commits. This slowdown or pause is *throttling*.

Throttling is expected behavior, not a defect. The guidelines in this article describe the forms that it takes and how to stay in a healthy operating range.

### Key terms

If you're new to Reliable Collections, the following terms appear throughout this article:

- **Transaction**: The unit of work you create with `CreateTransaction` to read from or write to Reliable Collections. Changes become durable only when you commit.
- **Commit**: The operation that finalizes a transaction, performed by calling the `CommitAsync` API. A commit isn't complete until the change is persisted to the log and replicated to a quorum of secondary replicas.
- **Replica**: A copy of a partition's state. The **primary** serves reads and writes; **secondary** replicas receive replicated operations and acknowledge them.
- **Quorum**: The majority of replicas that must acknowledge a write before the commit finishes.
- **Transactional log**: The on-disk log that every write appends to before a commit finishes. It provides durability and ordering.
- **Replication queue**: A bounded queue on the primary that holds each replicated operation until every secondary acknowledges it.
- **Checkpoint**: A periodic operation that persists in-memory state so that older log records can be discarded.
- **Truncation**: Discarding log records that are no longer needed, which advances the log head and frees log space.
- **Log reader**: A reader held at a position in the log (for example, by a backup or a replica build) that prevents truncation past that position until it's released.

## Types of throttling

Throttling takes two forms. *Capacity throttling* is a slope that you can ease back from by lowering concurrency. *Stall throttling* is a cliff that you want to stay well clear of.

### Capacity (partial) throttling

Capacity throttling is really a *write stall* caused by resource and capacity constraints rather than throttling in the strict sense. Nothing rejects new writes or blocks transactions in a controlled manner. The commit pipeline drains at a fixed maximum rate set by disk and replication speed. As you push more concurrent transactions than the pipeline can drain, each transaction waits longer in the queue. Commits queue and latency climbs in proportion, with no errors.

When the pipeline is saturated, individual commits can take a long time to finish (tens of seconds or more), so it can *feel* like the system is throttling you even though every write is still accepted and eventually completes.

| Aspect | Behavior |
| ------ | -------- |
| **Trigger** | More in-flight transactions occur than the pipeline can drain. |
| **Symptom** | Commit latency rises roughly linearly with concurrency. Throughput stays flat. Writes are still accepted, but slow commits can create the impression of throttling. |
| **Throughput** | Unchanged. Adding concurrency doesn't increase it. |
| **Recovery** | Recovery is immediate after concurrency drops back below the drain rate. |

Adding concurrency above the drain rate buys latency, not throughput. To increase throughput, reduce per-commit cost. Use smaller values, fewer collections, faster serialization, and faster storage.

### Stall (full) throttling

Stall throttling is the abrupt, full-stop case. It happens when the replicator stops accepting new writes entirely due to one of these factors:

- The log can't be truncated fast enough.
- The disk can't keep up.
- The primary's replication queue fills because secondaries aren't acknowledging fast enough.

While the condition persists, commits stop finishing and callers receive transient errors. Throughput resumes in a burst after the condition clears.

The triggers fall into two families:

- **Local to the primary**. The log head can't advance, so log usage crosses the throttle limit. Most things people think of as separate causes (replica build, backup, a stuck checkpoint) are really subcases of this one.

- **Replication problem rather than a logging one**. The primary holds each operation in a bounded queue until every secondary acknowledges it. A single slow, unhealthy, or still-building secondary fills the queue and stalls commits with a transient queue-full error. This situation can happen even when the log and disk are perfectly healthy.

  Stall throttling isn't purely a local I/O condition. It can be driven entirely by replication falling behind.

| Trigger | Subtrigger | Why it blocks commits | Relevant configurations | Automatic mitigations |
| ------- | ----------- | --------------------- | ---------------------- | ------------------------- |
| **Disk can't keep up** (write backpressure) | Not applicable | Buffered records are waiting to flush exceed the write-cache limit. The log writer can't persist to disk as fast as commits arrive, so new writes are rejected until the flush drains. | [`MaxWriteQueueDepthInKB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.maxwritequeuedepthinkb) sets the pending-flush ceiling. Writes throttle after unflushed bytes exceed the ceiling. Raising it adds headroom but increases data buffered in memory ahead of a flush. | Not applicable. |
| **Log usage over the throttle limit** (blocked truncation) | Pending checkpoint | A checkpoint already in progress is still holding log space. The log head can't advance until it finishes. | [`CheckpointThresholdInMB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.checkpointthresholdinmb) controls how much log accumulates before a checkpoint fires.  [`ThrottlingThresholdFactor`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.throttlingthresholdfactor) sets the throttle ceiling as `max(CheckpointThresholdInMB, MinLogSizeInMB) × ThrottlingThresholdFactor`. A larger factor adds headroom between checkpoint start and the throttle point. | The replicator automatically initiates checkpoints as the log grows, so truncation can proceed after the checkpoint finishes. No manual action is needed. |
| | Replica build/copy (full or partial) | The build holds a log reader at an early position while streaming state to the new replica, which prevents truncation. Log usage climbs past the limit. Build is never a throttle trigger on its own; it only stalls commits by blocking truncation. | Governed by the same [`ThrottlingThresholdFactor`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.throttlingthresholdfactor)/`CheckpointThresholdInMB` headroom. No setting disables the reader that a build must hold. | The log reader is released automatically when the build finishes or is canceled, which unblocks truncation. |
| | Backup | A backup holds a log reader while reading state and logs, which blocks truncation until it finishes. This is the same path as a replica build. | [`MaxAccumulatedBackupLogSizeInMB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.maxaccumulatedbackuplogsizeinmb) bounds how much log a backup chain can accumulate before it's forced to fail. | A backup that would exceed `MaxAccumulatedBackupLogSizeInMB` is failed, which releases the reader and relieves log pressure rather than letting it stall commits indefinitely. |
| **Primary replication queue full** | Not applicable | The primary holds each replicated operation in a bounded queue and can't free it until *every* secondary acknowledges it. Operations are retained for lagging replicas to catch up, even after a write quorum already acknowledged and the commit already finished. A single slow, unhealthy, or still-building secondary is enough to fill the queue. At that point, new commits are rejected as a transient queue-full error. The underlying replication engine enforces this condition, separately from the log throttle. | [`MaxPrimaryReplicationQueueSize`](/dotnet/api/system.fabric.replicatorsettings.maxprimaryreplicationqueuesize) and [`MaxReplicationQueueMemorySize`](/dotnet/api/system.fabric.replicatorsettings.maxreplicationqueuememorysize) set the bounded-queue limits. Raising them gives slower secondaries more time to catch up before commits are rejected, at the cost of higher memory usage on the primary. | The replicator restarts a secondary that it detects is too slow, so the queue can drain. This action is gated on queue usage crossing a configured threshold (or the oldest-operation age). The replicator restarts the secondary only while a write quorum is still maintained, so it never sacrifices availability to relieve the throttle. |

| Aspect | Behavior |
| ------ | -------- |
| **During the stall** | Commits are *rejected* with a transient error, not served slowly. Throughput and commit-completion count collapse, the error rate spikes, and CPU drops. Latency of *successful* commits doesn't climb here, because there are few or no completions to measure. |
| **After the stall** | A recovery burst drains the backlog. Queued in-flight transactions and retried operations all finish in a tight window. Each carries the time that it spent blocked or retrying as end-to-end latency. |
| **Tail impact** | The P95/P99 spike comes from the recovery burst and from end-to-end (attempt-to-success) latency that includes retries. It doesn't come from the stall window producing slow completions. If you measure only the single-call latency of successful commits, the stall shows up as a throughput hole plus error-rate spike, with the tail spiking when the backlog drains. P50 can still look normal if only a small fraction of operations hit the stall window. |

> [!NOTE]
> The more transactions are in flight when a stall hits, the more freeze at once. Lower concurrency shrinks the number.

Watch for operation timeouts as a stall symptom. When commits can't finish because of lock contention, log unavailability, or a full replication queue, in-flight operations block until they exceed their timeout and surface a `TimeoutException` (often wrapped in a transient error). A rising rate of operation timeouts, especially alongside a throughput drop and error-rate spike, is a strong signal that you're hitting a stall rather than gradual capacity throttling. Keep timeouts near the default so these stalls surface quickly instead of being masked by excessively long waits.

## Implement your own throttling

The platform throttles to protect durability, but it doesn't cap how much work your application sends into the pipeline. That responsibility is yours. Follow the best practices in the following sections to implement your own client-side throttling, bounding how many transactions you have in flight and how aggressively you retry. Without it, your application can push the pipeline past its drain rate, drive log and queue usage to their limits, and trigger capacity-related write stalls that you could have avoided.

The following do's, don'ts, and code patterns show how to build that throttling and stay in a healthy operating range.

## Do

- **Do bound the number of in-flight transactions with an explicit gate**. For example, use `SemaphoreSlim`, a bounded channel, or a Task Parallel Library (TPL) dataflow block, rather than fanning out unbounded `CommitAsync` calls. A rough cap of about 10,000 concurrent transactions is a reasonable starting limit. Adjust up or down based on observed performance. See [Bounding in-flight transactions](#bounding-in-flight-transactions) later in this article.
- **Do keep transactions short**. Perform any external or long-running work *before* you open the transaction so that it never holds locks or pins the log.
- **Do tune the checkpoint threshold to your write rate**. Tune so that checkpoints happen regularly but not constantly, to avoid large disruptive bursts of checkpoint and truncation work. If you don't want to instrument this capability yourself, Reliable Collections already expose a [`Log Flush Bytes/sec`](/azure/service-fabric/service-fabric-reliable-services-diagnostics#performance-counters) performance counter (under the **Service Fabric Transactional Replicator** category) that shows your sustained write rate to the log. Adjust `CheckpointThresholdInMB` so
that your typical write rate fills the log to it roughly every 30 to 120 seconds.
- **Do monitor tail latency (P95/P99), not just P50**. Rising tail latency is usually the earliest warning of an impending stall. Alert on it.
- **Do keep the default API timeout (or close to it)**. It helps ensure that genuine stalls and deadlocks surface quickly.
- **Do handle `InvalidOperationException` on transactions**. The system might abort a transaction (for example, on a replica role change). Dispose of it, recheck state, and retry with a new transaction.
- **Do use bounded retries with exponential backoff**. This action prevents retries from adding to the queue depth that's causing the latency. See [Bounded retries with exponential backoff](#bounded-retries-with-exponential-backoff) later in this article.
- **Do measure and minimize serialization cost before production**. Inefficient serializers inflate CPU and per-commit latency.
- **Do dispose of transactions promptly after commit**. Doing so releases resources quickly.

## Don't

- **Don't add concurrency and expect unbounded throughput gains**. Some concurrency does help. Batching and pipelining let the replicator amortize per-commit costs, so throughput climbs with in-flight transactions up to the point where the pipeline saturates. Beyond that point, the rate is fixed. Additional in-flight transactions only raise latency and widen the impact of a stall without improving throughput. Find that saturation point for your workload.
- **Don't run unbounded parallel commits against a single partition**. High in-flight counts raise latency and widen the impact of any stall.
- **Don't apply parallel or concurrent operations within a single transaction**. Only one user-thread operation per transaction is supported. Fan-out (`Task.WhenAll`, `Parallel.ForEachAsync`) causes lock issues and memory leaks. Issue operations sequentially. See [Sequential operations within a transaction](#sequential-operations-within-a-transaction) later in this article.
- **Don't hold a transaction open across long-running or blocking work (HTTP calls, large CPU loops, external I/O)**. It holds locks, blocks log truncation, and prevents memory reclamation.
- **Don't use `TimeSpan.MaxValue` or excessively large timeouts**. They hide stalls and deadlocks. They also delay failure detection.
- **Don't create nested transactions**. And don't use an enumeration or newly created state outside the transaction that produced it. See [Avoiding nested transactions](#avoiding-nested-transactions) later in this article.

## Avoid/Consider

- **Avoid hot keys**. Concentrated writes serialize behind a single lock. Distribute writes across keys.
- **Avoid mixing single-entity and multiple-entity operations in one transaction**. Differing isolation levels increase locking cost.
- **Consider keeping individual items small (well under 80 KB)**. Split large values into multiple rows to reduce flush, replication, checkpoint, and build cost.
- **Consider keeping the number of Reliable Collections per partition modest (well under a thousand)**. Prefer more items over more collections.

## Code patterns

The following examples are illustrative pseudocode. They show the shape of each pattern, not a copy-paste-ready implementation. Adapt the gate sizes, retry counts, and exception handling to your workload.

### Bounding in-flight transactions

Gate concurrency by using `SemaphoreSlim` so that only `maxInFlight` transactions (at most) are ever outstanding. The gate blocks new work after the limit is reached, instead of letting unbounded `CommitAsync` calls pile into the replication queue. See
[Do bound the number of in-flight transactions with an explicit gate](#do) earlier in this article.

```csharp
private readonly SemaphoreSlim gate = new SemaphoreSlim(maxInFlight);

async Task ProcessAsync(IEnumerable<Item> items, CancellationToken ct)
{
    var tasks = new List<Task>();
    foreach (var item in items)
    {
        await gate.WaitAsync(ct);             // blocks once maxInFlight is reached
        tasks.Add(CommitOneAsync(item, ct));
    }
    await Task.WhenAll(tasks);
}

async Task CommitOneAsync(Item item, CancellationToken ct)
{
    try
    {
        using (var tx = this.stateManager.CreateTransaction())
        {
            await dictionary.AddAsync(tx, item.Key, item.Value);
            await tx.CommitAsync();
        }
    }
    finally
    {
        gate.Release();                       // always release, even on failure
    }
}
```

### Bounded retries with exponential backoff

Retry only transient failures, because `FabricTransientException` covers throttling and queue-full situations. Cap the attempt count, and grow the delay exponentially so that retries don't pile back onto a queue that's already under pressure. See
[Do use bounded retries with exponential backoff](#do) earlier in this article.

```csharp
async Task CommitWithRetryAsync(Func<Task> commitAction, CancellationToken ct)
{
    const int maxAttempts = 5;
    var delay = TimeSpan.FromMilliseconds(100);

    for (int attempt = 1; ; attempt++)
    {
        try
        {
            await commitAction();
            return;
        }
        catch (FabricTransientException) when (attempt < maxAttempts)
        {
            // Transient (throttling / queue full): back off, then retry.
            await Task.Delay(delay, ct);
            delay += delay;                   // exponential: 100ms, 200ms, 400ms, ...
        }
        // Non-transient exceptions and the final attempt propagate to the caller.
    }
}
```

### Sequential operations within a transaction

A transaction supports only one user-thread operation at a time. Fanning out concurrent operations on the same transaction causes lock problems and memory leaks. Issue them sequentially instead. See
[Don't apply parallel or concurrent operations within a single transaction](#dont) earlier in this article.

```csharp
// WRONG: concurrent operations on one transaction are not supported.
using (var tx = stateManager.CreateTransaction())
{
    await Task.WhenAll(
        dictionary.AddAsync(tx, "a", 1),
        dictionary.AddAsync(tx, "b", 2));     // fan-out on the same tx
    await tx.CommitAsync();
}

// RIGHT: issue the operations one after another.
using (var tx = stateManager.CreateTransaction())
{
    await dictionary.AddAsync(tx, "a", 1);
    await dictionary.AddAsync(tx, "b", 2);
    await tx.CommitAsync();
}
```

### Avoiding nested transactions

Don't open a second transaction inside the scope of the first. Commit the outer transaction before you start the next, or perform both operations within a single transaction. See
[Don't create nested transactions](#dont) earlier in this article.

```csharp
// WRONG: a transaction created inside the scope of another.
using (var tx1 = stateManager.CreateTransaction())
{
    await dictionary.AddAsync(tx1, "a", 1);

    using (var tx2 = stateManager.CreateTransaction())   // nested — not supported
    {
        await dictionary.AddAsync(tx2, "b", 2);
        await tx2.CommitAsync();
    }
    await tx1.CommitAsync();
}

// RIGHT: one transaction, committed before any further work.
using (var tx = stateManager.CreateTransaction())
{
    await dictionary.AddAsync(tx, "a", 1);
    await dictionary.AddAsync(tx, "b", 2);
    await tx.CommitAsync();
}
```

## Related content

- [Guidelines and recommendations for Reliable Collections in Azure Service Fabric](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines)
- [Transactions and locks in Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-transactions-locks)
- [Reliable State Manager configuration](/azure/service-fabric/service-fabric-reliable-services-configuration)
- [Backup and restore for Reliable Services](/azure/service-fabric/service-fabric-reliable-services-backup-restore)
