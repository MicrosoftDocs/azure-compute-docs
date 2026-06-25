---
title: Throttling Guidelines for Reliable Collections
description: Description of how throttling manifests when using Service Fabric Reliable Collecitons and guidelines to prevent scenarios which result in throttling in an Azure Service Fabric Replication
ms.topic: concept-article
ms.author: curleyfries
author: curleyfries
ms.service: azure-service-fabric
services: service-fabric
ms.date: 06/25/2026
ms.update-cycle: 1095-days
# Customer intent: "As a developer using Azure Service Fabric, I want to understand how throttling works in Reliable Collections and implement best practices, so that I can reduce the possibility of throttling as a result of my application code."
---

# Throttling in Reliable Collections

A practical guide to the types of throttling you may observe when writing to Reliable
Collections in Azure Service Fabric, and the dos and don'ts for avoiding and handling it.

## Background

When you commit a change to a Reliable Collection, the write is appended to a transactional
log, persisted to disk, and replicated to a quorum of secondary replicas before the commit
completes. This commit pipeline drains at a finite *rate*, set by how fast it can flush to disk
and how fast secondaries acknowledge. It absorbs short bursts in bounded in-memory buffers and
queues, but those buffers are themselves finite. When the sustained rate of incoming work
exceeds the rate at which the pipeline can drain, the buffers fill and the system slows or
pauses commits to protect durability and consistency. That slowdown or pause is **throttling**.

Throttling takes two forms. **Capacity throttling** is the gradual, self-correcting case: as
concurrency rises above what the pipeline can drain, commits simply queue and latency climbs in
proportion, with no errors. **Stall throttling** is the abrupt, full-stop case: the replicator
stops accepting new writes entirely — because the log can't be truncated, the disk can't keep
up, or the replication queue is full — and callers receive transient errors until the condition
clears, after which throughput resumes in a burst. Capacity throttling is a slope you can ease
back from by lowering concurrency; stall throttling is a cliff you want to stay well clear of.

Throttling is expected behavior, not a defect. These guidelines describe the forms it takes
and how to stay in a healthy operating range.

## Types of throttling

### 1. Capacity throttling (partial)

The commit pipeline drains at a fixed maximum rate set by disk and replication speed. As you
push more concurrent transactions than the pipeline can drain, each one waits longer in the
queue. The effect is gradual and predictable.

| Aspect | Behavior |
|--------|----------|
| **Trigger** | More in-flight transactions than the pipeline can drain. |
| **Symptom** | Commit latency rises roughly linearly with concurrency; throughput stays flat. |
| **Throughput** | Unchanged -- adding concurrency does not increase it. |
| **Recovery** | Immediate once concurrency drops back below the drain rate. |

> Adding concurrency above the drain rate buys latency, not throughput. To increase
> throughput, reduce per-commit cost (smaller values, fewer collections, faster
> serialization, faster storage).

### 2. Stall throttling (full / 100%)

Full throttling happens when the replicator stops accepting new writes because the log cannot
be truncated fast enough, the disk cannot keep up, or the primary's replication queue fills
because secondaries are not acknowledging fast enough. While the condition persists, commits
stop completing and callers receive transient errors, then throughput resumes in a burst once
it clears. This is a cliff, not a slope.

The triggers fall into two families. The first is local to the primary: the log head can't
advance, so log usage crosses the throttle limit. Most things people think of as separate
causes (replica build, backup, a stuck checkpoint) are really
sub-cases of this one. The second is a replication problem rather than a logging one: the
primary holds each operation in a bounded queue until every secondary acknowledges it, so a
single slow, unhealthy, or still-building secondary can fill the queue and stall commits with
a transient queue-full error — even when the log and disk are perfectly healthy. Stall
throttling is therefore not purely a local I/O condition; it can be driven entirely by
replication falling behind.

| Trigger | Sub-trigger | Why it blocks commits | Relevant configurations | Existing auto-mitigations |
|---------|-------------|-----------------------|-------------------------|---------------------------|
| **Disk can't keep up** (write backpressure) | — | Buffered records waiting to flush exceed the write-cache limit. The log writer can't persist to disk as fast as commits arrive, so new writes are rejected until the flush drains. | [`MaxWriteQueueDepthInKB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.maxwritequeuedepthinkb) sets the pending-flush ceiling: writes throttle once unflushed bytes exceed it. Raising it adds headroom but increases data buffered in memory ahead of a flush. | |
| **Log usage over the throttle limit** (blocked truncation) | Pending checkpoint | A checkpoint already in progress is still holding log space, so the log head can't advance until it finishes. | [`CheckpointThresholdInMB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.checkpointthresholdinmb) controls how much log accumulates before a checkpoint fires; [`ThrottlingThresholdFactor`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.throttlingthresholdfactor) sets the throttle ceiling as `max(CheckpointThresholdInMB, MinLogSizeInMB) × ThrottlingThresholdFactor`. A larger factor adds headroom between checkpoint start and the throttle point. | The replicator auto-initiates checkpoints as the log grows, so truncation can proceed once the checkpoint completes; no manual action needed. |
| | Replica build / copy (full or partial) | The build holds a log reader at an early position while streaming state to the new replica, preventing truncation; log usage climbs past the limit. Build is never a throttle trigger on its own — it only stalls commits by blocking truncation. | Governed by the same [`ThrottlingThresholdFactor`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.throttlingthresholdfactor) / `CheckpointThresholdInMB` headroom; no setting disables the reader a build must hold. | The log reader is released automatically when the build completes or is cancelled, unblocking truncation. |
| | Backup | A backup holds a log reader while reading state and log, blocking truncation until it completes — the same path as a replica build. | [`MaxAccumulatedBackupLogSizeInMB`](/dotnet/api/microsoft.servicefabric.data.reliablestatemanagerreplicatorsettings.maxaccumulatedbackuplogsizeinmb) bounds how much log a backup chain may accumulate before it is forced to fail. | A backup that would exceed `MaxAccumulatedBackupLogSizeInMB` is failed, releasing the reader and relieving log pressure rather than letting it stall commits indefinitely. |
| **Primary replication queue full** | — | The primary holds each replicated operation in a bounded queue and cannot free it until *every* secondary has acknowledged it — operations are retained for lagging replicas to catch up, even after a write quorum has already acknowledged and the commit has completed. A single slow, unhealthy, or still-building secondary is therefore enough to fill the queue, at which point new commits are rejected as a transient queue-full error. This is enforced by the underlying replication engine, separately from the log throttle. | [`MaxPrimaryReplicationQueueSize`](/dotnet/api/system.fabric.replicatorsettings.maxprimaryreplicationqueuesize) and [`MaxReplicationQueueMemorySize`](/dotnet/api/system.fabric.replicatorsettings.maxreplicationqueuememorysize) set the bounded-queue limits. Raising them gives slower secondaries more time to catch up before commits are rejected, at the cost of higher memory usage on the primary. | The replicator restarts a secondary it detects is too slow — gated on queue usage crossing a configured threshold (or the oldest-operation age) — so the queue can drain. It does so only while a write quorum is still maintained, so it never sacrifices availability to relieve the throttle. |

| Aspect | Behavior |
|--------|----------|
| **During the stall** | Commits are *rejected* with a transient error, not served slowly. Throughput and commit-completion count collapse and the error rate spikes; CPU drops. Latency of *successful* commits does not climb here — there are simply few or no completions to measure. |
| **After the stall** | A recovery burst drains the backlog: queued in-flight transactions and retried operations all complete in a tight window, each carrying the time it spent blocked or retrying as end-to-end latency. |
| **Tail impact** | The P95/P99 spike comes from the recovery burst and from end-to-end (attempt-to-success) latency that includes retries — not from the stall window producing slow completions. If you measure only the single-call latency of successful commits, the stall shows up as a throughput hole plus error-rate spike, with the tail spiking when the backlog drains. P50 can still look normal if only a small fraction of operations hit the stall window. |

> The more transactions are in flight when a stall hits, the more freeze at once. Lower
> concurrency shrinks the blast radius.

## Do

- **Do bound the number of in-flight transactions** with an explicit gate (such as a
  `SemaphoreSlim`, a bounded channel, or a TPL Dataflow block) rather than fanning out
  unbounded `CommitAsync` calls. A rough cap of ~10,000 concurrent transactions is a
  reasonable starting limit; adjust up or down based on observed performance.
  See [Bounding in-flight transactions](#bounding-in-flight-transactions).
- **Do keep transactions short.** Perform any external or long-running work *before* opening
  the transaction so it never holds locks or pins the log.
- **Do tune the checkpoint threshold to your write rate** so checkpoints happen regularly but
  not constantly, avoiding large disruptive bursts of checkpoint and truncation work. If you
  don't want to instrument this yourself, Reliable Collections already expose a
  [`Log Flush Bytes/sec`](/azure/service-fabric/service-fabric-reliable-services-diagnostics#performance-counters)
  performance counter (under the *Service Fabric Transactional Replicator* category) that shows
  your sustained write rate to the log. As a rule of thumb, adjust `CheckpointThresholdInMB` so
  your typical write rate fills the log to it roughly every 30-120 seconds.
- **Do monitor tail latency (P95/P99), not just P50.** Rising tail latency is usually the
  earliest warning of an impending stall. Alert on it.
- **Do keep the default API timeout** (or close to it) so genuine stalls and deadlocks surface
  quickly.
- **Do handle `InvalidOperationException` on transactions.** The system may abort a
  transaction (for example on a replica role change). Dispose it, re-check state, and retry
  with a new transaction.
- **Do use bounded retries with exponential backoff** so retries do not add to the queue depth
  causing the latency. See [Bounded retries with exponential backoff](#bounded-retries-with-exponential-backoff).
- **Do measure and minimize serialization cost** before production -- inefficient serializers
  inflate CPU and per-commit latency.
- **Do dispose transactions promptly** after commit to release resources quickly.

## Don't

- **Don't add concurrency expecting unbounded throughput gains.** Some concurrency does help:
  batching and pipelining let the replicator amortize per-commit costs, so throughput climbs
  with in-flight transactions up to the point where the pipeline saturates. Beyond that point
  the rate is fixed, and additional in-flight transactions only raise latency (and widen the
  blast radius of a stall) without improving throughput. Find that saturation point for your
  workload.
- **Don't run unbounded parallel commits against a single partition.** High in-flight counts
  raise latency and widen the blast radius of any stall.
- **Don't apply parallel or concurrent operations within a single transaction.** Only one
  user-thread operation per transaction is supported; fan-out (`Task.WhenAll`,
  `Parallel.ForEachAsync`) causes lock issues and memory leaks. Issue operations sequentially.
  See [Sequential operations within a transaction](#sequential-operations-within-a-transaction).
- **Don't hold a transaction open across long-running or blocking work** (HTTP calls, large
  CPU loops, external I/O). It holds locks, blocks log truncation, and prevents memory
  reclamation.
- **Don't use `TimeSpan.MaxValue` or excessively large timeouts.** They hide stalls and
  deadlocks and delay failure detection.
- **Don't create nested transactions**, and don't use an enumeration or newly created state
  outside the transaction that produced it.
  See [Avoiding nested transactions](#avoiding-nested-transactions).

## Avoid / Consider

- **Avoid hot keys.** Concentrated writes serialize behind a single lock; distribute writes
  across keys.
- **Avoid mixing single-entity and multi-entity operations** in one transaction -- differing
  isolation levels increase locking cost.
- **Consider keeping individual items small** (well under 80 KB), splitting large values into
  multiple rows to reduce flush, replication, checkpoint, and build cost.
- **Consider keeping the number of Reliable Collections per partition modest** (well under a
  thousand), preferring more items over more collections.

## Code patterns

The following examples are illustrative pseudo-code. They show the shape of each pattern, not a
copy-paste-ready implementation — adapt the gate sizes, retry counts, and exception handling to
your workload.

### Bounding in-flight transactions

Gate concurrency with a `SemaphoreSlim` so that at most `maxInFlight` transactions are ever
outstanding. The gate blocks new work once the limit is reached, instead of letting unbounded
`CommitAsync` calls pile into the replication queue. See
[Do bound the number of in-flight transactions](#do).

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

Retry only transient failures (`FabricTransientException` covers throttling and queue-full), cap
the attempt count, and grow the delay exponentially so retries do not pile back onto a queue
that is already under pressure. See
[Do use bounded retries with exponential backoff](#do).

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

A transaction supports only one user-thread operation at a time. Fanning out concurrent
operations on the same transaction causes lock issues and memory leaks; issue them sequentially
instead. See
[Don't apply parallel or concurrent operations within a single transaction](#dont).

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

Do not open a second transaction inside the scope of the first. Commit the outer transaction
before starting the next, or perform both operations within a single transaction. See
[Don't create nested transactions](#dont).

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

## Further reading

- [Guidelines and recommendations for Reliable Collections in Azure Service Fabric](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines)
- [Transactions and locks in Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections-transactions-locks)
- [Reliable State Manager configuration](/azure/service-fabric/service-fabric-reliable-services-configuration)
- [Backup and restore for Reliable Services](/azure/service-fabric/service-fabric-reliable-services-backup-restore)
