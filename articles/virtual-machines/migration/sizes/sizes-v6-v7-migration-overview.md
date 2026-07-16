---
title: Migrate to the v6 and v7 VM series
description: Overview of migrating Azure VM workloads from the v2 through v5 series to the v6 and v7 series—what changes, effort by starting point, target families, and the migration journey.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: overview
ms.date: 07/03/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Migrate to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

The v6 and v7 Azure VM series give your workloads newer, faster infrastructure with strong price-performance. The move is a well-understood pattern: confirm a short but important list of platform prerequisites (boot mode, image, storage interface, network driver, and regional availability), deploy from an updated image, and validate in controlled waves.

This article introduces the migration and links to the detailed phases: **Assess**, **Plan**, **Migrate**, and **Validate and optimize**. It applies whether you're moving existing VMs from the v2 through v5 series or deploying greenfield.

> [!NOTE]
> The changes in this migration affect the physical host, the virtual hardware, and the image used to create the VM. Everything else *inside* the VM, your application, its configuration, autoscale rules, and health logic stays the same.

## Migration effort by starting point

The same short prerequisite list applies in every case. The remediation effort decreases as the source generation gets newer, and greenfield is the simplest because there's nothing to remediate.

| Starting point | What it usually means | Effort |
| --- | --- | --- |
| v2 / v3 (most common) | Usually Generation 1, an older OS image, SCSI disks, and a local temporary disk in use. The prerequisites are well understood and one-time. | Moderate, mostly an image refresh with a generation change. |
| v4 | Frequently Generation 2 already; the main work is image/driver and disk-path confirmation. | Low. |
| v5 | Usually Generation 2 and close to ready; mostly a validation pass. | Very low. |
| Greenfield | Start on a current Generation 2, NVMe- and MANA-ready image; no remediation. | Minimal. |

## Target VM families

| Family | Examples | Platform |
| --- | --- | --- |
| v6 (Intel) | Dlsv6, Dsv6, Esv6 | Intel Emerald Rapids with Azure Boost |
| v6 (AMD) | Dalsv6, Dasv6, Easv6 | AMD Genoa with Azure Boost |
| v7 (Intel) | Dlsv7, Dsv7, Esv7 | Intel Xeon 6 (Granite Rapids) with Azure Boost |
| v7 (AMD) | Dasv7, Easv7, Fasv7 | AMD Turin (5th Gen EPYC) with Azure Boost |

Choose the family by workload profile, then confirm region, zone, and quota. Availability differs by series and region, for example, the AMD v7 families reached general availability ahead of some Intel v7 sizes, so verify the specific size in your target regions. Sizes ending in **`d`** (for example, Ddsv6 or Ddsv7) include a local NVMe disk; sizes without it don't. For more information, see [VM sizes](/azure/virtual-machines/sizes/overview) and the [VM naming conventions](/azure/virtual-machines/vm-naming-conventions).

## What changes when the VM family changes

Focus your planning on the items that actually change at the platform level:

- **Boot mode:** The v6 and v7 series use Generation 2 (UEFI) and support [Trusted Launch](/azure/virtual-machines/trusted-launch) (Secure Boot and vTPM). Trusted Launch is the default security type for Generation 2 but isn't required.
- **Storage interface:** Disks are presented over [NVMe](/azure/virtual-machines/nvme-overview). The image and OS must include NVMe support.
- **Networking:** The [MANA](/azure/virtual-network/accelerated-networking-mana-overview) adapter requires a current OS and driver.
- **Local (temporary) disk:** Present only on `d`-suffixed sizes, and presented as NVMe.
- **Image:** Use a current Generation 2, NVMe- and MANA-ready marketplace or [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) image.
- **Availability and Commercial:** Confirm [regional and zonal availability](/azure/reliability/availability-zones-overview) and quota, consider [capacity reservations](/azure/virtual-machines/capacity-reservation-overview), and replan any family-scoped [reservations or savings plans](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations).

> [!IMPORTANT]
> Treat this migration as a planned upgrade, not as:
>
> - A blind in-place resize from an older family.
> - A guaranteed zonal fit in every region without checking availability and quota.
> - A move that skips a one-time image and driver refresh.

## The migration journey

1. **[Assess](sizes-v6-v7-migration-assess.md).** Use a short assessment to separate ready-now candidates from those needing image, driver, path, or capacity remediation.
1. **[Plan](sizes-v6-v7-migration-plan.md).** Work through the considerations for Generation 2 boot, NVMe storage, MANA networking, Azure Boost offload, region, zone, capacity, and images.
1. **[Migrate](sizes-v6-v7-migration-migrate.md).** Use a wave model that proves the platform pattern once, then scales through controlled rings.
1. **[Validate and optimize](sizes-v6-v7-migration-validate.md).** Keep validation focused on boot, disks, drivers, networking, and workload-owner sign-off, then optimize for cost and performance.

## Next steps

- [Assess readiness for the v6 and v7 series](sizes-v6-v7-migration-assess.md)
- [Plan a workload migration to the v6 and v7 series](sizes-v6-v7-migration-plan.md)
- [VM sizes overview](/azure/virtual-machines/sizes/overview)
