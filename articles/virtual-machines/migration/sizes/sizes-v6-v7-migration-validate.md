---
title: Validate and optimize after migrating to the v6 and v7 VM series
description: Confirm a successful move to the v6 and v7 series, boot, storage, networking, and workload sign-off, then optimize cost and performance.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 07/03/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Validate and optimize after migrating to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

Validation for a move to the v6 or v7 series is focused and quick because only the platform changed. Confirm the VM boots, disks attach over NVMe, networking is healthy on MANA, and the workload owner sees the application running. Then capture the performance and cost benefit.

## Validation model

Two layers are enough:

- **Platform validation:** boot, disks, drivers, and networking. Owned by the migration team.
- **Workload confirmation:** the application is up and serving. A simple owner sign-off, not a full regression.

## Platform validation

| Area | What to confirm | Success signal |
| --- | --- | --- |
| Boot | The VM starts after deployment. | No boot-diagnostics errors. |
| Drivers | NVMe and MANA drivers load. | Both present and healthy. |
| Disks | OS and data disks are visible and mounted. | Count, size, and mount points match the plan. |
| Disk paths | No references to old SCSI paths. | Stable identifiers resolve correctly. |
| Local/temp disk | Local NVMe disk present and formatted where used. | Page file, `tempdb`, or scratch in place; no missing-disk errors. |
| Network | NIC and IP configuration correct. | Connectivity tests pass. |
| Performance | CPU, memory, disk, and network within the expected range. | Meets or beats the prior baseline. |

> [!NOTE]
> On v6 and v7 sizes, the local NVMe temporary disk is presented raw and unformatted and is re-created on every stop/deallocate cycle, it isn't persistent. If a workload relies on it, confirm your boot-time task initializes and formats it, and that the page file or `tempdb` lands where you expect.

## Workload confirmation

A short owner sign-off:

- The application starts and is reachable.
- The workload behaves as it did on the previous family.
- Platform monitoring shows the VM healthy.

There's no need to revalidate flows that the size change doesn't touch.

## Operational validation

- The Azure VM Agent is present and healthy on the newly deployed VM. Reprovision and verify the extensions that don't carry forward on a fresh OS disk: boot diagnostics, the monitoring or Log Analytics agent, backup integration, and security agents.
- Monitoring and backup agents are reporting and current.
- A backup job runs and a restore point is visible ([Generation 2 and Trusted Launch support](/azure/backup/backup-support-matrix-iaas) confirmed).
- Runbooks are updated for the new family and any changed disk paths.

## Post-migration optimization

This is where the price-performance benefit is realized. Optimize in order:

1. **Rightsize compute.** The v6 and v7 series often deliver the same work with fewer or smaller vCPUs. Use [Azure Advisor](/azure/advisor/advisor-cost-recommendations) and observed usage to resize down where there's headroom.
1. **Tune storage.** Confirm IOPS and throughput against the [target size limits](/azure/virtual-machines/nvme-overview) and adjust the disk tier or caching.
1. **Confirm commercial coverage.** Apply or [exchange reservations or savings plans](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so the new family is covered, and review spend in [Cost Management](/azure/cost-management-billing/costs/overview-cost-management).
1. **Review resiliency.** Confirm zone placement still meets your availability target.
1. **Feed lessons into the next wave.**

## Metrics to capture

- **Leading:** candidates assessed, percentage passing readiness, pilots completed, and sign-off on the plan.
- **Lagging:** workloads running on v6/v7, migration success rate, rollback rate, performance delta versus the prior family, price-performance outcome, and next-wave candidates.

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

## When to pause before the next wave

- The pilot needed an emergency rollback.
- Disk-path or driver issues surfaced late.
- Backup isn't healthy on the new configuration.
- Required region or zone capacity changed.
- A remediation wasn't repeatable and needs to be standardized first.

## Next steps

- [Frequently asked questions](sizes-v6-v7-migration-faq.md)
- [Azure Advisor cost recommendations](/azure/advisor/advisor-cost-recommendations)
