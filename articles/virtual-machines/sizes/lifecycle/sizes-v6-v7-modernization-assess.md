---
title: Assess readiness for the v6 and v7 VM series
description: Qualify candidate workloads for the v6 and v7 series and confirm a clean modernization path, boot mode, image, storage, networking, and capacity.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/25/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Assess readiness for the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

**Workload patterns:** ✔️ B. Image-based hosts · E. Customer-managed VMs · F. Stateful and clustered · G. Certified appliances — ❌ Not for A, C, or D

To move to the v6 or v7 series, start with a quick readiness check. Readiness is a short checklist focused on the platform underneath: boot mode, image, storage interface, network driver, and regional and zonal availability.

> [!NOTE]
> **Who this article is for.** This readiness check assumes you own the image and the VM — customer-managed applications, stateful and clustered workloads, session-host golden images, and certified appliances.

## Greenfield deployments

For new deployments, you don't need to fix anything. Choose the v6 or v7 series intentionally and start clean:

- Confirm [region and zone availability](/azure/reliability/availability-zones-overview) and quota for the chosen family.
- Note a fallback size, zone, or region before launch.
- Use infrastructure as code ([Bicep](/azure/azure-resource-manager/bicep/overview) or Terraform) for repeatability.
- Enable [Trusted Launch](/azure/virtual-machines/trusted-launch) (the default for Generation 2) for Secure Boot and vTPM.
- Deploy from a current Generation 2, NVMe-ready, MANA-ready marketplace or [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) image.

Greenfield deployments can skip the remediation guidance that follows and go straight to sizing and capacity in [Plan the modernization](sizes-v6-v7-modernization-plan.md).

## Brownfield modernization from the v2 through v5 series

### Retiring v3 source series

> [!IMPORTANT]
> The Dv3, Dsv3, Ev3, and Esv3 series retire on November 15, 2029. After that date, you can't create, resize into, run, or purchase these sizes. Assess workloads on these series first. For retirement details, see the [Retired VM sizes modernization guide](./retirement/retired-sizes-modernization-guide.md).

Workloads on these series usually trigger several readiness signals at once. Check these signals for every v3 workload:

- **Generation 1:** Many v3 VMs are Generation 1. Plan a Generation 2 image. You can't convert Generation 1 Dv3 VMs to Generation 2 in place. For more information, see [Decide a modernization approach](sizes-v6-v7-modernization-plan.md#decide-a-modernization-approach).
- **Image:** Refresh custom images to a current Generation 2, NVMe-ready, and MANA-ready version.
- **NVMe:** The v3 series presents disks over SCSI. Replace hard-coded SCSI device paths with stable identifiers.
- **MANA:** Confirm the OS and driver meet the MANA version floors.
- **Local disk:** Every v3 size includes a local temporary disk. If the workload uses it, choose a `d`-suffixed target size, such as Ddsv6 or Edsv6, or relocate that data to a managed data disk.
- **Region and quota:** [Capacity growth restrictions](./retirements-and-capacity-restrictions.md) already block extra v3 quota. Request quota for the target family early in every region and zone you need.

If the v6 or v7 prerequisites block a v3 workload, the v5 series provides the smoothest transition. For more information, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md).

### What changes, and what doesn't


| What changes | What doesn't change |
| --- | --- |
| Boot mode (Generation 2, UEFI, Trusted Launch) | Application code and configuration |
| Disk interface (NVMe) and disk device paths | Business logic, user flows, APIs |
| Network driver (MANA) | Autoscale rules and scaling thresholds |
| Local and temporary disk presence and format | Load balancer, ingress, and DNS topology |
| Image driver readiness | Identity and authentication design |
| Low-level agents that install kernel or filter drivers | Application-layer monitoring dashboards |


### Hard gates

Two categories of workloads have a gate that comes before this checklist. Confirm the gate first. If the target family isn't supported, the remaining signals don't matter.

> [!IMPORTANT]
> - **SAP workloads.** Only sizes on the SAP-certified list are supported, regardless of platform readiness. Treat certification as a gate that comes before all other planning, not as a validation item. See [What SAP software is supported on Azure VMs](/azure/sap/workloads/supported-product-on-azure).
> - **ISV virtual appliances.** Firewalls, network virtual appliances, backup dec, and storage appliances are vendor-certified products. The vendor certifies specific VM families, NIC layouts, driver configurations, and disk presentation. Confirm the vendor supports your exact target family before assessing anything else. See [ISV network and storage appliances](sizes-v6-v7-modernization-plan.md#isv-virtual-appliances).

### Readiness signals to check


| Signal | Why it matters | Recommended action |
| --- | --- | --- |
| Generation 1 source (common on v2, v3, and v4) | The v6 and v7 series require Generation 2 (UEFI). | Plan a Generation 2 image, or use the [Generation 1 to Generation 2 upgrade](/azure/virtual-machines/generation-2). |
| OS NVMe support | The OS must discover disks over [NVMe](/azure/virtual-machines/nvme-overview). | Confirm a supported OS version and refresh the image if needed, or follow the [SCSI to NVMe conversion](../../migration/scsi-to-nvme-migration.md). |
| Custom image not NVMe/Generation 2 ready | The image might not boot or attach storage on the target size. | Rebuild or update the image; test boot and disk discovery once. |
| MANA driver readiness | Networking uses the [MANA](/azure/virtual-network/accelerated-networking-mana-overview) adapter. | Confirm OS and driver support (see version floors under [Assessment Checklist](#assessment-checklist). |
| Hard-coded SCSI disk paths | Scripts or agents that reference `/dev/disk/azure/scsi*` need stable IDs. | Switch to stable identifiers (by-UUID or Azure disk symlinks). |
| Persistent data on the OS disk | Cross-generation moves redeploy from a fresh image, so anything an app writes straight to the OS volume (configuration, license/activation files, local databases, certificates, or state) doesn't carry over. | Inventory what each workload persists to the OS disk and plan a capture-and-restore step, or relocate it to a managed data disk or external store. See [Persistent application data on the OS disk](sizes-v6-v7-modernization-plan.md#persistent-application-data-on-the-os-disk). |
| Temporary-disk dependency | A local or temporary disk exists only on `d`-suffixed sizes and is presented as NVMe. A source VM that already has a temporary disk (for example, `E32ds_v5`) can't convert in place directly to a v6 size. | Choose a `d`-size if local scratch is needed, and relocate the page file or `tempdb` accordingly. Where the source has a temporary disk, plan a redeploy path rather than a direct conversion. |
| Azure Disk Encryption (ADE) for Linux | ADE for Linux isn't supported with NVMe, so converting an ADE-encrypted Linux VM to NVMe produces an unsupported configuration. | Decrypt the VM before you convert, or use the redeploy-from-image path. Check for ADE yourself first, the community SCSI-to-NVMe conversion script doesn't currently detect ADE and can complete without warning. |
| Local NVMe temporary disk isn't auto-formatted | On v6 and v7 sizes, the local NVMe temporary disk is presented raw and unformatted and is re-created on every stop/deallocate cycle. Nothing left on it is persistent, and on Windows the `D:` mapping can be lost after a deallocate/start cycle. | Initialize and format the local disk on each start (for example, a boot-time task), and keep nothing persistent there. |
| Region and zone requirement | Availability varies by region and zone. | Confirm the target size is available in the required regions and zones before committing the design. See [Region, zone, and capacity planning](sizes-v6-v7-modernization-plan.md#region-zone-and-capacity-planning). |
| Availability and quota for the new family | The v6 and v7 series use distinct quota families. | Request quota early, verify capacity for the target size in the required regions and zones, and use [capacity reservations](/azure/virtual-machines/capacity-reservation-overview) for guaranteed supply. |
| Family-scoped reservations and savings plans | Discounts are scoped to a VM family. | Replan [reservations or savings plans](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) so coverage follows the move. |
| Low-level ISV agents (antivirus, backup, monitoring with kernel or filter drivers) | Secure Boot and NVMe can affect unsigned or older drivers. | Confirm a current, signed agent version. |
| Marketplace or ISV network appliance | Vendors certify specific VM families and NIC/driver configurations. | Confirm vendor support for the target family and image. |

#### Session-host golden images

For **non-persistent** session hosts — Azure Virtual Desktop pooled host pools, Citrix DaaS random (pooled) catalogs, and Omnissa Horizon floating instant-clone pools (pattern B) — the readiness signals above apply to the **golden image**, not to each session host. Assess the image once, and pay particular attention to two signals that bite hardest on session hosts:

- **Low-level agents.** Antivirus, EDR, and monitoring agents that install kernel or filter drivers must be current and signed to work with Secure Boot.
- **Persistent data on the OS disk.** This pattern assumes user profiles and application data already live outside the session host. Confirm that's true before you rebuild the image.

**Persistent desktops don't follow this pattern.** Azure Virtual Desktop personal host pools, Citrix DaaS static (dedicated) catalogs, and Omnissa Horizon dedicated assignments bind a user to a specific VM, so there's no shared image to assess once and no host to drain. Assess each one as a customer-managed VM, and expect the OS-disk data and hibernation signals in the preceding table to apply.

For the host replacement sequence, see [Image-based desktop and application hosts](sizes-v6-v7-modernization-discover.md#b-image-based-desktop-and-application-hosts).


## Assessment checklist

Keep the checklist lean and platform-focused.

**Scope**

- Confirm sponsor and workload owner.
- Document business driver (capacity, performance, cost, or modernization).
- Identify source family (v2/v3/v4/v5) and deployment type (modernization or greenfield).
- Separate production and non-production.
- Agree on modernization window and success/rollback criteria.

**Compute and capacity**

- Select current size and target v6/v7 size.
- Confirm VM generation (Generation 2 path planned if the source is Generation 1).
- Confirm target region and zone requirement.
- Confirm quota and availability per wave; consider [capacity reservation](/azure/virtual-machines/capacity-reservation-overview).
- Note fallback size or region.

**OS and image**

- Support OS version; image is Generation 2, NVMe, and MANA ready.
- Azure Disk Encryption, BitLocker, or similar.
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
> - **Windows:** Use current Windows Server images with the built-in MANA driver and the latest updates.
> - MANA is also rolling out to some existing VM series, so confirming OS compatibility benefits the whole estate, not just the v6 and v7 series.

**Commercial**

- Replan family-scoped reservations and savings plans so discount coverage follows the workload.

### Inventory query (Azure Resource Graph)

The following Azure Resource Graph query surfaces the items that gate a clean move: size, generation, OS, image source, disk controller type, and more. Run this query in the portal's [Resource Graph Explorer](/azure/governance/resource-graph/overview) or via `az graph query`.

For more queries, see [Starter queries](/azure/governance/resource-graph/samples/starter).

```kusto
resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend p = properties
// ---- identity & placement ----
| extend
    resourceId          = id,
    availabilityZone    = tostring(zones[0]),
    availabilitySetId   = tostring(p.availabilitySet.id),
    vmssId              = tostring(p.virtualMachineScaleSet.id),
    platformFaultDomain = tostring(p.platformFaultDomain)
| extend
    availabilitySetName = iff(isnotempty(availabilitySetId), tostring(split(availabilitySetId, '/')[-1]), ''),
    vmssName            = iff(isnotempty(vmssId),            tostring(split(vmssId, '/')[-1]), '')
| extend
    membershipModel = case(
        isnotempty(vmssId),            'VMSS Flex',
        isnotempty(availabilitySetId), 'Availability Set',
                                       'Standalone'),
    zonePlacement   = iff(isnotempty(availabilityZone), 'Zonal', 'Regional')
// ---- compute & generation ----
| extend
    vmSize            = tostring(p.hardwareProfile.vmSize),
    hyperVGeneration  = tostring(p.extended.instanceView.hyperVGeneration)
// ---- power state + data quality ----
| extend
    vmState              = tostring(p.extended.instanceView.powerState.displayStatus)
| extend
    instanceViewCaptured = iff(isempty(hyperVGeneration) and isempty(vmState), 'No', 'Yes')
// ---- OS & image ----
| extend
    osType            = tostring(p.storageProfile.osDisk.osType),
    osVersion         = tostring(p.extended.instanceView.osVersion),
    computerName      = tostring(p.extended.instanceView.computerName),
    isMarketplaceVm   = isnotempty(plan),
    imagePublisher    = tostring(p.storageProfile.imageReference.publisher),
    imageOffer        = tostring(p.storageProfile.imageReference.offer),
    imageSku          = tostring(p.storageProfile.imageReference.sku),
    imageVersion      = tostring(p.storageProfile.imageReference.exactVersion)
// ---- storage / disks ----
| extend
    diskControllerType= tostring(p.storageProfile.diskControllerType),
    osDiskType        = tostring(p.storageProfile.osDisk.managedDisk.storageAccountType),
    osDiskSizeGB      = toint(p.storageProfile.osDisk.diskSizeGB),
    ephemeralOsDisk   = iff(tostring(p.storageProfile.osDisk.diffDiskSettings.option) =~ 'Local', 'Yes', 'No'),
    dataDiskCount     = array_length(p.storageProfile.dataDisks)
// ---- capabilities / licensing ----
| extend
    hibernationEnabled= tostring(p.additionalCapabilities.hibernationEnabled),
    ultraSsdEnabled   = tostring(p.additionalCapabilities.ultraSSDEnabled),
    licenseType       = tostring(p.licenseType)
// ---- network (VM level) ----
| extend
    nicCount          = array_length(p.networkProfile.networkInterfaces)
// ---- security & encryption ----
| extend
    securityType      = tostring(p.securityProfile.securityType),
    secureBootEnabled = tostring(p.securityProfile.uefiSettings.secureBootEnabled),
    vTpmEnabled       = tostring(p.securityProfile.uefiSettings.vTpmEnabled),
    adeOsDiskEnabled  = tostring(p.storageProfile.osDisk.encryptionSettings.enabled)
// ---- NIC flags: any NIC on the VM ----
| extend joinKey = tolower(tostring(id))
| join kind=leftouter (
    resources
    | where type =~ 'microsoft.network/networkinterfaces'
    | extend vmId = tolower(tostring(properties.virtualMachine.id))
    | where isnotempty(vmId)
    | summarize
        anyAccelNet = countif(tobool(properties.enableAcceleratedNetworking) == true),
        anyIpFwd    = countif(tobool(properties.enableIPForwarding) == true)
      by vmId
) on $left.joinKey == $right.vmId
| extend
    acceleratedNetworking = iff(coalesce(anyAccelNet, 0) > 0, 'Yes', 'No'),
    ipForwarding          = iff(coalesce(anyIpFwd, 0) > 0, 'Yes', 'No')
| project
    name, resourceGroup, location,
    membershipModel, zonePlacement, availabilityZone, vmssName, availabilitySetName, platformFaultDomain,
    vmSize, hyperVGeneration,
    vmState, instanceViewCaptured,
    osType, osVersion, computerName, isMarketplaceVm,
    imagePublisher, imageOffer, imageSku, imageVersion,
    diskControllerType, osDiskType, osDiskSizeGB, ephemeralOsDisk, dataDiskCount,
    hibernationEnabled, ultraSsdEnabled, licenseType,
    nicCount, acceleratedNetworking, ipForwarding,
    securityType, secureBootEnabled, vTpmEnabled, adeOsDiskEnabled,
    subscriptionId, resourceId, vmssId, availabilitySetId
| order by membershipModel asc, vmSize asc, name asc
```

| Column | Values | Meaning / why it matters |
|---|---|---|
| `name` / `resourceGroup` / `location` | string | VM name, resource group, Azure region (target availability is per-region). |
| `membershipModel` | `Standalone` · `Availability Set` · `VMSS Flex` | How the VM is grouped for fault tolerance; shapes redeploy and sequencing. |
| `zonePlacement` | `Zonal` · `Regional` | Zone posture (AvSet VMs are always `Regional`). |
| `availabilityZone` | `1`/`2`/`3` or blank | Blank = regional or in an availability set. |
| `vmssName` / `availabilitySetName` | string or blank | Parent VMSS Flex or availability set, if any. |
| `platformFaultDomain` | integer or blank | Fault-domain index for Flex and AvSet members. |
| `vmSize` | e.g. `Standard_D4s_v3` | The VM size; decode CPU platform from it (see CPU table). |
| `hyperVGeneration` | `V1` · `V2` · blank | `V1` = cross-gen redeploy needed (biggest effort driver). Blank = state not captured. |
| `vmState` | `VM running` · `VM deallocated` · `VM stopped` · … | Current power state. |
| `instanceViewCaptured` | `Yes` · `No` | `No` → `hyperVGeneration`/`osVersion`/`vmState` unreliable; start the VM before trusting them. |
| `osType` | `Windows` · `Linux` | Drives OS-support checks for Gen2 and Trusted Launch. |
| `osVersion` / `computerName` | string or blank | Guest OS version and hostname; blank on deallocated VMs. |
| `isMarketplaceVm` | `true` · `false` | `true` → target needs a Gen2 or plan-matched image SKU. |
| `imagePublisher` / `imageOffer` / `imageSku` / `imageVersion` | strings or blank | Source image; all blank = custom or specialized image (trace lineage manually). |
| `diskControllerType` | `SCSI` · `NVMe` · blank | Blank ≈ `SCSI`. NVMe target needs a driver-ready OS. |
| `osDiskType` | `Premium_LRS` · `PremiumV2_LRS` · `StandardSSD_LRS` · `Standard_LRS` · `UltraSSD_LRS` · `*_ZRS` | OS managed-disk SKU. |
| `osDiskSizeGB` | integer | OS disk size (cross-gen move = fresh OS disk). |
| `ephemeralOsDisk` | `Yes` · `No` | `Yes` = stateless; nothing on the OS disk persists. |
| `dataDiskCount` | integer | Data disks to detach and re-attach; check target SKU max. |
| `hibernationEnabled` / `ultraSsdEnabled` | `true` · `false` · blank | Capability flags the target family and zone must also support. |
| `licenseType` | `Windows_Server` · `Windows_Client` · `RHEL_BYOS` · `SLES_BYOS` · blank | Azure Hybrid Benefit or BYOL; blank = pay-as-you-go. |
| `nicCount` | integer | NIC count; check target SKU max. |
| `acceleratedNetworking` | `Yes` · `No` | Enabled on any NIC; target size must support it. |
| `ipForwarding` | `Yes` · `No` | Enabled on any NIC; signals a network-appliance role. |
| `securityType` | `TrustedLaunch` · `ConfidentialVM` · blank | Blank = `Standard`. Gen2 is a prerequisite for Trusted Launch. |
| `secureBootEnabled` / `vTpmEnabled` | `true` · `false` · blank | Trusted Launch posture. |
| `adeOsDiskEnabled` | `true` · blank | Azure Disk Encryption on the OS disk only (not data disks; platform SSE is separate). |
| `subscriptionId` / `resourceId` / `vmssId` / `availabilitySetId` | GUID / ARM ID / blank | Identifiers. |

For the CPU decode, use the following table:

| Letter | CPU platform | Example |
|---|---|---|
| *(none)* | Intel x86-64 | Standard_D4s_v5 |
| `a` | AMD x86-64 (EPYC) | Standard_D4**a**s_v5 |
| `p` | Arm64 (Ampere Altra / Microsoft Cobalt) | Standard_D4**p**s_v5 |

## Exit criteria

- You clear hard gates: SAP workloads have a certified target size, and appliance workloads have written vendor support for the exact family.
- Workloads are scored as ready now, ready after a small refresh, or plan the region or family.
- Target family, region, zone, and quota or capacity plan are documented.
- Any small remediation is assigned.
- Pilot (or greenfield first deployment) is selected.
- Rollback approach is noted, and workload owners approve the first wave.

## Next steps

- [3. Plan a workload modernization to the v6 and v7 series](sizes-v6-v7-modernization-plan.md)
