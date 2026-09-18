---
title: Frequently asked questions about migration to v6 and v7 VM series
description: Answers to common questions about moving Azure VM workloads to the v6 and v7 series — which guidance applies, Generation 2 conversion, disk encryption, technical gates, rollback, tooling, and cost.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: faq
ms.date: 07/24/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Frequently asked questions about migration to v6 and v7 VM series

This article answers common questions about moving workloads to the v6 and v7 Azure VM series.

## Which parts of this guidance apply to my workload?

That depends on who owns the image and what you replace. The Assess and Plan articles cover image and VM-level remediation, so they apply only when you own the image. The Migrate runbook replaces individual VMs, so it applies only when that's your unit of replacement.

- **Pool-replaced and service-managed compute** (AKS and Batch pools, Databricks, Data Explorer, HDInsight): the platform supplies the image and already meets the Generation 2, NVMe, and MANA prerequisites. Your check is short — confirm the size is in the service's supported list, confirm region, zone, and quota — and then you replace the pool or cluster through your service's process.
- **Non-persistent session hosts** (Azure Virtual Desktop pooled host pools, Citrix DaaS random catalogs, Omnissa Horizon floating instant clones): Assess and Plan apply to the golden image; hosts are then replaced by ring rather than by VM. Persistent desktops — AVD personal host pools, Citrix static catalogs, Horizon dedicated assignments — bind a user to a specific VM and migrate as customer-managed VMs instead.
- **Customer-managed, stateful, clustered, and appliance workloads:** the full journey applies.

 To find which guidance applies to your workload, start with [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md).


## Is moving to v6 or v7 a normal VM resize?

No, treat it as a platform migration. The target VM can change the boot mode, storage interface, networking, image requirements, and regional or zonal availability. Validate readiness first, then migrate in controlled waves so boot, storage, networking, application behavior, monitoring, and rollback are all covered.

## Can a Generation 1 VM be converted to Generation 2 in place?

Yes — but only through the [Gen1 to Trusted Launch upgrade](/azure/virtual-machines/trusted-launch-existing-vm-gen-1); Trusted Launch is enabled as part of the conversion, and upgrading to Generation 2 without it isn't supported. The disk layout is converted in-guest with MBR2GPT first, then the security-type change flips the VM to Generation 2 (UEFI). vTPM is enabled by default; Secure Boot is optional, so workloads with unsigned kernels or filter drivers can leave it off.

Know the constraints before choosing this path: there's no rollback to Generation 1 except a full restore from a pre-upgrade backup; Windows Server 2016 isn't supported (upgrade the guest OS first); Azure Backup must use the Enhanced policy; and the OS volume can't be encrypted during the upgrade. For how this fits the migration, see [Choose your execution method](sizes-v6-v7-migration-migrate.md#choose-your-execution-method).

## Why move to a newer VM series?

A newer series gives you access to current Azure infrastructure, better price-performance, and a more future-ready foundation. When a newer generation is already available in your region, it's the better fit for future workloads. Plan the move around outcomes and readiness rather than the size name.

## Is zone support always available?

No. Availability varies by SKU, family, zone, and region. Confirm the required region and availability zone needs before you commit the architecture.

## When is a regional (non-zonal) deployment appropriate?

A regional deployment can be appropriate when the region doesn't offer Availability Zones.

## Should we use v6 or v7 for all future waves?

Yes. If v6 or v7 meets your technical, regional, zonal, and commercial requirements, plan future waves around the same approach for consistency. If later waves have different workload or resiliency requirements, reassess rather than assuming the first-wave pattern applies everywhere.

## What are the most important technical gates?

The core gates are:

- Generation 2 (UEFI) readiness.
- An NVMe-ready OS and image.
- A MANA-ready OS and drivers.
- Disk-path remediation (replace hard-coded SCSI paths).
- A temporary-disk dependency review.
- An Azure Disk Encryption review — ADE isn't available on the v6 and v7 series at all.
- Target region and zone availability.
- Backup and rollback readiness.

Two workload types have a gate that comes *before* this list. For SAP, only sizes on the SAP-certified list are supportable, regardless of platform readiness. For ISV virtual appliances, the vendor certifies specific families, NIC layouts, and disk presentation. If either applies, confirm certification first — the rest of the checklist is moot without it.

For the full list, see [Assess readiness](sizes-v6-v7-migration-assess.md).

## What if we use custom images?

Validate and update custom images before migration. A custom image that works on older VM generations might not be prepared for NVMe or the v6/v7 deployment requirements. Standardize on [Azure Image Builder](/azure/virtual-machines/image-builder-overview) and [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) so you make the fix once and reuse it.

## What if a workload depends on `/dev/disk/azure/scsi1/lunX` paths?

Those paths don't exist after the move to NVMe-based disk presentation. Identify the dependencies and replace them with stable identifiers (by-UUID references or `/dev/disk/azure/*` symlinks) before migration.

## What if the workload uses SQL Server tempdb on the local temporary disk?

Don't assume the same temporary-disk behavior on v6 and v7. The local disk exists only on `d`-suffixed sizes, is presented as NVMe, and is re-created raw on every stop/deallocate cycle. Review `tempdb` placement, performance requirements, and persistence expectations, and move or redesign the dependency before migration if needed.

## What if security or antivirus software uses filter drivers?

Review those agents before migration. Filter drivers, especially on Windows, can cause stability or boot issues when the storage interface changes, and Secure Boot requires signed drivers. Confirm a current, signed version and validate in a pilot before production.

## What if the VM uses Azure Disk Encryption (BitLocker or dm-crypt)?

Azure Disk Encryption isn't available on the v6 or v7 series for either operating system, and it's scheduled for retirement on 15 September 2028 — after that date, encrypted disks fail to unlock after a reboot. The replacement is [encryption at host](/azure/virtual-machines/disk-encryption).

Plan for a rebuild rather than a conversion: encryption at host can't be enabled on a VM that currently has, or has ever had, ADE — and the restriction survives decryption, snapshots, and disk copies. The supported path is new disks and a new VM. For the constraints, decisions, and procedure, see [Azure Disk Encryption and encryption at host](sizes-v6-v7-migration-plan.md#azure-disk-encryption-and-encryption-at-host) in Plan.

## Can you roll back?

Plan and test rollback before migration, and match it to the [execution method](sizes-v6-v7-migration-migrate.md#choose-your-execution-method):

- **Redeploy:** rollback is "keep the old VM or pool until the new one is validated, then retire it" — redirect clients back; no restore involved.
- **In-place upgrade:** the SCSI-to-NVMe conversion reverts by switching the controller back to SCSI and the original size, with the disks untouched. The Gen1 to Trusted Launch step **doesn't revert** — recovery from it is a full restore from the backup taken before the upgrade.
- **Clustered workloads:** rollback means transferring the role back and keeping the old node until the window closes, not restoring an image. Domain controllers in particular must never be restored or cloned from another domain controller — see [Stateful and clustered workloads](sizes-v6-v7-migration-migrate.md#stateful-and-clustered-workloads-f).

Either way, keep a documented plan with a backup or restore point, an owner, a maintenance window, and decision criteria. See [Rollback guidance](sizes-v6-v7-migration-migrate.md#rollback-guidance).

## What tooling is involved?

The SCSI-to-NVMe conversion behind the [in-place upgrade](sizes-v6-v7-migration-migrate.md#choose-your-execution-method) is a supported platform operation. The automation most teams use for it is a community script — validate it against a non-production VM before using it in a production wave, and always run it with a backup and a tested revert in hand. To learn more, see [Convert a VM from SCSI to NVMe in place](scsi-to-nvme-migration.md). Tooling automates checks and conversion steps, but it doesn't replace workload-dependency assessment or application validation.

## Should we take the backup with the VM running or powered off?

Powered off — deallocated — is the only state where a raw disk snapshot is guaranteed consistent, because individual managed-disk snapshots are taken per disk with no cross-disk coordination. On a running multi-disk VM, anything spanning disks (a striped volume, a database with data and log separated) can restore inconsistent.

You usually don't need the outage, though — you need the right mechanism. Use [VM restore points](/azure/virtual-machines/virtual-machines-create-restore-points), which capture all the VM's disks as a set, and prefer **application-consistent** over crash-consistent. That choice matters especially here: Microsoft documents that crash-consistent restore points on Intel v6+ and AMD v7+ VMs **with more than one data disk might not be consistent across disks**.

For a migration cutover, the question is often moot because you're deallocating anyway. Quiesce the application, shut it down to Stop (deallocated), and then take the restore point. Whichever route you take, restore the backup in a test before the wave instead of only confirming that the backup exists.

## What if v6 or v7 aren't available in the target region or zone?

Confirm whether you can use a regional deployment, another zone, another region, a fallback size, or a phased timing plan. Don't commit to a zonal architecture until availability is validated. Consider [capacity reservations](/azure/virtual-machines/capacity-reservation-overview) for predictable supply.

## What about cost?

Use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) for current pricing, and frame the decision around efficiency, performance, and capacity. Because the v6 and v7 series deliver better performance, you often need fewer or smaller instances for the same workload, see [Post-migration optimization](sizes-v6-v7-migration-validate.md#post-migration-optimization).

## How should Azure Virtual Desktop, AKS, Databricks, or scale-set workloads move?

Move these workloads through the platform's own rollout mechanism, not through the per-VM runbook. Replace the pool, host pool, or cluster rather than resizing instances, and keep the old one until the new one is validated. Avoid one-off manual changes across large pools — use image versioning, health checks, and rollback controls.


One exception: this guidance applies only where hosts are non-persistent. An Azure Virtual Desktop **personal** host pool binds each user to a specific VM, so there's no pool to replace. Those workloads migrate one VM at a time, like any customer-managed VM. The same guidance applies to Citrix static (dedicated) catalogs and Omnissa Horizon dedicated assignments.

For the sequence and references for each of these workloads, see [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md).


## Should we use Azure Image Builder?

Use [Azure Image Builder](/azure/virtual-machines/image-builder-overview), or an equivalent governed image pipeline, when you need repeatable custom images. It's especially valuable for scale sets, Azure Virtual Desktop pooled host pools, golden images, regulated environments, and large estates where you must test, version, promote, and roll back the same image consistently.


## What if we use multiple VM sizes?

Multi-size strategies can improve capacity flexibility, but you must validate them. Confirm that the workload tolerates the selected CPU families, memory ratios, disk limits, networking limits, and performance differences. For AKS, use separate node pools with labels and taints. For Azure Virtual Desktop, separate host pools or rings can simplify the user experience. For tightly coupled stateful tiers, avoid multi-size designs unless the application is tested across the full allowed set.

## How should marketplace or ISV appliances be handled?

Confirm vendor support before production migration. For products such as firewalls, monitoring tools, endpoint security, backup, or packet-capture appliances, validate the exact version, image, licensing model, NIC count, accelerated networking, throughput, failover, logging, and support path. If you deploy the product from Azure Marketplace, confirm the publisher image supports the target VM family. If you directly procure the product, confirm support with the vendor. See [ISV virtual appliances](sizes-v6-v7-migration-plan.md#isv-virtual-appliances) in Plan for the certification checklist, and [Cut over a certified appliance](sizes-v6-v7-migration-migrate.md#cut-over-a-certified-appliance-g) for the cutover sequence. You usually migrate an ISV appliance by redeploying new from the Marketplace.

## How should large migrations be approached?

For hundreds or thousands of VMs, use automation and governance:

- Build the inventory with [Azure Resource Graph](/azure/governance/resource-graph/overview), [Azure Migrate](/azure/migrate/migrate-services-overview) discovery, dependency data, and tags.
- Group by application dependency and migration pattern.
- Use automated pre-flight checks for OS, image, driver, disk-path, extension, and agent readiness.
- Use infrastructure as code, pipelines, runbooks, or platform-specific rollout mechanisms.
- Use Azure Monitor, Log Analytics, health probes, and application tests for validation.
- Track readiness, exceptions, wave status, rollback decisions, and post-migration health in a shared system.

For the estate-scale approach, see [Migrate in waves](sizes-v6-v7-migration-migrate.md#phase-2-migrate-in-waves).

## Next steps

- [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md)
