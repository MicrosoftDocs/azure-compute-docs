---
title: Validate and optimize after modernizing to the v6 and v7 VM series
description: Confirm a successful move to the v6 and v7 series, boot, storage, networking, and workload sign-off, then optimize cost and performance.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/24/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Validate and optimize after modernizing to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

**Workload patterns:** ✔️ All patterns — platform validation is universal; closure criteria vary by pattern

This article describes the full validation process. It finishes **before the wave's rollback window closes**, while the rollback stock — the source VM or the pre-upgrade restore point — still exists. The process captures the performance and cost benefits.

The checks are the same whether a VM was [redeployed from an image or upgraded in place](sizes-v6-v7-migration-migrate.md#choose-your-execution-method). If a step differs by method, the article calls it out.

## Validation model

Three layers:

- **Platform validation:** boot, disks, drivers, and networking. The modernization team owns this layer, and it's identical for every pattern — including nodes inside a pool.
- **Workload confirmation:** the application is up and serving. For most workloads, this layer requires a simple owner sign-off, not a full regression.
- **Closure criteria:** what "done" means for your unit of replacement. This layer differs by pattern. This layer is where a wave is most often declared complete too early.

## Platform validation

| Area | What to confirm | Success signal |
| --- | --- | --- |
| Boot | The VM starts after the deployment or the in-place upgrade. | No boot diagnostics errors. |
| Drivers | NVMe and MANA drivers load. | Both present and healthy. |
| Disks | OS and data disks are visible and mounted. | Count, size, and mount points match the plan. |
| Disk paths | No references to old SCSI paths. | Stable identifiers resolve correctly. |
| Local/temp disk | Local NVMe disk present and formatted where used. | Page file, `tempdb`, or scratch in place; no missing-disk errors. |
| Network | NIC and IP configuration correct. | Connectivity tests pass. |
| Performance | CPU, memory, disk, and network within the expected range. | Meets or beats the prior baseline. |

> [!NOTE]
> On v6 and v7 sizes, the local NVMe temporary disk is presented raw and unformatted and is re-created on every stop/deallocate cycle, it isn't persistent. If a workload relies on it, confirm your boot-time task initializes and formats it, and that the page file or `tempdb` lands where you expect.

### Transitions from the v3 series

A transition from Dv3, Dsv3, Ev3, or Esv3 usually changes every row in the platform validation table: Generation 1 to Generation 2 boot, SCSI to NVMe storage, standard networking to MANA, and a formatted temporary disk to a raw local NVMe disk on `d`-suffixed sizes. Validate each row explicitly, and compare performance against a baseline captured on the v3 source.

The Dv3, Dsv3, Ev3, and Esv3 series retire on November 15, 2029. Close validation and the rollback window before that date, because you can't run a v3 source VM for rollback after retirement. For retirement details, see the [Retired VM sizes modernization guide](../../sizes/lifecycle/retirement/retired-sizes-modernization-guide.md).

## Workload confirmation

A short owner sign-off:

- The application starts and is reachable.
- The workload behaves as it did on the previous family.
- Platform monitoring shows the VM healthy.

> [!IMPORTANT]
> Two patterns need more than a sign-off.
> - For **stateful and clustered workloads**, replication and quorum behavior *is* the thing that changed operationally. Validate it explicitly rather than inferring health from the application being reachable.
> - For **certified appliances**, exercise the datapath and failover, don't assume them. See [Closure criteria by pattern](#closure-criteria-by-pattern).

## Closure criteria by pattern

Platform validation tells you the VM is healthy. Closure criteria tell you the wave is finished. They depend on what you replaced — the workload's [modernization pattern](sizes-v6-v7-migration-discover.md).

| Pattern | A wave is complete when |
| --- | --- |
| **A. Compute pools** | Workloads are rescheduled onto the new pool, the old pool is drained and removed, and no workload is pinned to the old node selector, taint, or label. |
| **B. Image-based hosts**| The new image version is published, replacement hosts serve sessions, old hosts are drained with no active sessions, and the old hosts are removed. |
| **C. Service-managed compute** | The service reports the new node type or SKU active, jobs and queries run at expected performance, and local scratch behavior is confirmed. |
| **D. Cluster re-creation** | Jobs and pipelines run against the new cluster with output parity, external storage and metastore connections are verified, and the old cluster is retired. |
| **E. Customer-managed VMs** | The application serves from the modernized VM, OS-disk data is in place and verified, clients are redirected, and the rollback stock is released after the window — the old VM retired (redeploy), or the pre-upgrade restore points expired per retention (in-place upgrade). |
| **F. Stateful and clustered** | Quorum is healthy at the target node count, replication is caught up with no backlog, a failover has been exercised, roles are transferred, and the old node is removed. |
| **G. Certified appliances** | Traffic passes through the new appliances, policy and routing match the source, high-availability failover is exercised, and the old pair is retired. |

## Operational validation

**Applies to:** ✔️ B. Image-based hosts ✔️ E. Customer-managed VMs ✔️ F. Stateful and clustered ✔️ G. Certified appliances — for service-managed compute, the service owns the agent and extension lifecycle.

The first check depends on the [execution method](sizes-v6-v7-migration-migrate.md#choose-your-execution-method):

- **Redeployed VMs** start from a fresh OS disk, so the extensions don't carry forward. Confirm the Azure VM Agent is present and healthy, then reprovision and verify boot diagnostics, the monitoring or Log Analytics agent, backup integration, and security agents.
- **In-place upgraded VMs** keep their OS disk, so agents carry forward — confirm they're healthy after the controller change. For Gen1 sources, note a documented limitation of the Trusted Launch upgrade: the VM's **image reference still shows the source Gen1 image**. Automatic guest patching keys off the image reference, and a reimage of the VM will fail. Record these VMs so patching is managed deliberately.

For every VM, regardless of method:

- Monitoring and backup agents are reporting and current.
- A backup job runs and a restore point is visible ([Generation 2 and Trusted Launch support](/azure/backup/backup-support-matrix-iaas) confirmed — Trusted Launch VMs require the Enhanced backup policy).
- Runbooks are updated for the new family and any changed disk paths.

## Post-modernization optimization

This is where the price-performance benefit is realized. Optimize in order:

1. **Rightsize compute.** The v6 and v7 series often deliver the same work with fewer or smaller vCPUs. Use [Azure Advisor](/azure/advisor/advisor-cost-recommendations) and observed usage to resize down where there's headroom.
2. **Tune storage.** Confirm IOPS and throughput against the [target size limits](/azure/virtual-machines/nvme-overview) and adjust the disk tier or caching.
3. **Confirm commercial coverage.** Apply or [exchange reservations or savings plans](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so the new family is covered, and review spend in [Cost Management](/azure/cost-management-billing/costs/overview-cost-management).
4. **Review resiliency.** Confirm zone placement still meets your availability target.
5. **Modernize to PaaS.** Evaluate Azure PaaS services.
6. **Feed lessons into the next wave.**

## Metrics to capture

- **Leading:** candidates assessed, percentage passing readiness, pilots completed, and sign-off on the plan.
- **Lagging:** workloads running on v6/v7, modernization success rate, rollback rate, performance delta versus the prior family, price-performance outcome, and next-wave candidates.

## Validation checklist

| Item | Owner | Status | Notes |
| --- | --- | --- | --- |
| VM boots cleanly |  |  |  |
| NVMe driver healthy |  |  |  |
| MANA driver healthy |  |  |  |
| Disks visible and mounted |  |  |  |
| Disk paths resolve (no SCSI references) |  |  |  |
| Local/temp disk in place where used |  |  |  |
| Network connectivity passes |  |  |  |
| Performance baseline met or beaten |  |  |  |
| Backup job completes |  |  |  |
| Reservation or savings coverage applied |  |  |  |
| Workload owner confirms the app is up |  |  |  |
| In-place upgraded Gen1 VMs: image-reference limitation recorded, patching plan confirmed |  |  |  |
| v3 sources: rollback window closes before the November 15, 2029 retirement |  |  |  |
| [Closure criteria](#closure-criteria-by-pattern) met for the pattern |  |  |  |

## When to pause before the next wave

- The pilot needed an emergency rollback.
- Disk-path or driver issues surfaced late.
- Backup isn't healthy on the new configuration.
- Required region or zone capacity changed.
- A remediation wasn't repeatable and needs to be standardized first.

## Next steps

- [Frequently asked questions](sizes-v6-v7-migration-faq.md)
