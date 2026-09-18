---
title: Migrate to the v6 and v7 VM series with a wave-based runbook
description: A wave-based runbook for moving Azure VM workloads to the v6 and v7 series — choose between redeploy and the in-place upgrade, then execute the pilot, waves, and rollback.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 07/24/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Migrate to the v6 and v7 VM series with a wave-based runbook

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

**Workload patterns:** ✔️ E. Customer-managed VMs · F. Stateful and clustered · G. Certified appliances — ❌ Not for A, B, C, or D

## Introduction

This runbook consumes the output from **Discover**, **Assess**, and **Plan**. Ensure you complete these phases before the pilot:

- The workload's [migration pattern](sizes-v6-v7-migration-discover.md), which decides whether this per-VM runbook applies at all.
- A [readiness score](sizes-v6-v7-migration-assess.md) for every workload in scope, with any remediation assigned and closed.
- A [plan](sizes-v6-v7-migration-plan.md): target size, region and zone, quota or capacity reservation, image approach, disk-path remediation, and commercial replan.
- The **execution method** for each workload — redeploy or in-place upgrade. This is the [deployment approach decision](sizes-v6-v7-migration-plan.md#decide-a-migration-approach) from Plan; [Choose your execution method](#choose-your-execution-method) describes what each one means at execution time.
- A maintenance window, rollback decision points, and workload-owner approval.

> [!NOTE]
> **Who this article is for.** This runbook replaces **individual VMs**. If your unit of replacement is a pool, a host pool, a cluster, or a service-managed SKU, the wave model still applies — prove the pattern once, then expand in rings — but perform the replacement through your service's process. See [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md#which-phases-apply-to-your-pattern).

## Where you start

Workloads arrive from different starting points, but they all converge on the same path. Find your row, do the one thing in the last column, then [choose your execution method](#choose-your-execution-method) and run the [pilot](#phase-1-pilot).

| Your starting point | What's different | Do this first |
| --- | --- | --- |
| **Generation 1** | The v6 and v7 series require Generation 2 (UEFI), and the only supported in-place route to Generation 2 is the Gen1 to Trusted Launch upgrade — there's no plain Gen1 to Gen2 conversion. | Plan a Generation 2 image (highly recommended), or take the in-place upgrade path (which for a Gen1 source starts with the [Gen1 to Trusted Launch upgrade](/azure/virtual-machines/trusted-launch-existing-vm-gen-1) followed by the SCSI-to-NVMe conversion). |
| **Generation 2** | Often close to ready — the boot mode is already right. | Confirm the image is NVMe- and MANA-ready. |
| **Already on NVMe** | Prerequisites are usually satisfied. | Confirm the target size, then go straight to [choosing your execution method](#choose-your-execution-method) — image remediation is usually already done. |
| **Greenfield** | Nothing to remediate. | Deploy from a current Generation 2, NVMe- and MANA-ready image. |

## Choose your execution method

| | Redeploy from image (highly recommended) | In-place upgrade |
| --- | --- | --- |
| **What happens** | A new VM is created from a current Generation 2, NVMe-, and MANA-ready image. The old VM keeps running until the new one validates. | The existing VM is transformed where it stands. A Generation 2 source takes one operation: the [SCSI-to-NVMe conversion](scsi-to-nvme-migration.md) with a resize to the target series. A Generation 1 source takes two, in order: the [Gen1 to Trusted Launch upgrade](/azure/virtual-machines/trusted-launch-existing-vm-gen-1), then the SCSI-to-NVMe conversion. The OS disk is kept throughout. |
| **When to choose it** | The default, especially for production. It's repeatable at scale, rollback is simply keeping the old VM, and it folds in the image refresh you need anyway. | Choose it when you can't practically rebuild because licensing or activation is bound to the OS disk, in-guest configuration is costly to reproduce, or no maintainable image exists. |
| **Eligibility** | Any workload that can deploy from an image. | A source size **without** any OS dependency on the local temporary disk (no `d`-suffix); no Azure Disk Encryption; OS already NVMe-ready. Generation 1 sources also carry the [Trusted Launch upgrade prerequisites](/azure/virtual-machines/trusted-launch-existing-vm-gen-1#unsupported-gen1-vm-configurations) — Windows Server 2016 and older aren't supported, Azure Backup must use the Enhanced policy rather than Standard, and the OS volume can't be encrypted during the upgrade. |
| **OS-disk data** | Doesn't carry over — plan the [capture-and-restore step](sizes-v6-v7-migration-plan.md#persistent-application-data-on-the-os-disk). | Carries over, but the device paths the OS sees still change. |
| **Rollback** | Keep the old VM until the rollback window closes. | Split by operation: the SCSI-to-NVMe conversion reverts by switching back to SCSI and the original size, with the disks untouched. The Trusted Launch upgrade **doesn't revert** — recovery from that step is a full restore from the pre-upgrade backup. |

> [!WARNING]
> Both operations behind the in-place upgrade are supported platform operations. For more information, see [Upgrade Gen1 VMs to Trusted Launch](/azure/virtual-machines/trusted-launch-existing-vm-gen-1) and [Convert SCSI to NVMe for Linux and Windows VMs](/azure/virtual-machines/nvme-linux). However, Microsoft doesn't officially support the community script that automates SCSI-to-NVMe conversion. Validate these processes against non-production VMs before using them in a production wave, and always run the script with a backup and a tested revert in hand. For the script, its parameters, and the revert procedure, see [Convert a VM from SCSI to NVMe in place](scsi-to-nvme-migration.md).

> [!IMPORTANT]
> A Generation 1 source makes the in-place upgrade a chain of two one-way-leaning operations on the same production VM. That's the maximal-risk version of this method — prefer redeploy for Gen1 sources unless one of the "when to choose it" conditions genuinely applies.

### Stateful and clustered workloads (F)

The platform steps are the same as any other VM. The sequencing is what differs: replace nodes side by side and let the application move the role, rather than cutting over a VM.

1. **Confirm the hard gate first.** For SAP, only sizes on the SAP-certified list are supportable, regardless of platform readiness.
2. **Add, don't replace.** Deploy a new node or replica on the target series and join it to the cluster, availability group, or replication topology.
3. **Let synchronization finish.** Don't proceed on a partial sync, however long the rollback window looks.
4. **Validate before you move anything.** Confirm quorum, replication health, storage latency, and monitoring on the new node.
5. **Transfer the role deliberately** during the window — a planned failover or role transfer, not an induced one.
6. **Keep the old node until the rollback window closes.** Then remove it and reconfirm quorum.

Never migrate all members of a quorum-based cluster in a single wave, and don't leave the cluster with an even number of voting members between steps.

For standalone stateful VMs that can't take a replica, the [execution method choice](#choose-your-execution-method) applies after all: use a redeploy with an application-consistent backup and restore, or the in-place upgrade as the maintenance-window migration. Either way, rehearse the restore before the production cutover.

> [!IMPORTANT]
> **Don't clone or restore a domain controller** from an image or backup of another domain controller. Deploy a new domain controller on the target series, let directory and SYSVOL replication populate it, validate domain and DNS health, transfer the operations master (FSMO) roles, then demote the old one. Update static references — DNS client settings, and anything pinned to a specific domain controller — before you demote.

### Cut over a certified appliance (G)

Vendor certification is settled during planning — see [Confirm before you commit](sizes-v6-v7-migration-plan.md#confirm-before-you-commit). Don't start this sequence until you have written confirmation for the exact target family. What follows is the cutover itself.

1. **Deploy in parallel.** Stand up the new appliances alongside the existing pair, and reproduce licensing and bootstrap configuration.
2. **Reproduce policy and routing.** Rule sets, routes, IP forwarding, and health probes — then diff them against the source rather than assuming the export was complete.
3. **Verify the datapath.** Pass controlled test traffic through the new appliances before any production traffic moves.
4. **Exercise failover on the new pair.** Don't infer high-availability behavior from the old pair; the target family might present NICs differently.
5. **Shift traffic, then retire.** Move production traffic, hold the old appliances through the rollback window, and retire them after sign-off.

## Phase 1: Pilot

The pilot is wave zero: one representative workload, run end to end, to prove the pattern the later waves repeat. Confirm the [prerequisites](#introduction) are met, including the execution method chosen for this workload. The pilot sequence differs by method:

| Step | Redeploy from image | In-place upgrade |
| --- | --- | --- |
| **1. Protect** | Take an application-consistent restore point of the source VM, and confirm the inventory of anything the application persists to the OS disk. | Restart the VM to commit any pending changes and prove it boots cleanly, then take an application-consistent restore point. For a Generation 1 source, this backup is the only way back from the Trusted Launch step. |
| **2. Execute** | Deploy the v6/v7 instance from the updated image, alongside the source VM, which keeps running. | Deallocate the VM, then run the operations in order: a Generation 1 source takes the [Gen1 to Trusted Launch upgrade](/azure/virtual-machines/trusted-launch-existing-vm-gen-1) first, then the [SCSI-to-NVMe conversion](scsi-to-nvme-migration.md) with the resize to the target series. Start the VM. |
| **3. Validate** | On the **new** VM: boot, disk discovery and mounts, NVMe and MANA drivers healthy, network connectivity. | The same checks, on the **transformed** VM — plus confirm no mount, script, or application setting still references an old SCSI device path. |
| **4. Cut over** | Install apps and services, migrate data from the source, redirect clients, and confirm owner sign-off. | No data copy — the disks came along. Confirm the workload is up and the owner signs off. |
| **5. Hold the rollback** | Keep the source VM running until the rollback window closes, then retire it. | Keep the pre-upgrade backup until the window closes. If validation fails, revert the conversion to SCSI; a failed Trusted Launch step needs the restore. |

The downtime profile differs too: redeploy takes an outage only at cutover, while the in-place upgrade takes the VM down for the whole execute-and-validate window. Set the pilot's maintenance window accordingly.

For [stateful and clustered workloads (F)](#stateful-and-clustered-workloads-f) and [certified appliances (G)](#cut-over-a-certified-appliance-g), the **Execute** and **Cut over** rows are replaced by the pattern-specific sequence. Protect, Validate, and the rollback hold apply as written.

Close the pilot the same way for both methods: capture any findings, and lock the repeatable pattern for later waves.

### Pilot success criteria

- The VM boots cleanly (no boot-diagnostics errors).
- OS and data disks are present and mounted as planned.
- NVMe and MANA drivers are loaded and healthy.
- Network connectivity passes.
- The workload owner confirms the application is up.

These criteria are the in-window gates — the subset of [platform validation](sizes-v6-v7-migration-validate.md#platform-validation) that decides proceed-or-roll-back while the revert is still cheap. The full pass — [operational validation](sizes-v6-v7-migration-validate.md#operational-validation) and the pattern's [closure criteria](sizes-v6-v7-migration-validate.md#closure-criteria-by-pattern) — happens in [Validate and optimize](sizes-v6-v7-migration-validate.md), and it completes **before the rollback window closes**.

## Phase 2: Migrate in waves

Settle the wave sequence during [planning](sizes-v6-v7-migration-plan.md#wave-sequencing). The wave sequence determines which applications move together and in what order. This phase executes the wave sequence.

1. Keep waves small enough to validate within the window.
2. Reconfirm capacity in the target region and zone at the start of each wave.
3. Don't split tightly coupled application tiers across old and new families for longer than the wave window.

### Automate the repeatable work

1. Build the inventory with [Azure Resource Graph](/azure/governance/resource-graph/overview), [Azure Migrate](/azure/migrate/migrate-services-overview), and tags. See the [inventory query](sizes-v6-v7-migration-assess.md#inventory-query-azure-resource-graph) in Assess.
2. Use infrastructure as code and pipelines for repeatable deployment, and Azure Image Builder with Azure Compute Gallery for image rebuilds.
3. Steer new deployments with [Azure Policy](/azure/governance/policy/overview) allowed-SKU and image rules.
4. Roll out in rings (canary, pilot, production), and keep the old VMs and images until the new ones are validated.

### Per-wave steps

For each VM in the wave, run the method sequence the pilot locked in. The [Phase 1 table](#phase-1-pilot) shows the per-VM procedure. At wave level:

1. Confirm wave readiness, and recheck the [prerequisites](#introduction) for every workload. This check includes each in-place VM's eligibility, because configuration drift since planning (a new data disk type, encryption enabled, a size change) can break the conversion.
2. Work through the per-VM sequence for each workload.
3. Record results per VM. Proceed only after the exit criteria are met.

What changes at wave scale is how the two methods batch:

| | Redeploy from image | In-place upgrade |
| --- | --- | --- |
| **Batching** | New VMs can deploy in parallel where dependencies allow. The sources keep serving throughout. | Each VM is down for its entire transform. Batch by maintenance window, not by count. |
| **Capacity** | Old and new VMs run side by side, so the wave needs quota and capacity for **both** sets at once. | No double footprint, but the resize must find target-size capacity at execution time. Reconfirm at wave start. |
| **Rollback stock** | The source VMs. Keep them until the wave's exit criteria pass. | The pre-upgrade restore points. Keep them until the wave's exit criteria pass. |

### Wave exit criteria

1. The in-window platform gates pass (boot, disk, network).
2. The workload owner confirms the application is healthy.
3. The full [validation pass](sizes-v6-v7-migration-validate.md) is complete – operational validation and the pattern's closure criteria – while the rollback stock still exists.
4. The rollback window closes with approval, and the migration record is updated.

## Phase 3: Close and expand

1. Capture the migrated count and scope; document any one-time remediations.
2. Release the rollback stock deliberately: retire and delete the source VMs and their disks (redeploy), and expire the pre-upgrade restore points per your retention policy (in-place upgrade). Don't let either accumulate silently – both carry cost.
3. Review cost and performance, and rightsize where headroom exists. See [Validate and optimize](sizes-v6-v7-migration-validate.md).
4. Identify the next waves or additional v6/v7 candidates.

## Rollback guidance

Plan rollback before you start, and match the plan to the execution method:

| Method | Redeploy | In-place upgrade |
| --- | --- | --- |
| **How you roll back** | Redirect clients back to the source VM, which you never touched. Retire the new VM. | Run the conversion back to SCSI and the original size; the disks are untouched. The Trusted Launch step (Gen1 sources) **doesn't roll back** — recovery from it is a full restore from the pre-upgrade backup. |
| **What it costs** | Nothing but the cutover window — no restore involved, unless the source is already retired. | A second outage for the revert; a full restore if the Trusted Launch step is what failed. |

Document the trigger conditions, decision owner, latest rollback time, backup reference, and communication plan.

## Escalation triggers

For either method:

- The VM fails to boot after the deployment or the transform.
- The OS doesn't discover disks as expected.
- An NVMe or MANA driver or connectivity issue occurs.
- Secure Boot blocks a required low-level driver.
- Required zone capacity is unavailable.

Specific to the in-place upgrade:

- The Trusted Launch step completes but the VM doesn't boot — restore from the pre-upgrade backup; don't attempt the conversion on top of a failed upgrade.
- The conversion doesn't validate and the revert to SCSI also fails — restore from the pre-conversion backup and fall back to the redeploy path.

## Communicate with stakeholders

Keep workload owners and the sponsor informed at each wave boundary: what's planned for the wave, what was validated (boot, storage, networking, application health, and backup), and what comes next. Confirm the rollback window is closed before you consider a wave complete.

## Next steps

- [5. Validate and optimize](sizes-v6-v7-migration-validate.md)
