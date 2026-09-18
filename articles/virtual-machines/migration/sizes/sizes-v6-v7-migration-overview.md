---
title: Migrate to the v6 and v7 VM series
description: Overview of migrating Azure VM workloads from the v2 through v5 series to the v6 and v7 series—what changes, effort by starting point, target families, and the migration journey.
author: rod-reis
ms.author: rosanto
ms.service: azure-virtual-machines
ms.topic: overview
ms.date: 07/24/2026
ms.collection:
  - migration
  - v2-5-to-v6-7
ai-usage: ai-assisted

#customer intent: As a workload architect and engineer, I want to understand how to migrate to Azure Virtual Machines from Gen 1 v2-v3-v4-v5 to Gen 2 v6-v7 as part of my workload's efficiency optimization in Azure. Without this guidance I will miss behavior differences or implementation details that could cause my migration experience delay, frustration, or be to a failure.
---

# Migrate to the v6 and v7 VM series

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

**Workload patterns:** ✔️ All patterns

The v6 and v7 Azure VM series give your workloads newer, faster infrastructure with strong price-performance. Confirm platform prerequisites (boot mode, image, storage interface, network driver, and regional availability), deploy from an updated image, and validate in controlled waves.

This migration playbook is for architects and infrastructure teams planning a move to the v6 or v7 series, for a single workload or across a large estate.

This article introduces the migration and links to the detailed phases: **Discover**, **Assess**, **Plan**, **Migrate**, and **Validate & optimize**. It applies whether you're moving existing VMs from the v2 through v5 series or deploying greenfield.

> [!NOTE]
> The changes in this migration affect the physical host, the virtual hardware, and the image used to create the VM. Your application, its configuration, autoscale rules, and health logic stays the same. Stateful and clustered workloads are the exception: the application itself doesn't change, but the migration adds application-aware steps for replication and role transfer. See [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md).

## Discover migration pattern by workload type

The migration effort depends on how the workload deploys, stores state, and recovers from the replacement of an individual VM. Because the prerequisites apply per image, not per VM, workloads built from a shared, validated image migrate by replacing the pool, while customer-managed applications, clustered databases, and virtual appliances need application-aware replication, configuration migration, or vendor certification.

Seven patterns cover most estates:

- **A. Compute pools** and **B. Image-based hosts** replace a pool of disposable nodes or session hosts.
- **C. Service-managed compute** and **D. Cluster re-creation** are driven by the service's supported-size list.
- **E. Customer-managed VMs** and **F. Stateful and clustered** replace individual VMs, with F adding state synchronization and role transfer.
- **G. Certified appliances** are gated on vendor certification for the target family.

Discover each workload's pattern first, most estates contain several, and the pattern determines which phases of this journey apply to you. For examples, migration approach, effort, and decision criteria for each pattern, see [Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md).


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
- **Availability and commercial:** Confirm [regional and zonal availability](/azure/reliability/availability-zones-overview) and quota for the target family. For capacity reservations and family-scoped discount replanning, see [Plan the migration](sizes-v6-v7-migration-plan.md#region-zone-and-capacity-planning).

> [!IMPORTANT]
> Treat this migration as a planned upgrade, not as:
>
> - A blind in-place resize from an older family.
> - A guaranteed zonal fit in every region without checking availability and quota.
> - A move that skips a one-time image and driver refresh.

## The migration journey

1. **[Discover](sizes-v6-v7-migration-discover.md).** Determine which pattern each workload follows. The pattern decides which of the phases below apply to you, and service-managed workloads finish here.
2. **[Assess](sizes-v6-v7-migration-assess.md).** Use a short readiness check to separate ready-now candidates from those needing image, driver, path, or capacity remediation.
3. **[Plan](sizes-v6-v7-migration-plan.md).** Work through the considerations for Generation 2 boot, NVMe storage, MANA networking, Azure Boost offload, region, zone, capacity, and images.
4. **[Migrate](sizes-v6-v7-migration-migrate.md).** Use a wave model that proves the platform pattern once, then scales through controlled rings.
5. **[Validate and optimize](sizes-v6-v7-migration-validate.md).** Keep validation focused on boot, disks, drivers, networking, and workload-owner sign-off, then optimize for cost and performance.

## Next steps

- [1. Discover migration pattern by workload type](sizes-v6-v7-migration-discover.md)
