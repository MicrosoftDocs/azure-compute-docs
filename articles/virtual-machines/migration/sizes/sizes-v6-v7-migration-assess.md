---
title: Assess readiness for the v6 and v7 VM series
description: Qualify candidate workloads for the v6 and v7 series and confirm a clean migration path, boot mode, image, storage, networking, and capacity.
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

# Assess readiness for the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

To move to the v6 or v7 series, start with a quick readiness check, not a blind resize. Because the application inside the VM is unchanged, readiness is a short, well-understood checklist focused on the platform underneath: boot mode, image, storage interface, network driver, and regional and zonal availability.

The assessment sorts each workload into one of three outcomes:

| Outcome | Meaning | Next step |
| --- | --- | --- |
| **Ready now** | Prerequisites are satisfied. | Move to a pilot. |
| **Ready after a small refresh** | A small, known change is needed, usually an image or driver update, or a disk-path fix. | Remediate, then retest. |
| **Plan the region or family** | A hard blocker, such as VM SKU not available yet in the required region. | Adjust the region, family, or timing. |

This is one of the lower-risk upgrades you can run. Once the new image is prepared and validated, the pattern is highly repeatable.

## When the v6 and v7 series are a strong fit

- The workload needs better price-performance from current silicon.
- The workload runs on a supported, reasonably current operating system, or can take a one-time image refresh.
- You can deploy from an updated image rather than requiring an in-place resize.
- The target family is available in the required region. When a newer generation is already available in your region, choose it rather than an older one, it's the better fit for future workloads.

## Greenfield deployments

For new deployments, you don't need to fix anything. Choose the v6 or v7 series intentionally and start clean:

- Confirm [region and zone availability](/azure/reliability/availability-zones-overview) and quota for the chosen family.
- Note a fallback size, zone, or region before launch.
- Use infrastructure as code ([Bicep](/azure/azure-resource-manager/bicep/overview) or Terraform) for repeatability.
- Enable [Trusted Launch](/azure/virtual-machines/trusted-launch) (the default for Generation 2) for Secure Boot and vTPM.
- Deploy from a current Generation 2, NVMe-ready, MANA-ready marketplace or [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) image.

Greenfield deployments can skip the remediation guidance that follows and go straight to sizing and capacity in [Plan the migration](sizes-v6-v7-migration-plan.md).

## Brownfield migrations from the v2 through v5 series

### What changes, and what doesn't

Keep the assessment focused. Only the platform underneath the VM changes. Your application, its configuration, autoscale policies, load balancing, and internal health logic are unaffected, so don't spend assessment time revalidating them as if this were an application upgrade.

| Changes with the family | Doesn't change with the family |
| --- | --- |
| Boot mode (Generation 2, UEFI, Trusted Launch) | Application code and configuration |
| Disk interface (NVMe) and disk device paths | Business logic, user flows, APIs |
| Network driver (MANA) | Autoscale rules and scaling thresholds |
| Local and temporary disk presence and format | Load balancer, ingress, and DNS topology |
| Image driver readiness | Identity and authentication design |
| Low-level agents that install kernel or filter drivers | Application-layer monitoring dashboards |

### Readiness signals to check

Most of these signals are one-time and easy to fix.

| Signal | Why it matters | Recommended action |
| --- | --- | --- |
| Generation 1 source (common on v2/v3) | The v6 and v7 series require Generation 2 (UEFI). | Plan a Generation 2 image, or use the [Generation 1 to Generation 2 upgrade](/azure/virtual-machines/generation-2) before sizing. |
| OS NVMe support | The OS must discover disks over [NVMe](/azure/virtual-machines/nvme-overview). | Confirm a supported OS version and refresh the image if needed. |
| Custom image not NVMe/Generation 2 ready | The image might not boot or attach storage on the target size. | Rebuild or update the image; test boot and disk discovery once. |
| MANA driver readiness | Networking uses the [MANA](/azure/virtual-network/accelerated-networking-mana-overview) adapter. | Confirm OS and driver support (see version floors under [Discovery checklist](#discovery-checklist)). |
| Hard-coded SCSI disk paths | Scripts or agents that reference `/dev/disk/azure/scsi*` need stable IDs. | Switch to stable identifiers (by-UUID or Azure disk symlinks). |
| Persistent data on the OS disk | Cross-generation moves redeploy from a fresh image, so anything an app writes straight to the OS volume (configuration, license/activation files, local databases, certificates, or state) doesn't carry over. | Inventory what each workload persists to the OS disk and plan a capture-and-restore step, or relocate it to a managed data disk or external store. See [Persistent application data on the OS disk](sizes-v6-v7-migration-plan.md#persistent-application-data-on-the-os-disk). |
| Temporary-disk dependency | A local or temporary disk exists only on `d`-suffixed sizes and is presented as NVMe. A source VM that already has a temporary disk (for example, `E32ds_v5`) can't convert in place directly to a v6 size. | Choose a `d`-size if local scratch is needed, and relocate the page file or `tempdb` accordingly. Where the source has a temporary disk, plan a redeploy path rather than a direct conversion. |
| Azure Disk Encryption (ADE) for Linux | ADE for Linux isn't supported with NVMe, so converting an ADE-encrypted Linux VM to NVMe produces an unsupported configuration. | Decrypt the VM before you convert, or use the redeploy-from-image path. Check for ADE yourself first, the community SCSI-to-NVMe conversion script doesn't currently detect ADE and can complete without warning. |
| Local NVMe temporary disk isn't auto-formatted | On v6 and v7 sizes, the local NVMe temporary disk is presented raw and unformatted and is re-created on every stop/deallocate cycle. Nothing left on it is persistent, and on Windows the `D:` mapping can be lost after a deallocate/start cycle. | Initialize and format the local disk on each start (for example, a boot-time task), and keep nothing persistent there. |
| Region and zone requirement | Availability varies by region and zone. | Confirm size availability in the Region  before committing the design. |
| Availability and quota for the new family | The v6 and v7 series use distinct quota families. | Request quota early, verify capacity for the target size in the required regions and zones, and use [capacity reservations](/azure/virtual-machines/capacity-reservation-overview) for guaranteed supply. |
| Family-scoped reservations and savings plans | Discounts are scoped to a VM family. | Replan [reservations or savings plans](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so coverage follows the move. |
| Low-level ISV agents (antivirus, backup, monitoring with kernel or filter drivers) | Secure Boot and NVMe can affect unsigned or older drivers. | Confirm a current, signed agent version. |
| Marketplace or ISV network appliance | Vendors certify specific VM families and NIC/driver configurations. | Confirm vendor support for the target family and image. |

### Platform-managed workloads

For fleets that deploy VMs for you, the move is a new pool or image version, not a per-VM resize. Validation stays focused on the platform layer.

| Pattern | What to do | What to confirm (platform only) |
| --- | --- | --- |
| Azure Virtual Desktop host pools | Build a v6 or v7 golden image, add canary hosts, drain old hosts, and expand by ring. | Image boots, sign-in works, profiles attach, and agents are healthy. |
| AKS node pools | Add a new v6 or v7 node pool, then cordon and drain old nodes. | Nodes provision, boot, and join; pods schedule; node drivers are healthy. |
| Azure Databricks | Point cluster policies and pools at a supported v6 or v7 size. | Clusters launch; local NVMe scratch behaves as expected; capacity is available. |
| Virtual Machine Scale Sets | Deploy a new image or model via a canary or parallel scale set, then roll forward. | Instances boot from the new image; capacity and rollback path are confirmed. |

> [!NOTE]
> Pod disruption budgets, ingress rules, autoscale thresholds, and application health probes aren't changed by the VM size. You don't need to revalidate them as part of this move.

## Discovery checklist

Keep the checklist lean and platform-focused.

**Scope**

- Confirm sponsor and workload owner.
- Document business driver (capacity, performance, cost, or modernization).
- Identify source family (v2/v3/v4/v5) and deployment type (migration or greenfield).
- Separate production and non-production.
- Agree on migration window and success/rollback criteria.

**Compute and capacity**

- Select current size and target v6/v7 size.
- Confirm VM generation (Generation 2 path planned if the source is Generation 1).
- Confirm target region and zone requirement.
- Confirm quota and availability per wave; consider [capacity reservation](/azure/virtual-machines/capacity-reservation-overview).
- Note fallback size or region.

**OS and image**

- Support OS version; image is Generation 2, NVMe, and MANA ready.
- Rebuild and validate custom gallery images once; enable boot diagnostics for the pilot.
- Use [Azure Image Builder](/azure/virtual-machines/image-builder-overview) and [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) for image build and versioning to ensure repeatability.

**Storage**

- Inventory OS and data disks; check disk-path references for hard-coded SCSI paths.
- Review temporary and local disk use; confirm page file, `tempdb`, and scratch placement.
- Take backup or snapshot before cutover; confirm [Azure Backup support](/azure/backup/backup-support-matrix-iaas) for Generation 2 and Trusted Launch.

**Networking**

- Confirm MANA OS and driver readiness.
- For network appliances: confirm NIC count, IP forwarding, and vendor support for the family.

> [!NOTE]
> **MANA OS version floors**
>
> - **Linux:** MANA Ethernet drivers are upstream in Linux kernel 5.15 and later. Kernel 6.2 and later adds support for advanced features such as RDMA/InfiniBand and DPDK. Prefer a recent kernel from an [endorsed distribution](/azure/virtual-network/accelerated-networking-mana-linux).
> - **Windows:** Use current Windows Server images with the in-box MANA driver and the latest updates.
> - MANA is also rolling out to some existing VM series, so confirming OS compatibility benefits the whole estate, not just the v6 and v7 series.

**Commercial**

- Replan family-scoped reservations and savings plans so discount coverage follows the workload.

## Exit criteria

- Workloads are scored as ready now, ready after a small refresh, or plan the region or family.
- Target family, region, zone, and quota or capacity plan are documented.
- Any small remediation is assigned.
- Pilot (or greenfield first deployment) is selected.
- Rollback approach is noted, and workload owners approve the first wave.

## Next steps

- [Plan a workload migration to the v6 and v7 series](sizes-v6-v7-migration-plan.md)
- [Capacity reservations overview](/azure/virtual-machines/capacity-reservation-overview)
