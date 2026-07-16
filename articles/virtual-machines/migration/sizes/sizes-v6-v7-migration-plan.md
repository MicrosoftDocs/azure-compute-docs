---
title: Plan a workload migration to the v6 and v7 VM series
description: Considerations for migrating Azure VM workloads to the v6 and v7 series, including Generation 2, NVMe storage, MANA networking, hibernation, capacity, and commercial planning.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 07/03/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Plan a workload migration to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

The v6 and v7 Azure VM series are built on [Azure Boost](/azure/azure-boost/overview) and introduce a few platform changes compared with earlier generations: a Generation 2 (UEFI) foundation with Trusted Launch, NVMe-based storage, and the Microsoft Azure Network Adapter (MANA) for accelerated networking. Most workloads need only a one-time image refresh. Screen each workload against the considerations in this article before you migrate.

This article is for architects and infrastructure teams planning a move to the v6 or v7 series, for a single workload or across a large estate.

## Migration considerations at a glance

| Consideration | What changes | Typical action |
| --- | --- | --- |
| [Generation 2 and Trusted Launch](#generation-2-and-trusted-launch) | v6 and v7 use Generation 2 (UEFI) and support Trusted Launch | Confirm the generation; upgrade Generation 1 sources |
| [NVMe storage interface](#nvme-storage-interface) | Disks are presented over NVMe instead of SCSI | Use an NVMe-ready image; replace hard-coded device paths |
| [Image prerequisites](#image-prerequisites-for-generation-2-nvme-and-mana) | Generation 2, NVMe, and MANA readiness live in the image | Use current marketplace images or rebuild custom images |
| [MANA networking](#mana-networking) | Accelerated networking uses the MANA adapter | Meet the OS and driver version floors |
| [OS-disk data](#persistent-application-data-on-the-os-disk) | Redeploy starts from a fresh OS disk | Identify and migrate data persisted to the OS disk |
| [Local (temporary) disk](#local-temporary-disk-behavior) | Present only on `d`-suffixed sizes, as NVMe | Choose a `d`-size or use managed data disks |
| [Hibernation](#resume-hibernated-vms-before-you-migrate) | Saved memory state doesn't migrate | Resume and deallocate before you migrate |
| [Region, zone, and capacity](#region-zone-and-capacity-planning) | Availability and quota vary by family | Confirm availability; request quota early |
| [Commercial continuity](#commercial-continuity) | Reservations and savings plans are family-scoped | Replan or exchange coverage; rightsize |
| [ISV appliances](#isv-network-and-storage-appliances) | MANA and NVMe changes affect certified appliances | Confirm support with each vendor |
| [Sequencing and automation](#sequence-and-automate-estate-scale-migrations) | Estate moves are a program | Sequence by dependency; automate repeatable work |

## Generation 2 and Trusted Launch

The v6 and v7 series use a Generation 2 (UEFI) foundation and support [Trusted Launch](/azure/virtual-machines/trusted-launch) (Secure Boot and vTPM), which is the default security type for Generation 2 VMs. Most customers migrate from the v2 or v3 series, which are often Generation 1. Sources on the v4 or v5 series are frequently Generation 2 already, so confirm the generation of each source VM early.

**How to prepare:**

1. Confirm the VM generation early. If the source is Generation 1, plan a Generation 2 image or use the [in-place Generation 1 to Generation 2 upgrade](/azure/virtual-machines/generation-2).
1. With Secure Boot enabled, make sure OS drivers and any low-level agents (antivirus, backup, or monitoring components that use kernel or filter drivers) are current and signed.
1. Trusted Launch integrates with [Microsoft Defender for Cloud](/azure/defender-for-cloud/) guest attestation, which strengthens your security baseline.

> [!TIP]
> If a workload runs on a Generation 1 VM, confirm a supported path to Generation 2 before you migrate. This one-time step also raises your security baseline.

## NVMe storage interface

The v6 and v7 series present disks over [NVMe](/azure/virtual-machines/nvme-overview). Earlier families commonly used SCSI. This change has two practical implications:

- **Device paths change.** Linux references such as `/dev/disk/azure/scsi1/lunX` no longer apply.
- **Cross-generation moves are a redeploy, not an in-place resize.** Most v6 and v7 sizes are NVMe-only, and the disk controller type is fixed when a VM is created. The supported, predictable pattern is to redeploy from a Generation 2, NVMe-ready image, the same way image-based fleets already deploy.

**How to prepare:**

1. Confirm the OS image includes NVMe support. See [NVMe on Linux](/azure/virtual-machines/nvme-linux) and the [NVMe FAQ](/azure/virtual-machines/enable-nvme-faqs).
1. Replace hard-coded SCSI paths with stable identifiers, by-UUID references, filesystem labels, or Azure disk symlinks, in mount configurations, backup scripts, and database storage settings.
1. Plan cross-generation moves as redeploy-from-image rather than a portal resize.

### Device paths by generation

Disks keep their data, but the device names the OS sees change as a VM moves from SCSI to Azure Boost to a v6 size on NVMe.

| Disk | Pre-Boost (SCSI) | Azure Boost | v6 (NVMe) |
| --- | --- | --- | --- |
| OS disk | `/dev/sda` | `/dev/nvme0n1` | `/dev/nvme0n1` |
| Temp disk | `/dev/sdb` | `/dev/sda` | `/dev/nvme1n1` |
| Temp disk count | 1 | 1 | Up to 4 |
| First data disk | `/dev/sdc` | `/dev/nvme0n2` | `/dev/nvme0n2` |

A v6 size can present up to four local NVMe namespaces. If you stripe or pin scratch storage to local disk, account for the new count and naming. For more information, see [Local (temporary) disk behavior](#local-temporary-disk-behavior).

> [!WARNING]
> Any mount, script, or database setting that hard-codes an old device path points at the wrong device after migration. Update these references to stable identifiers before you migrate.

> [!NOTE]
> The disk controller type is fixed when a VM is created, so you can't resize a SCSI-based size directly to a remote-NVMe size. On Windows, you also can't resize between a size that has a temporary disk and one that doesn't. In both cases, redeploy from an image, or snapshot and rebuild, instead of resizing in place.

## Image prerequisites for Generation 2, NVMe, and MANA

The image is where Generation 2, NVMe, and MANA readiness actually live. A current marketplace image already includes all three. Custom images might need a one-time rebuild.

**How to prepare:**

1. Inventory custom images, and confirm Generation 2, NVMe, and MANA readiness.
1. Standardize on [Azure Image Builder](/azure/virtual-machines/image-builder-overview) and [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) with versioning, so you make the fix once and reuse it everywhere.
1. Test boot and disk discovery on one VM before a broad rollout.
1. After deployment, confirm the Azure VM Agent is present and healthy on the new VM. Then reprovision and verify the extensions and agents that don't carry forward on a fresh OS disk: boot diagnostics, the monitoring or Log Analytics agent, backup integration, and any security agents.

## MANA networking

The v6 and v7 series use the [MANA](/azure/virtual-network/accelerated-networking-mana-overview) adapter, part of Azure Boost. Accelerated networking is built in; the only requirement is a current OS and driver.

**Version floors:**

- **Linux:** MANA Ethernet drivers are upstream in Linux kernel 5.15 and later. Kernel 6.2 and later adds support for advanced features such as RDMA/InfiniBand and DPDK. Prefer a recent kernel from an [endorsed distribution](/azure/virtual-network/accelerated-networking-mana-linux).
- **Windows:** Current Windows Server images include the in-box driver. Keep updates applied.

**How to prepare:** Confirm OS and driver readiness, and capture a simple before-and-after network baseline during the pilot. MANA is also rolling out to existing sizes, so this check pays off across the estate.

## Persistent application data on the OS disk

Because a cross-generation move is a redeploy rather than an in-place resize, the new VM starts from a fresh OS disk built from the updated image. Any data or configuration that a workload writes directly to the OS disk, rather than to a managed data disk or an external store, doesn't carry over automatically. Some applications write state, license or activation files, configuration, or working data straight to the OS volume. Identify this content and migrate it to the new VMs after you provision them.

**How to prepare:**

1. Inventory what each workload persists to the OS disk (application configuration, license or activation files, local databases, certificates, and working or state data) versus what already lives on managed data disks or external services.
1. For ISV or custom applications, confirm with the vendor or development team where state is stored and whether a supported export/import or backup/restore path exists.
1. Add an explicit data-migration step to your runbook: capture the OS-disk data before cutover, and restore it to the new VM after deployment.
1. Keep a safety copy before you migrate. Snapshot or back up the source VM. For workloads that hold important data on the OS disk or rely on hard-coded device paths, also keep a bootable clone on a compatible older size that still uses the SCSI controller. Because the move changes the disk controller to NVMe and the device paths the OS sees, this clone gives you both a clean rollback and a running copy you can read the OS-disk data from while you copy it to the new VM.
1. Where practical, relocate persistent application data to managed data disks or external services, so future image refreshes don't require a data copy.
1. Validate the migrated data and application state in the pilot before you scale to later waves.

> [!WARNING]
> A redeploy starts from a fresh OS disk. Data that an application persists only to the OS disk, configuration, license files, local databases, or state, doesn't carry forward. Identify it up front and migrate it to the new VMs so nothing is lost.

## Local (temporary) disk behavior

A local (temporary) disk is present only on `d`-suffixed sizes (for example, Ddsv6 or Ddsv7) and is presented as NVMe. Sizes without the `d` have no local disk.

**How to prepare:**

1. If the workload uses local scratch storage (page file, SQL Server `tempdb`, or cache), choose a `d`-size or relocate that data to a managed data disk.
1. On Linux, the local NVMe disk isn't auto-formatted the way the old temporary disk was, and it's re-created raw on every stop/deallocate cycle. Plan a one-time format and mount at each start if you use it, and keep nothing persistent there.
1. Treat local-disk use as a sizing decision, not an afterthought.

## Resume hibernated VMs before you migrate

[Azure VM hibernation](/azure/virtual-machines/hibernate-resume) performs a suspend-to-disk: Azure stores the VM's memory contents on the OS disk and then deallocates the VM. It's common for virtual desktops and dev/test servers that don't run all day. Two states behave differently:

- A VM in the **hibernated state** holds a saved memory image. While it stays there, you can't resize it, attach or detach disks, or change NICs.
- A VM with hibernation only **enabled**, sitting in Running or Stop (deallocated), resizes normally.

A move to v6 or v7 is a redeploy or a disk-controller conversion. Neither path carries a saved memory state forward, so you must resume and clear the hibernated image first.

**How to prepare:**

1. **Resume, then migrate.** The working path is: resume (unhibernate) so the saved memory state is restored, let the workload settle, shut down to Stop (deallocated), then convert or redeploy and restart. A guest-OS shutdown alone isn't enough, the resize, disk, and NIC blocks clear only after the VM leaves the hibernated state and reaches Stop (deallocated).
1. **Treat the memory image as throwaway.** It doesn't transfer to the new VM, so confirm anything important is written to a disk or an external store before you resume and cut over.
1. **Re-validate hibernation support on the exact v6 or v7 target.** Support is gated by both VM size and a RAM ceiling, up to 64 GB on supported general-purpose series and up to 112 GB on supported GPU series. As of this writing, the documented hibernation-supported sizes are v5-series general-purpose families (Dasv5, Dadsv5, Dsv5, Ddsv5, Easv5, Eadsv5, Esv5, Edsv5) and the NVv4 and NVadsA10v5 GPU series, **no v6 or v7 sizes are listed yet.** Confirm the specific target on the [supported-sizes list](/azure/virtual-machines/hibernate-resume) before you rely on hibernation.
1. **Re-enable hibernation on the new VM.** Deallocate it, set `supportsHibernation=true` on the OS disk, run `az vm update --enable-hibernation true`, then reconfigure the guest OS. The OS disk must be large enough to hold the memory contents plus the OS, so a larger-memory size might need a larger OS disk. On Windows, the page file must be on the OS disk (C:), not the temporary disk.
1. **Check virtualization-based security (VBS).** If the workload uses Device Guard or Credential Guard, hibernation also requires Trusted Launch and Nested Virtualization. On Windows 11 24H2, that combination requires OS build 26100.3037 or later. Re-validate whenever the image or VM generation changes.
1. **Plan the resume step as a capacity risk.** Hibernated VMs carry no capacity guarantee on resume, and capacity reservations don't cover them. Bringing a VDI pool back at once depends on available capacity in the target region and zone. See [Region, zone, and capacity planning](#region-zone-and-capacity-planning).

> [!IMPORTANT]
> Resume a hibernated VM and reach Stop (deallocated) before you migrate. The saved session state stays on the old VM and is discarded, so save anything important first. Re-enable hibernation on the new size only after you confirm the size supports it and the OS disk can hold the new memory image.

## Region, zone, and capacity planning

Availability of the v6 and v7 series varies by [region and availability zone](/azure/reliability/availability-zones-overview). These families use distinct quota families and can have more limited capacity in some regions.

**How to prepare:**

1. Confirm the target size is available in the required regions and zones before you finalize the design.
1. Verify capacity for the target size in those regions and zones.
1. Request quota early to secure predictable supply of newer sizes.
1. Use [On-demand Capacity Reservations](/azure/virtual-machines/capacity-reservation-overview) to guarantee capacity.
1. If only regional (non-zonal) availability exists initially, confirm it meets the workload's resiliency requirements before you proceed.

> [!TIP]
> Request quota and reserve capacity early. Newer sizes can be more limited in some regions while Azure rolls out across all regions, and quota for these families is separate from earlier generations.

## Commercial continuity

[Reserved Instances](/azure/cost-management-billing/reservations/save-compute-costs-reservations?toc=%2Fazure%2Fvirtual-machines%2Ftoc.json) and [Savings Plans](/azure/cost-management-billing/savings-plan/) are scoped to a VM family, and the efficiency of the v7 series often means the same workload needs fewer or smaller vCPUs.

**How to prepare:**

1. Replan or [exchange reservations](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so discount coverage follows the workload to the new family.
1. Rightsize against observed usage rather than matching the old vCPU count one-for-one. This is where much of the price-performance benefit comes from.
1. Reapply Azure Hybrid Benefit and any bring-your-own-license configuration on the redeployed VM. Confirm it's set rather than assuming it transfers from the source.

## ISV network and storage appliances

Some workloads aren't just an application inside a VM, the VM *is* the appliance. Network virtual appliances (NVAs), Storage Appliances and others, certify specific VM families, NIC and driver configurations, and disk presentation. The move to MANA networking and an NVMe disk controller is exactly the kind of change they're sensitive to. Confirm support with the vendor before you migrate either one.

### Network virtual appliances

Firewalls, NVAs, and security or monitoring appliances certify specific VM families and NIC/driver configurations.

**How to prepare:**

1. Confirm the exact product, version, image source, licensing, and vendor support for the target family.
1. For NVAs, confirm NIC count, IP forwarding, accelerated networking, and HA/failover support on the family. Validate the data plane and failover before cutover.
1. For low-level agents, confirm a current signed version compatible with Secure Boot and NVMe.

### ISV storage appliances

Third-party storage and data-protection products often run as virtual appliances on Azure VMs (file, block, object, or backup). Because they sit directly on the VM's network and disk path, the MANA adapter and the SCSI-to-NVMe controller change can affect their drivers, throughput, and supportability, the same way they affect an NVA.

Managed and native services such as Azure NetApp Files, Azure Native Qumulo, and Azure Managed Lustre run on an Azure-managed substrate, so the platform handles MANA and Boost. The appliances that need real scrutiny are the VM-based products you run in your own subscription.

The following table lists common examples and what to confirm with each vendor. It's illustrative, not a support statement. Validate the exact product version, image, OS and driver, and target v6 or v7 size with the vendor.

| ISV | Offering | Type | What to confirm with the vendor |
| --- | --- | --- | --- |
| NetApp | Azure NetApp Files | Managed service | Platform-managed; no action needed for MANA or NVMe |
| NetApp | Cloud Volumes ONTAP | VM appliance | MANA and NVMe support for your version and target size |
| Dell | PowerScale / APEX File Storage | Managed (Dell and Microsoft) | Support on the target family; MANA and NVMe readiness |
| Pure Storage | Cloud Block Store | VM appliance | MANA and NVMe support for the target size |
| Cohesity | Data Platform | VM appliance | MANA and NVMe support; update to a supported version |
| Rubrik | Cloud Cluster | VM appliance | MANA and NVMe support; update to a supported version |
| WEKA | Data Platform | VM appliance (local NVMe) | NIC/driver support and local-NVMe namespace layout on the target size |
| Silk | Cloud Data Platform | VM appliance | Storage and NIC support with MANA and NVMe |
| Nasuni | Edge Appliance | VM appliance | Generation 2 image (version 10.0+); confirm NVMe support |
| Veeam, Commvault | Backup proxies / media agents | Customer VMs | Supported OS, driver, and VM family; throughput after the change |

**How to prepare:**

1. Treat VM-based storage appliances like NVAs: confirm the vendor supports the target size with the MANA adapter, an NVMe disk controller, and a Generation 2 image.
1. For scale-out file or cache products that use local NVMe, confirm the new local-disk count and NVMe namespace naming match what the product expects. See [Local (temporary) disk behavior](#local-temporary-disk-behavior).
1. For data-protection proxies and media agents, confirm the backup vendor supports the target family and that throughput holds after the NIC and disk-controller change.
1. Capture a storage and network baseline in the pilot, and validate failover before you scale to later waves.
1. Where a managed or native Azure storage service fits, prefer it. The platform handles MANA and Boost for you.

## Sequence and automate estate-scale migrations

For multi-tier applications, migration order matters. At estate scale, hundreds or thousands of VMs are a program rather than a series of manual resizes.

### Sequence by workload dependency

- Group migration waves by application or service dependency, so related tiers move together.
- Avoid splitting tightly coupled tiers across old and new families for extended periods.
- Keep waves small enough to validate quickly. This is about sequencing, not the size itself.

### Automate the repeatable work

1. Build inventory with [Azure Resource Graph](/azure/governance/resource-graph/overview), [Azure Migrate](/azure/migrate/migrate-services-overview), and tags.
1. Use infrastructure as code and pipelines for repeatable deployment, and use Azure Image Builder with Azure Compute Gallery for image rebuilds.
1. Steer new deployments with [Azure Policy](/azure/governance/policy/overview) allowed-SKU and image rules.
1. Roll out in rings (canary, pilot, production), and keep old pools and images until the new ones are validated.

## Architecture decision record template

Use this template to record the key decisions for each migration wave.

| Decision | Selected option | Rationale | Owner | Date |
| --- | --- | --- | --- | --- |
| Target VM family |  |  |  |  |
| Target region |  |  |  |  |
| Availability model | Regional / zonal |  |  |  |
| Deployment approach | Greenfield / redeploy-from-image / pool replacement |  |  |  |
| Image approach | Marketplace / rebuilt custom |  |  |  |
| Disk-path remediation | Required / not required |  |  |  |
| Local-disk (`d`-size) need | Yes / no |  |  |  |
| Availability assurance | Quota / capacity reservation |  |  |  |
| Commercial | Reservation or savings-plan replan |  |  |  |
| Rollback plan |  |  |  |  |

## Next steps

- [Migrate: the wave-based runbook](sizes-v6-v7-migration-migrate.md)
- [Trusted Launch for Azure VMs](/azure/virtual-machines/trusted-launch)
- [On-demand Capacity Reservations](/azure/virtual-machines/capacity-reservation-overview)
