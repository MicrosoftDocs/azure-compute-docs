---
title: Frequently asked questions about migrating to the v6 and v7 VM series
description: Answers to common questions about moving Azure VM workloads to the v6 and v7 series, resize versus migration, zonal availability, technical gates, tooling, and cost.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: faq
ms.date: 07/03/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Frequently asked questions about migrating to the v6 and v7 VM series

This article answers common questions about moving workloads to the v6 and v7 Azure VM series. For the full planning guidance, see [Plan a workload migration](sizes-v6-v7-migration-plan.md).

## Is moving to v6 or v7 a normal VM resize?

No, treat it as a platform migration. The target VM can change the boot mode, storage interface, networking, image requirements, and regional or zonal availability. Validate readiness first, then migrate in controlled waves so boot, storage, networking, application behavior, monitoring, and rollback are all covered.

## Why move to a newer VM series?

A newer series gives you access to current Azure infrastructure, better price-performance, and a more future-ready foundation. When a newer generation is already available in your region, it's the better fit for future workloads. Plan the move around outcomes and readiness rather than the size name.

## Is zonal support always available?

No. Availability varies by size, region, and zone. Confirm the required region and availability-zone needs before you commit the architecture.

## When is a regional (non-zonal) deployment appropriate?

A regional deployment can be appropriate when it meets your SLA, resiliency, compliance, and operational requirements. For most production workloads, use zone redundancy. If you expect zonal resiliency and the target size is regional-only in the target location, pause and revisit the architecture.

## Should we use v6 or v7 for all future waves?

Yes. If v6 or v7 meets your technical, regional, zonal, and commercial requirements, plan future waves around the same approach for consistency. If later waves have different workload or resiliency requirements, reassess rather than assuming the first-wave pattern applies everywhere.

## What are the most important technical gates?

The core gates are:

- Generation 2 (UEFI) readiness.
- An NVMe-ready OS and image.
- A MANA-ready OS and drivers.
- Disk-path remediation (replace hard-coded SCSI paths).
- A temporary-disk dependency review.
- Target region and zone availability.
- Backup and rollback readiness.

For the full list, see [Assess readiness](sizes-v6-v7-migration-assess.md).

## What if we use custom images?

Validate and update custom images before migration. A custom image that works on older VM generations might not be prepared for NVMe or the v6/v7 deployment requirements. Standardize on [Azure Image Builder](/azure/virtual-machines/image-builder-overview) and [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) so you make the fix once and reuse it.

## What if a workload depends on `/dev/disk/azure/scsi1/lunX` paths?

Those paths don't exist after the move to NVMe-based disk presentation. Identify the dependencies and replace them with stable identifiers (by-UUID references or `/dev/disk/azure/*` symlinks) before migration.

## What if the workload uses SQL Server tempdb on the local temporary disk?

Don't assume the same temporary-disk behavior on v6 and v7. The local disk exists only on `d`-suffixed sizes, is presented as NVMe, and is re-created raw on every stop/deallocate cycle. Review `tempdb` placement, performance requirements, and persistence expectations, and move or redesign the dependency before migration if needed.

## What if security or antivirus software uses filter drivers?

Review those agents before migration. Filter drivers, especially on Windows, can cause stability or boot issues when the storage interface changes, and Secure Boot requires signed drivers. Confirm a current, signed version and validate in a pilot before production.

## Can you roll back?

Plan and test rollback before migration. Because you deploy alongside the existing instances, rollback is usually "keep the old VM or pool until the new one is validated, then retire it." The in-place conversion can also switch the controller back from NVMe to SCSI, but this approach isn't recommended for production. Either way, keep a documented plan with a backup or snapshot, an owner, a maintenance window, and decision criteria.

## What tooling is involved?

For non-production workloads, for in-place moves, a community script switches a VM's controller from SCSI to NVMe. To learn more, see [Convert a VM from SCSI to NVMe in place](scsi-to-nvme-migration.md). Tooling automates checks and conversion steps, but it doesn't replace workload-dependency assessment or application validation.

## What if v6 or v7 aren't available in the target region or zone?

Confirm whether you can use a regional deployment, another zone, another region, a fallback size, or a phased timing plan. Don't commit to a zonal architecture until availability is validated. Consider [capacity reservations](/azure/virtual-machines/capacity-reservation-overview) for predictable supply.

## What about cost?

Use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) for current pricing, and frame the decision around efficiency, performance, and capacity. Because v7 delivers better performance, you often need fewer or smaller instances for the same workload, see [Post-migration optimization](sizes-v6-v7-migration-validate.md#post-migration-optimization).

## How should Azure Virtual Desktop, AKS, Databricks, or scale-set workloads move?

Use the rollout pattern that fits the platform:

- **Azure Virtual Desktop:** deploy a validated image to a canary host pool or ring, use drain mode, then replace hosts in stages.
- **AKS:** add a new v6 or v7 node pool, validate workloads, then cordon and drain old nodes.
- **Azure Databricks:** validate representative clusters and jobs, then update cluster policies or pools.
- **Virtual Machine Scale Sets:** validate a new image or scale-set model, then roll out by canary, rolling upgrade, or parallel scale set.

Avoid one-off manual changes across large pools. Use image versioning, health checks, and rollback controls.

## Should we use Azure Image Builder?

Use [Azure Image Builder](/azure/virtual-machines/image-builder-overview), or an equivalent governed image pipeline, when you need repeatable custom images. It's especially valuable for scale sets, Azure Virtual Desktop host pools, golden images, regulated environments, and large estates, where the same image must be tested, versioned, promoted, and rolled back consistently.

## What if we use multiple VM sizes?

Multi-size strategies can improve capacity flexibility, but you must validate them. Confirm that the workload tolerates the selected CPU families, memory ratios, disk limits, networking limits, and performance differences. For AKS, use separate node pools with labels and taints. For Azure Virtual Desktop, separate host pools or rings can simplify the user experience. For tightly coupled stateful tiers, avoid multi-size designs unless the application is tested across the full allowed set.

## How should marketplace or ISV appliances be handled?

Confirm vendor support before production migration. For products such as firewalls, monitoring tools, endpoint security, backup, or packet-capture appliances, validate the exact version, image, licensing model, NIC count, accelerated networking, throughput, failover, logging, and support path. If the product is deployed from Azure Marketplace, confirm the publisher image supports the target VM family; if it's directly procured, confirm support with the vendor. See [ISV network and storage appliances](sizes-v6-v7-migration-plan.md#isv-network-and-storage-appliances). The migration of an ISV appliance is usually achieved by redeploying new from the Marketplace.

## How should large migrations be approached?

For hundreds or thousands of VMs, use automation and governance:

- Build the inventory with [Azure Resource Graph](/azure/governance/resource-graph/overview), [Azure Migrate](/azure/migrate/migrate-services-overview) discovery, dependency data, and tags.
- Group by application dependency and migration pattern.
- Use automated pre-flight checks for OS, image, driver, disk-path, extension, and agent readiness.
- Use infrastructure as code, pipelines, runbooks, or platform-specific rollout mechanisms.
- Use Azure Monitor, Log Analytics, health probes, and application tests for validation.
- Track readiness, exceptions, wave status, rollback decisions, and post-migration health in a shared system.

For the estate-scale approach, see [Sequence and automate estate-scale migrations](sizes-v6-v7-migration-plan.md#sequence-and-automate-estate-scale-migrations).

## Next steps

- [Assess readiness for the v6 and v7 series](sizes-v6-v7-migration-assess.md)
- [Plan a workload migration](sizes-v6-v7-migration-plan.md)
