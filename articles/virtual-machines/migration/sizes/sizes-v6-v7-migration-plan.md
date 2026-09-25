---
title: Plan a workload modernization to the v6 and v7 VM series
description: Considerations for modernizing Azure VM workloads to the v6 and v7 series, including Generation 2, NVMe storage, MANA networking, hibernation, capacity, and commercial planning.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 09/24/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Plan a workload modernization to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs ✔️ Flexible scale sets

**Workload patterns:** ✔️ B. Image-based hosts · E. Customer-managed VMs · F. Stateful and clustered · G. Certified appliances — ❌ Not for A, C, or D

The v6 and v7 Azure VM series are built on [Azure Boost](/azure/azure-boost/overview) and introduce a few platform changes compared with earlier generations: a Generation 2 (UEFI) foundation with Trusted Launch, NVMe-based storage, and the Microsoft Azure Network Adapter (MANA) for accelerated networking. Most workloads need only a one-time image refresh. Plan your modernization by screening each workload against the considerations in this article before you modernize.

Consider the following factors as you plan the modernization.

## Decide a modernization approach

### Option 1. Deploy in parallel and modernize (highly recommended)

The Microsoft recommended modernization approach is to deploy new v6 and v7 instances in parallel, then reinstall or migrate the application and data. A typical modernization wave is:

`Deploy new → Move workload → Validate → Retire old`

Use a controlled, wave-based approach. Confirm prerequisites, deploy from an updated image, validate the modernization, and then repeat at scale. Start with a small pilot to validate the pattern before expanding to additional workloads.

### Option 2. In-place upgrade

> [!IMPORTANT]
> You can't convert Generation 1 Dv2 and Dv3 VMs to Generation 2. For the full list of unsupported families, see [Trusted Launch supported VM size families](/azure/virtual-machines/trusted-launch#virtual-machines-sizes).

In some cases, you might need an in-place upgrade. This process aims to convert the VM from Generation 1 to Generation 2 and change the storage controller from SCSI to NVMe. Compared with redeployment, an in-place upgrade is more complex, has several prerequisites and steps, isn't fully compatible with all OSs and Server or VM configurations (these conditions are discussed in the following considerations in this article), requires extra careful validation, takes longer to complete, and introduces greater modernization risk. A typical modernization wave would be:

`Confirm prerequisites → Back up → Convert Gen 1 to Gen 2 → Switch SCSI to NVMe → Validate → Resume workload`

> [!IMPORTANT]
> The in-place conversion that switches an existing VM's controller from SCSI to NVMe has its own article, including setup, parameters, and how to revert. This approach has very specific requirements. Use it with caution and ensure you back up VMs. Ensure you read all the other considerations in this article. For more information, see [Convert a VM from SCSI to NVMe in place](scsi-to-nvme-migration.md).

## Image prerequisites for Generation 2, NVMe, and MANA

The image is where Generation 2, NVMe, and MANA readiness actually live. A current marketplace image already includes all three. Custom images might need a one-time rebuild.

**How to prepare:**

1. Inventory custom images, and confirm Generation 2, NVMe, and MANA readiness.
2. Standardize on [Azure Image Builder](/azure/virtual-machines/image-builder-overview) and [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) with versioning, so you make the fix once and reuse it everywhere.
3. Test boot and disk discovery on one VM before a broad rollout.
4. After deployment, confirm the Azure VM Agent is present and healthy on the new VM. Then reprovision and verify the extensions and agents that don't carry forward on a fresh OS disk: boot diagnostics, the monitoring or Log Analytics agent, backup integration, and any security agents.

## Generation 2 and Trusted Launch

The v6 and v7 series use a Generation 2 (UEFI) foundation and support [Trusted Launch](/azure/virtual-machines/trusted-launch) (Secure Boot and vTPM), which is the default security type for Generation 2 VMs. Most customers modernize from the v2 or v3 series, which are often Generation 1. Sources on the v4 or v5 series are frequently Generation 2 already, so confirm the generation of each source VM early.

Azure Virtual Machines supports upgrading Generation 1 virtual machines (VM) to Generation 2 by upgrading to the Trusted launch security type. Enabling Trusted Launch is required.

**How to prepare:**

1. Confirm the VM generation early. If the source is Generation 1, plan a Generation 2 image or use the [in-place Generation 1 to Generation 2 upgrade](/azure/virtual-machines/generation-2), for in-place conversion.
2. Read carefully the compatibility and prerequisites when Disk Encryption is configured.
3. Review the [Trusted Launch best practices](/azure/virtual-machines/trusted-launch-existing-vm-gen-1?tabs=windows%2Cportal#best-practices).
4. [Prepare the guest OS volume](/azure/virtual-machines/trusted-launch-existing-vm-gen-1?tabs=windows%2Cportal#update-guest-os-volume).
5. Review the [upgrade options and process for the Azure portal, PowerShell, the Azure CLI, and templates](/azure/virtual-machines/trusted-launch-existing-vm-gen-1?tabs=windows%2Cportal#upgrade-gen1-vm-to-trusted-launch).
6. With Secure Boot enabled, make sure OS drivers and any low-level agents (antivirus, backup, or monitoring components that use kernel or filter drivers) are current and signed.
7. Trusted Launch integrates with [Microsoft Defender for Cloud](/azure/defender-for-cloud/) guest attestation, which strengthens your security baseline.

## NVMe storage interface

The v6 and v7 series present disks over [NVMe](/azure/virtual-machines/nvme-overview). Earlier families commonly used SCSI. This change has two practical implications:

- **Device paths change.** Linux references such as `/dev/disk/azure/scsi1/lunX` no longer apply.
- **Cross-generation upgrade often requires redeploy, not a simple VM SKU change or resize.** Most v6 and v7 sizes are NVMe-only, and the disk controller type is fixed when a VM is created. The supported, predictable pattern is to redeploy from a Generation 2, NVMe-ready image, the same way image-based fleets already deploy.

> [!NOTE]
> The disk controller type is fixed when a VM is created, so you can't resize a SCSI-based size directly to a remote-NVMe size. On Windows, you also can't resize between a size that has a temporary disk and one that doesn't. In both cases, redeploy from an image, or snapshot and rebuild, instead of resizing in place.

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
> Any mount, script, or database setting that hard-codes an old device path points at the wrong device after modernization. Update these references to stable identifiers before you modernize.

**How to prepare:**

1. Generation 2 VM is required for NVMe support.
2. Confirm the OS image includes NVMe support. See [NVMe on Linux](/azure/virtual-machines/nvme-linux) and the [NVMe FAQ](/azure/virtual-machines/enable-nvme-faqs).
3. Plan the replacement of hard-coded SCSI paths with stable identifiers, by-UUID references, filesystem labels, or Azure disk symlinks, in mount configurations, backup scripts, and database storage settings.
4. Plan cross-generation moves as redeploy-from-image rather than a portal resize.
5. Understand the considerations around the [Local Temp disk](/azure/virtual-machines/enable-nvme-temp-faqs#what-changes-should-i-prepare-for-when-configuring-my-vms-with-temp-nvme-disks-) and [Resize details](/azure/virtual-machines/azure-vms-no-temp-disk#can-i-resize-a-vm-size-that-has-a-local-temp-disk-to-a-vm-size-with-no-local-temp-disk---)

## Local (temporary) disk behavior

A local (temporary) disk is present only on `d`-suffixed sizes (for example, Ddsv6 or Ddsv7) and is presented as NVMe. Sizes without the `d` have no local disk.

**How to prepare:**

1. If the workload uses local scratch storage (OS page file, SQL Server `tempdb`, or cache), choose a `d`-size or relocate that data to a managed data disk.
2. On Linux, the local NVMe disk isn't auto-formatted the way the old temporary disk was, and it's re-created raw on every stop/deallocate cycle. Plan a one-time format and mount at each start if you use it, and keep nothing persistent there.
3. Treat local-disk use as a sizing decision, not an afterthought.

## Persistent application data on the OS disk

**Applies to:** ✔️ E. Customer-managed VMs ✔️ F. Stateful and clustered ✔️ G. Certified appliances — pool-replaced and service-managed nodes hold no per-node state by design.

A cross-generation move is usually a redeploy rather than an in-place resize, the new VM starts from a fresh OS disk built from the updated image. Any data or configuration that a workload writes directly to the OS disk, rather than to a managed data disk or an external store, doesn't carry over automatically. Some applications write state, license or activation files, configuration, or working data straight to the OS volume. Identify this content and migrate it to the new VMs after you provision them.

**How to prepare:**

1. Inventory what each workload persists to the OS disk (application configuration, license or activation files, local databases, certificates, and working or state data) versus what already lives on managed data disks or external services.
2. For ISV or custom applications, confirm with the vendor or development team where state is stored and whether a supported export/import or backup/restore path exists.
3. Add an explicit data-migration step to your runbook: capture the OS-disk data before cutover, and restore it to the new VM after deployment.
4. Keep a safety copy before you modernize. Snapshot or back up the source VM. For workloads that hold important data on the OS disk or rely on hard-coded device paths, also keep a bootable clone on a compatible older size that still uses the SCSI controller. Because the move changes the disk controller to NVMe and the device paths the OS sees, this clone gives you both a clean rollback and a running copy you can read the OS-disk data from while you copy it to the new VM.
5. Where practical, relocate persistent application data to managed data disks or external services, so future image refreshes don't require a data copy.
6. Validate the migrated data and application state in the pilot before you scale to later waves.

> [!WARNING]
> A redeploy starts from a fresh OS disk. Data that an application persists only to the OS disk, configuration, license files, local databases, or state, doesn't carry forward. Identify it up front and migrate it to the new VMs so nothing is lost.

### Azure Disk Encryption and encryption at host

Azure Disk Encryption (ADE) encrypts volumes inside the guest — BitLocker on Windows, dm-crypt on Linux — with keys held in Key Vault. It's not available on the v6 or v7 series for either operating system, and it's scheduled for retirement on 15 September 2028. After that date, ADE-enabled workloads keep running, but encrypted disks fail to unlock after a reboot.

The replacement is [encryption at host](/azure/virtual-machines/disk-encryption), which encrypts on the host before data reaches storage. It covers the temporary disk, the ephemeral OS disk, and the OS and data disk caches, supports customer-managed keys through a disk encryption set, and doesn't consume VM CPU.

> [!WARNING]
> An ADE-encrypted VM can't reach v6 or v7 by conversion. Encryption at host can't be enabled on a VM that currently has, or has ever had, ADE enabled, and disks previously encrypted with ADE can't use customer-managed keys. Decrypting doesn't clear it: the Unified Data Encryption (UDE) flag survives decryption, snapshots, and disk copies made with the Copy option. On Gen1 VMs, an encrypted Windows OS volume also blocks the [Gen1 to Trusted Launch upgrade](/azure/virtual-machines/trusted-launch-existing-vm-gen-1).

**What the move requires:**

- **New disks and a new VM.** There's no in-place conversion. Create the target disk by using the upload method and copy the VHD blob. Add a 512-byte offset because Azure omits the footer when it reports disk size.
- **Windows:** disable ADE first, then confirm `manage-bde -status` reports every volume fully decrypted before you copy. ADE on Windows encrypts either the OS disk alone or the OS plus data disks; there's no data-only pattern.
- **Linux with an encrypted OS disk:** ADE can't be disabled. Build a new VM from a current image and migrate data at the application level. There's no disk-copy path.
- **Domain-joined VMs:** remove the VM from the domain before deleting the original, and rejoin afterward. The replacement has a different computer SID, which affects anything bound to machine identity.
- **Downtime.** The disk copy and VM recreation can't be done online.

**How to prepare:**

1. Inventory which VMs have ADE enabled, and whether the OS disk, the data disks, or both are encrypted.
2. Choose the target encryption model: encryption at host with customer-managed keys for most workloads, or [Confidential VM sizes with OS disk encryption](/azure/confidential-computing/confidential-vm-overview#confidential-os-disk-encryption) where the control objective requires the platform not to handle plaintext.
3. Confirm the target size supports encryption at host. There's no static list — retrieve the supported sizes programmatically.
4. Fold the rebuild into the modernization wave rather than running it as a separate project. It's the same work, and it's required before the 2028 retirement regardless.
5. After cutover, verify `securityProfile.encryptionAtHost` on the new VM, then update Key Vault access policies to disable the disk-encryption setting once nothing depends on it.

For the full procedure, see [Migrate from Azure Disk Encryption to encryption at host](/azure/virtual-machines/disk-encryption-migrate).

## MANA networking

The v6 and v7 series use the [MANA](/azure/virtual-network/accelerated-networking-mana-overview) adapter, which is part of Azure Boost. This adapter includes accelerated networking. The only requirement is a current OS and driver. MANA is also rolling out to existing sizes, so this check pays off across the estate.

**Version floors:**

- **Linux:** MANA Ethernet drivers are upstream in Linux kernel 5.15 and later. Kernel 6.2 and later adds support for advanced features such as RDMA/InfiniBand and DPDK. Use a recent kernel from an endorsed distribution. For more information, see [Linux VMs with the Microsoft Azure Network Adapter](/azure/virtual-network/accelerated-networking-mana-linux).
- **Windows:** Current Windows Server images include the built-in driver. Keep updates applied. For more information, see [Windows VMs with the Microsoft Azure Network Adapter](/azure/virtual-network/accelerated-networking-mana-windows).

**How to prepare:**

1. Confirm OS and driver readiness.
2. Plan driver installation and validation.
3. Capture a simple before-and-after network baseline during the pilot.

## Resume hibernated VMs before you migrate

**Applies to:** ✔️ E. Customer-managed VMs, including Azure Virtual Desktop personal host pools — pooled session hosts and service-managed compute don't hibernate.

[Azure VM hibernation](/azure/virtual-machines/hibernate-resume) performs a suspend-to-disk: Azure stores the VM's memory contents on the OS disk and then deallocates the VM. It's common for virtual desktops and dev/test servers that don't run all day. Two states behave differently:

- A VM in the **hibernated state** holds a saved memory image. While it stays there, you can't resize it, attach or detach disks, or change NICs.
- A VM with hibernation only **enabled**, sitting in Running or Stop (deallocated), resizes normally.

A move to v6 or v7 is a redeploy or a VM Generation and disk-controller conversion. Neither path carries a saved memory state forward, so you must resume and clear the hibernated image first.

**How to prepare:**

1. **Resume, then modernize.** The working path is: resume (unhibernate) so the saved memory state is restored, let the workload settle, shut down to Stop (deallocated), then convert or redeploy and restart. A guest-OS shutdown alone isn't enough, the resize, disk, and NIC blocks clear only after the VM leaves the hibernated state and reaches Stop (deallocated).
2. **Treat the memory image as throwaway.** It doesn't transfer to the new VM, so confirm anything important is written to a disk or an external store before you resume and cut over.
3. **Re-validate hibernation support on the exact v6 or v7 target.** Support is gated by both VM size and a RAM ceiling, up to 64 GB on supported general-purpose series and up to 112 GB on supported GPU series. As of this writing, the documented hibernation-supported sizes are v5-series general-purpose families (Dasv5, Dadsv5, Dsv5, Ddsv5, Easv5, Eadsv5, Esv5, Edsv5) and the NVv4 and NVadsA10v5 GPU series, **no v6 or v7 sizes are listed yet.** Confirm the specific target on the [supported-sizes list](/azure/virtual-machines/hibernate-resume) before you rely on hibernation.
4. **Re-enable hibernation on the new VM.** Deallocate it, set `supportsHibernation=true` on the OS disk, run `az vm update --enable-hibernation true`, then reconfigure the guest OS. The OS disk must be large enough to hold the memory contents plus the OS, so a larger-memory size might need a larger OS disk. On Windows, the page file must be on the OS disk (C:), not the temporary disk.
5. **Check virtualization-based security (VBS).** If the workload uses Device Guard or Credential Guard, hibernation also requires Trusted Launch and Nested Virtualization. On Windows 11 24H2, that combination requires OS build 26100.3037 or later. Re-validate whenever the image or VM generation changes.
6. **Plan the resume step as a capacity risk.** Hibernated VMs carry no capacity guarantee on resume, and capacity reservations don't cover them. Bringing a VDI pool back at once depends on available capacity in the target region and zone. See [Region, zone, and capacity planning](#region-zone-and-capacity-planning).

> [!IMPORTANT]
> Resume a hibernated VM and reach Stop (deallocated) before you modernize. The saved session state stays on the old VM and is discarded, so save anything important first. Re-enable hibernation on the new size only after you confirm the size supports it and the OS disk can hold the new memory image.

## Region, zone, and capacity planning

Availability of the v6 and v7 series varies by [region and availability zone](/azure/reliability/availability-zones-overview). These families use distinct quota families and can have more limited capacity in some regions.

**How to prepare:**

1. Confirm the target size is available in the required regions and zones before you finalize the design.
2. Verify capacity for the target size in those regions and zones.
3. Request quota early to secure predictable supply of newer sizes.
4. Use [On-demand Capacity Reservations](/azure/virtual-machines/capacity-reservation-overview) to guarantee capacity.
5. If only regional (non-zonal) availability exists initially, confirm it meets the workload's resiliency requirements before you proceed.

> [!TIP]
> Request quota and reserve capacity early. Newer sizes can be more limited in some regions while Azure rolls out across all regions, and quota for these families is separate from earlier generations.

## Placement for clustered workloads

For availability groups, failover clusters, and other quorum-based workloads, modernization temporarily changes your placement. You run old and new nodes side by side, and the new nodes land wherever the target family has capacity — which might not be the zones the cluster was designed around.

**How to prepare:**

1. Map the current zone layout of every node or replica before you start.
2. Confirm the target size is available in *each* zone the cluster spans, not just in the region. If it isn't, plan the wave so the cluster doesn't end up with all replicas in one zone.
3. Account for the extra nodes in quota. A side-by-side modernization needs headroom for both the old and new set simultaneously.
4. Use [capacity reservations](/azure/virtual-machines/capacity-reservation-overview) for the zones you must land in, so a wave doesn't stall midway with the cluster in a degraded layout.

## Commercial continuity

[Reserved Instances](/azure/cost-management-billing/reservations/save-compute-costs-reservations?toc=%2Fazure%2Fvirtual-machines%2Ftoc.json) and [Savings Plans](/azure/cost-management-billing/savings-plan/) are scoped to a VM family, and the efficiency of the v7 series often means the same workload needs fewer or smaller vCPUs.

**How to prepare:**

1. Replan or [exchange reservations](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so discount coverage follows the workload to the new family.
2. Rightsize against observed usage rather than matching the old vCPU count one-for-one. This is where much of the price-performance benefit comes from.
3. Reapply Azure Hybrid Benefit and any bring-your-own-license configuration on the redeployed VM. Confirm it's set rather than assuming it transfers from the source.

## Retiring v3 workloads

The Dv3, Dsv3, Ev3, and Esv3 series retire on November 15, 2029. After that date, you can't create, resize into, run, or purchase these sizes, and you can't fall back to a v3 source VM.

**Planning milestones:**

1. Inventory every VM, scale set, and reservation on the four v3 series.
2. Set a program completion date well before November 15, 2029. Leave time for a final wave and its rollback window.
3. Schedule v3 workloads in early waves, and track them as a separate milestone.
4. Plan commercial changes. One-year and three-year reservations for these series can't be purchased or renewed after July 1, 2026.

**Replacement size selection:**

| Source series | v5 (smoothest transition) | v6 (Current) | v7 (Current) |
| --- | --- | --- | --- |
| Dv3, Dsv3 | Dv5, Dsv5, Ddv5, Ddsv5, Dasv5, Dadsv5 | Dsv6, Ddsv6, Dasv6, Dadsv6 | Dsv7, Ddsv7, Dasv7, Dadsv7 |
| Ev3, Esv3 | Ev5, Esv5, Edv5, Edsv5, Easv5, Eadsv5 | Esv6, Edsv6, Easv6, Eadsv6 | Esv7, Edsv7, Easv7, Eadsv7 |

Every v3 size includes a local temporary disk. If the workload uses it, choose a `d`-suffixed target size. Size against observed usage rather than the v3 vCPU count. For the v5 path, see [Modernize to the v5 VM series](../../sizes/lifecycle/sizes-v5-modernization-overview.md). For retirement details, see the [Retired VM sizes modernization guide](../../sizes/lifecycle/retirement/retired-sizes-modernization-guide.md).

## Wave sequencing

For mult-tier applications, modernization order matters. At estate scale, hundreds or thousands of VMs are a program rather than a series of manual resizes. The sequence is a planning decision you make once rather than a judgment call per wave.

**How to prepare:**

1. Group waves by application or service dependency, so related tiers move together.
2. Avoid splitting tightly coupled tiers across old and new families for extended periods.
3. Sequence non-production before production for every application.
4. Keep waves small enough to validate inside one window. This guidance is about sequencing, not about the size itself.

Record the resulting wave list with owners and target windows. For how to execute waves, see [4. Modernize in waves](sizes-v6-v7-migration-migrate.md#phase-2-modernize-in-waves).

## ISV virtual appliances

**Applies to:** ✔️ G. Certified appliances — vendor certification is a gate, not a validation item. Confirm it before you plan anything else.

Some workloads aren't just an application inside a VM, the VM *is* the appliance. Network virtual appliances (NVAs), Storage Appliances and others, certify specific VM families, NIC and driver configurations, and disk presentation. The move to MANA networking and an NVMe disk controller is exactly the kind of change they're sensitive to. Confirm support with the vendor before you modernize either one.

### Network virtual appliances

Firewalls, NVAs, and security or monitoring appliances certify specific VM families and NIC/driver configurations.

**How to prepare:**

1. Confirm the exact product, version, image source, licensing, and vendor support for the target family.
1. For NVAs, confirm NIC count, IP forwarding, accelerated networking, and HA/failover support on the family. Validate the data plane and failover before cutover.
1. For low-level agents, confirm a current signed version compatible with Secure Boot and NVMe.

### Storage and backup virtual appliances

Third-party storage and data-protection products often run as virtual appliances on Azure VMs (file, block, object, or backup). Because they sit directly on the VM's network and disk path, the MANA adapter and the SCSI-to-NVMe controller change can affect their drivers, throughput, and supportability, the same way they affect an NVA.

Managed and native services such as Azure NetApp Files, Azure Native Qumulo, and Azure Managed Lustre run on an Azure-managed substrate, so the platform handles MANA and Boost. The appliances that need real scrutiny are the VM-based products you run in your own subscription.

The following table lists common examples and what to confirm with each vendor. It's illustrative, not a support statement. **Validate the exact product version, image, OS and driver, and target v6 or v7 size with the vendor.**

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

### Confirm before you commit

Treat these items as gates, not validation items. If the vendor doesn't support the target family, there's no modernization to plan.

| Confirm with the vendor | Why it gates the decision |
| --- | --- |
| Support for the exact VM family and size | Certification is granted per family, not per generation. |
| Generation 2, Trusted Launch, NVMe, and MANA support | The appliance image and its drivers must handle all four. |
| Required NIC count and accelerated networking | NVAs often need a NIC layout that not every size offers. |
| Licensing and bootstrap on the new size | Licenses are sometimes bound to instance attributes. |
| High-availability topology on the target family | The pair must be supported as a pair, not just as single instances. |

For the execution sequence once certification is confirmed, see [Cut over a certified appliance](sizes-v6-v7-migration-migrate.md#cut-over-a-certified-appliance-g).

## Exit criteria

- Every required planning decision has a selection, a rationale, and an owner.
- The target family, region, zone, and availability model are recorded.
- Quota is confirmed, or a capacity reservation is in place for the zones you must land in.
- The image approach is chosen, and any custom-image rebuild is assigned.
- Disk-path and OS-disk data remediation is scoped and owned.
- Commercial coverage is replanned so the discount follows the workload.
- Appliance workloads have written vendor confirmation for the exact target family.
- The wave list exists, with the pilot identified and workload owners informed.

## Next steps

- [4. Modernize: the wave-based runbook](sizes-v6-v7-migration-migrate.md)
