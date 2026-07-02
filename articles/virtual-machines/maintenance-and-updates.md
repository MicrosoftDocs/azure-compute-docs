---
title: Maintenance and updates
description: Overview of maintenance and updates for virtual machines running in Azure.
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 04/30/2026
#pmcontact:shants
# Customer intent: As a cloud administrator, I want to understand the maintenance processes for virtual machines, so that I can effectively manage uptime and minimize disruptions during scheduled updates.
---
# Maintenance for virtual machines in Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure periodically updates its platform to improve the reliability, performance, and security of the host infrastructure for virtual machines. The purpose of these updates ranges from patching software components in the hosting environment to upgrading networking components or decommissioning hardware. 

Updates rarely affect the hosted VMs. When updates do have an effect, Azure chooses the least impactful method for updates:

- If the update doesn't require a reboot, the VM is paused while the host is updated, or the VM is live-migrated to an already updated host. 
- If maintenance requires a reboot, you're notified of the planned maintenance. Azure also provides a time window in which you can start the maintenance yourself, at a time that works for you. The self-maintenance window is typically 35 days (for Host machines) unless the maintenance is urgent. Azure is investing in technologies to reduce the number of cases in which planned platform maintenance requires the VMs to be rebooted. For instructions on managing planned maintenance, see Handling planned maintenance notifications using the Azure [CLI](maintenance-notifications-cli.md), [PowerShell](maintenance-notifications-powershell.md) or [portal](maintenance-notifications-portal.md).

This page describes how Azure performs both types of maintenance. For more information about unplanned events (outages), see [Manage the availability of VMs for Windows](./availability.md) or the corresponding article for [Linux](./availability.md).

Within a VM, you can get notifications about upcoming maintenance by [using Scheduled Events for Windows](./windows/scheduled-events.md) or for [Linux](./linux/scheduled-events.md).



## Maintenance that doesn't require a reboot

Most platform updates don't affect customer VMs. When a no-impact update isn't possible, Azure chooses the update mechanism that's least impactful to customer VMs. 

When VM impacting maintenance is required it will almost always be completed through a VM pause for less than 10 seconds. In rare circumstances, no more than once every 18 months for general purpose VM sizes, Azure uses a mechanism that will pause the VM for about 30 seconds. After any pause operation the VM clock is automatically synchronized upon resume. 

Memory-preserving maintenance works for more than 90 percent of Azure VMs. It doesn't work for G, L, N, and H series. For more information, see [which VM sizes support memory-preserving maintenance](./sizes/overview.md). Azure increasingly uses live-migration technologies and improves memory-preserving maintenance mechanisms to reduce the pause durations.  

These maintenance operations that don't require a reboot are applied one fault domain at a time. They stop if they receive any warning health signals from platform monitoring tools. Maintenance operations that do not require a reboot may occur simultaneously in paired regions or Availability Zones. For a given change, the deployment are mostly sequenced across Availability Zones and across Region pairs, but there can be overlap at the tail.

These types of updates can affect some applications. When the VM is live-migrated to a different host, some sensitive workloads might show a slight performance degradation in the few minutes leading up to the VM pause. To prepare for VM maintenance and reduce impact during Azure maintenance, try [using Scheduled Events for Windows](./windows/scheduled-events.md) or [Linux](./linux/scheduled-events.md) for such applications. 

For greater control on all maintenance activities including zero-impact and rebootless updates, you can create  a Maintenance Configuration feature. Creating a Maintenance Configuration gives you the option to skip all platform updates and apply the updates at your choice of time. For more information, see [Managing platform updates with Maintenance Configurations](maintenance-configurations.md).


### Live migration

Live migration is an operation that doesn't require a reboot and that preserves memory for the VM. It causes a pause or freeze, typically lasting no more than 5 seconds. Except for G, L, N, and H series, all infrastructure as a service (IaaS) VMs, are eligible for live migration. Live migration is available on majority of M-Series SKUs. Eligible VMs represent more than 90 percent of the IaaS VMs that are deployed to the Azure fleet. 

> [!NOTE]
> You won't receive a notification in the Azure portal for live migration operations that were attempted or don't require a reboot. To see a list of live migrations that don't require a reboot, [query for scheduled events](./windows/scheduled-events.md#query-for-events).
>
> Live migration is performed on a best effort basis. In some rare cases, live migration may not succeed and the VM will be scheduled to be Service Healed, if required, prior to notification. Live migration is not a guaranteed operation.

The Azure platform triggers live migration in the following scenarios:
- Planned maintenance
- Hardware failure
- Allocation optimizations

Some planned-maintenance scenarios use live migration, and you can use Scheduled Events to know in advance when live migration operations will start. 

Live migration can also be used to move VMs when Azure Machine Learning algorithms predict an impending hardware failure or VM allocations optimization. For more information about predictive modeling that detects instances of degraded hardware, see [Improving Azure VM resiliency with predictive machine learning and live migration](https://azure.microsoft.com/blog/improving-azure-virtual-machine-resiliency-with-predictive-ml-and-live-migration/?WT.mc_id=thomasmaurer-blog-thmaure). Live-migration notifications appear in the Azure portal in the Monitor and Service Health logs as well as in Scheduled Events if you use these services.

#### TCP connection resilience during live migration

Applications that maintain long-lived TCP connections, such as database servers, message brokers, and caching layers, can experience connection disruption during live migration. While the VM pause is typically under 5 seconds, the TCP stack behavior during and after the pause can extend application-level recovery time if not addressed.

**How live migration affects TCP connections:**

- During the pause, in-flight TCP segments aren't acknowledged by the migrating VM.
- The sending side (client or load balancer health probe) begins TCP retransmission with exponential backoff.
- Azure Standard Load Balancer sends a TCP RST to idle connections that exceed the configured idle timeout. However, for active connections with in-flight data, the load balancer does not send a TCP RST during the migration pause. The connection remains open but unresponsive, and the client has no immediate signal of failure.
- Without application-level tuning, the default TCP retransmission behavior (`tcp_retries2 = 15` on Linux) can delay connection failure detection by approximately 15 minutes.

> [!IMPORTANT]
> The impact varies significantly by operating system defaults. On Linux, `tcp_retries2` defaults to 15, resulting in approximately 15 minutes before a dead connection is detected. On Windows, `TcpMaxDataRetransmissions` defaults to 5, which limits detection time to approximately 25-50 seconds without any tuning. The mitigations described in this article are most critical for Linux-based workloads.

> [!NOTE]
> For HTTP/1.1 workloads, the impact is typically limited: only requests in-flight at the moment of migration are affected, and since HTTP/1.1 clients don't pipeline over keep-alive connections, they recover quickly by opening a new connection for the next request. For HTTP/2, the blast radius is wider because multiple concurrent streams share a single TCP connection.
>
> When the load balancer operates in L4 TLS passthrough mode, it cannot inspect, retry, or inject error responses into the encrypted stream. In this configuration, the client is solely responsible for detecting and recovering from the stalled connection.

**Reduce blast radius with multi-instance deployments:**

Before applying TCP-level mitigations, consider the architectural baseline. Live migration affects one VM at a time within an availability set or virtual machine scale set. Spreading connections across multiple backend instances limits the impact of any single migration event:

- A scale set with three instances means each migration event affects at most one-third of active connections.
- Deploying across Availability Zones ensures migrations in different zones don't overlap.
- Clients with connection pools distributed across multiple backends recover faster because unaffected connections continue serving requests immediately.

During the VM pause, Azure Standard Load Balancer health probes to the paused backend also fail. The load balancer marks the backend as unhealthy within approximately 10 seconds (two consecutive probe failures at the default 5-second interval) and stops routing new connections to it. This condition means new connections are naturally protected. The TCP mitigations described in this article address existing connections that were already established before the migration began.

**Recommended mitigations:**

The following mitigations are complementary. When implemented together, they reduce the impact of a live migration event from minutes of potential downtime to seconds of automatic recovery.

| Priority | Mitigation | Effort | Impact |
|----------|-----------|--------|--------|
| 1 | Set `TCP_USER_TIMEOUT` at the socket level | Low | Reduces dead connection detection from ~15 minutes to 30 seconds |
| 2 | Subscribe to Scheduled Events | Medium | Enables proactive connection draining before the freeze occurs |
| 3 | Tune TCP keepalive parameters | Low | Detects idle connections that go stale after migration |
| 4 | Implement client-side retry logic | Medium | Provides resilience regardless of root cause |

**Mitigation 1: TCP_USER_TIMEOUT (fastest detection)**

`TCP_USER_TIMEOUT` controls how long the kernel waits for acknowledgment of transmitted data before declaring a connection dead. Setting this to 30 seconds (30000 ms) per socket significantly reduces detection time.

```c
// Per-socket (recommended)
int timeout = 30000; // 30 seconds in milliseconds
setsockopt(fd, IPPROTO_TCP, TCP_USER_TIMEOUT, &timeout, sizeof(timeout));
```

Alternatively, reduce the system-wide retransmission count:

```bash
# /etc/sysctl.conf — reduces retransmit ceiling to ~25-50 seconds
net.ipv4.tcp_retries2 = 5
```

> [!TIP]
> Set `TCP_USER_TIMEOUT` at the SDK or socket level rather than system-wide. A value of 30 seconds is a good starting point. Values below 10 seconds may cause false positives during normal network jitter.

**Windows considerations:**

The `TCP_USER_TIMEOUT` socket option is specific to Linux. On Windows, TCP retransmission behavior is controlled differently:

- Windows defaults to 5 retransmissions (`TcpMaxDataRetransmissions`), which already provides approximately 25-50 seconds of detection time without any tuning.
- To further reduce detection time on Windows, adjust the registry:

```powershell
# Reduce TCP retransmissions (system-wide, requires reboot)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" `
    -Name "TcpMaxDataRetransmissions" -Value 3 -Type DWord
```

With `TcpMaxDataRetransmissions` set to 3, the detection time reduces to approximately 10-20 seconds depending on the initial retransmission timeout.

> [!NOTE]
> Unlike Linux, Windows doesn't expose a per-socket equivalent of `TCP_USER_TIMEOUT`. The registry setting applies to all TCP connections on the system. For granular control on Windows, rely on application-level timeouts and health checks (Mitigation 4).

**Mitigation 2: Scheduled Events (proactive drain)**

The [Scheduled Events](./linux/scheduled-events.md) service provides advance notice before a live migration begins. Applications can listen for `Freeze` events and proactively drain connections before the pause occurs.

```
GET http://169.254.169.254/metadata/scheduledevents?api-version=2020-07-01
Headers: Metadata: true
```

A live migration event appears as:

```json
{
  "EventType": "Freeze",
  "ResourceType": "VirtualMachine",
  "Resources": ["myVM"],
  "EventStatus": "Scheduled",
  "NotBefore": "2026-04-29T18:00:00Z"
}
```

When a `Freeze` event is detected:
1. Stop accepting new connections on the affected node.
2. Drain existing connections (signal clients to reconnect to other nodes).
3. Wait for in-flight operations to complete with a bounded timeout.
4. Optionally acknowledge the event by posting back the EventId.

> [!NOTE]
> The advance notice period is typically 15 minutes but can be as short as 30 seconds in rare cases. A poll frequency of once per second is recommended for production workloads.

**Mitigation 3: TCP keepalive tuning**

TCP keepalive probes detect connections that become idle after the migration event:

```bash
net.ipv4.tcp_keepalive_time = 30      # seconds before first probe (default: 7200)
net.ipv4.tcp_keepalive_intvl = 10     # seconds between probes (default: 75)
net.ipv4.tcp_keepalive_probes = 3     # probes before declaring dead (default: 9)
```

With these settings, an idle stale connection is detected within 60 seconds (30 + 10 x 3). Keepalive probes also count as activity for the Standard Load Balancer idle timeout, preventing the load balancer from timing out idle connections independently.

**Mitigation 4: Client-side retry logic**

Application-level reconnection and retry logic ensures recovery regardless of the failure detection method:

1. Detect connection error (timeout, RST, or connection refused).
2. Close the dead connection and remove it from the connection pool.
3. Open a new connection to the same or a different node.
4. Retry the operation with exponential backoff.

For database SDKs and connection pools, enable periodic health checks (for example, a lightweight ping every 10-15 seconds) to validate connections proactively.

**Connection pool configuration:**

Connection pools that maintain long-lived connections benefit from a maximum lifetime setting. This forces periodic connection recycling, ensuring no single connection accumulates unbounded risk from future migration events:

| Pool Technology | Setting | Recommended Value |
|----------------|---------|-------------------|
| HikariCP (Java) | `maxLifetime` | 1800000 (30 minutes) |
| PgBouncer | `server_lifetime` | 1800 (30 minutes) |
| Go `database/sql` | `SetConnMaxLifetime` | 30 * time.Minute |
| Node.js (pg Pool) | `idleTimeoutMillis` | 30000 (30 seconds idle eviction; max-lifetime requires custom logic) |
| .NET `SqlConnection` | Connection string: `Connection Lifetime` | 1800 (30 minutes) |

Setting a maximum lifetime of 30 minutes means that even without active health checks, connections are naturally replaced before they can accumulate long periods of undetected staleness.

**Monitoring and observability:**

To detect and measure the impact of live migration events on TCP connections, use the following approaches:

- **Azure Monitor VM Availability metric (Preview):** Drops to 0 during the VM pause. Create an alert rule on `VmAvailabilityMetric` with a threshold of less than 1 to detect migration events.
- **Scheduled Events Activity Log:** Live migration events appear in the Activity Log under the `Microsoft.Compute` provider with operation name `Microsoft.Compute/virtualMachines/liveMigration/action` or as `Freeze` events when queried through the Metadata Service.
- **Application-level connection error rate:** Monitor TCP connection resets, timeouts, and reconnection counts in your application metrics. A spike in connection errors correlating with a VM Availability dip confirms migration impact.
- **TCP retransmission counters:** On Linux, monitor `/proc/net/netstat` field `TCPTimeouts` or use `ss -ti` to observe retransmission counts on individual sockets. Elevated retransmissions during a known maintenance window indicate connections were affected.

```bash
# Linux: Check TCP timeout statistics
cat /proc/net/netstat | grep -i timeout
# Or per-socket retransmission info
ss -ti | grep -i retrans
```

Establishing a baseline for these metrics during normal operation makes it straightforward to quantify the impact of migration events and validate that your mitigations are working as expected.

**Workloads with zero tolerance for live migration interruption**

For workloads that can't tolerate any interruption from live migration, consider using [Azure Dedicated Hosts](./dedicated-hosts.md) with [Maintenance Configurations](maintenance-configurations.md). Dedicated Hosts give you control over when host-level maintenance occurs, eliminating surprise live migration events.

## Maintenance that requires a reboot

In the rare case where VMs need to be rebooted for planned maintenance, you'll be notified in advance. Planned maintenance has two phases: the self-service phase and a scheduled maintenance phase.

During the *self-service phase*, which typically lasts four weeks, you start the maintenance on your VMs. As part of the self-service, you can query each VM to see its status and the result of your last maintenance request.

> [!NOTE]
> For VM-series that do not support [Live Migration](#live-migration), local (ephemeral) disks data can be lost during the maintenance events. See each individual VM-series for information on if Live Migration is supported. 

When you start self-service maintenance, your VM is redeployed to an already updated node. Because the VM is redeployed, the temporary disk is lost and public dynamic IP addresses associated with the virtual network interface are updated.

If an error arises during self-service maintenance, the operation stops, the VM isn't updated, and you get the option to retry the self-service maintenance. 

When the self-service phase ends, the *scheduled maintenance phase* begins. During this phase, you can still query for the maintenance phase, but you can't start the maintenance yourself.

For more information on managing maintenance that requires a reboot, see **Handling planned maintenance notifications** using the Azure [CLI](maintenance-notifications-cli.md), [PowerShell](maintenance-notifications-powershell.md) or [portal](maintenance-notifications-portal.md). 

### Availability considerations during scheduled maintenance 

If you decide to wait until the scheduled maintenance phase, there are a few things you should consider to maintain the highest availability of your VMs. 

#### Paired regions

Each Azure region is paired with another region within the same geographical vicinity. Together, they make a region pair. During the scheduled maintenance phase, Azure updates only the VMs in a single region of a region pair. For example, while updating the VM in North Central US, Azure doesn't update any VM in South Central US at the same time. However, other regions such as North Europe can be under maintenance at the same time as East US. Understanding how region pairs work can help you better distribute your VMs across regions. For more information, see [Azure region pairs](/azure/reliability/cross-region-replication-azure).

#### Availability zones

Availability zones are unique physical locations within an Azure region. Each zone is made up of one or more datacenters equipped with independent power, cooling, and networking. To ensure resiliency, there's a minimum of three separate zones in all enabled regions. 

An availability zone is a combination of a fault domain and an update domain. If you create three or more VMs across three zones in an Azure region, your VMs are effectively distributed across three fault domains and three update domains. The Azure platform recognizes this distribution across update domains to make sure that VMs in different zones are not updated at the same time.

Each infrastructure update rolls out zone by zone, within a single region. But, you can have deployment going on in Zone 1, and different deployment going in Zone 2, at the same time. Deployments are not all serialized. But, a single  deployment that requires a reboot only rolls out one zone at a time to reduce risk. In general, updates that require a reboot are avoided when possible, and Azure attempts to use Live Migration or provide customers control.

#### Virtual machine scale sets

Virtual machine scale sets in **Flexible** orchestration mode are an Azure compute resource allow you to combine the scalability of virtual machine scale sets in Uniform orchestration mode with the regional availability guarantees of availability sets.

With Flexible orchestration, you can choose whether your instances are spread across multiple zones, or spread across fault domains within a single region. 

#### Availability sets and Uniform scale sets

When deploying a workload on Azure VMs, you can create the VMs within an *availability set* to provide high availability to your application. Using availability sets, you can ensure that during either an outage or maintenance events that require a reboot, at least one VM is available.

Within an availability set, individual VMs are spread across up to 20 update domains. During scheduled maintenance, only one update domain is updated at any given time. Update domains aren't necessarily updated sequentially. 

Virtual machine *scale sets* in **Uniform** orchestration mode are an Azure compute resource that you can use to deploy and manage a set of identical VMs as a single resource. The scale set is automatically deployed across UDs, like VMs in an availability set. As with availability sets, when you use Uniform scale sets, only one UD is updated at any given time during scheduled maintenance.

For more information about setting up your VMs for high availability, see [Manage the availability of your VMs for Windows](./availability.md) or the corresponding article for [Linux](./availability.md).

## Next steps 

You can use the [Azure CLI](maintenance-notifications-cli.md), [Azure PowerShell](maintenance-notifications-powershell.md), or the [portal](maintenance-notifications-portal.md) to manage planned maintenance.
