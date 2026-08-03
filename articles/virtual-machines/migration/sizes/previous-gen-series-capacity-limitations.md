---
title: Previous-generation VM size series capacity limitations and migration guidance
description: Learn about capacity limitations for previous-generation Azure VM size series beginning July 2026 and get migration guidance to newer-generation VM families.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 08/03/2026
ms.author: mattmcinnes
---

# Previous-generation VM size series capacity limitations and migration guidance

As Azure continues to invest in higher-performing, more secure, and modern compute infrastructure, it moves away from older-generation hardware. Azure infrastructure expansion focuses on deploying newer-generation hardware that supports the latest virtual machine (VM) offerings.

As a result, certain previous-generation VM series are subject to capacity limitations starting in July 2026. Plan migrations to newer-generation VM families and series to ensure continued access to current platform capabilities, improved performance, enhanced security, and long-term capacity availability.

## Impacted VM series

The following VM series are affected by these capacity limitations:

| VM category | Impacted VM series |
|---|---|
| Compute optimized | [F](../../sizes/compute-optimized/f-family.md), [Fs](../../sizes/compute-optimized/f-family.md), [Fsv2](../../sizes/compute-optimized/fsv2-series.md) |
| General purpose | [D](../../sizes/general-purpose/d-family.md), [Ds](../../sizes/general-purpose/d-family.md), [Dv2](../../sizes/general-purpose/dv2-series.md), [Dsv2](../../sizes/general-purpose/dsv2-series.md), [Dv3](../../sizes/general-purpose/dv3-series.md), [Dsv3](../../sizes/general-purpose/dsv3-series.md), [Dv4](../../sizes/general-purpose/dv4-series.md), [Dsv4](../../sizes/general-purpose/dsv4-series.md), [Ddv4](../../sizes/general-purpose/ddv4-series.md), [Ddsv4](../../sizes/general-purpose/ddsv4-series.md), [Dav4](../../sizes/general-purpose/dav4-series.md), [Dasv4](../../sizes/general-purpose/dasv4-series.md), [B](../../sizes/general-purpose/b-family.md), [Bs](../../sizes/general-purpose/b-family.md), [Av2](../../sizes/general-purpose/av2-series.md), [Amv2](../../sizes/general-purpose/a-family.md) |
| Memory optimized | [Ev3](../../sizes/memory-optimized/e-family.md), [Esv3](../../sizes/memory-optimized/e-family.md), [Ev4](../../sizes/memory-optimized/ev4-series.md), [Esv4](../../sizes/memory-optimized/esv4-series.md), [Edv4](../../sizes/memory-optimized/edv4-series.md), [Edsv4](../../sizes/memory-optimized/edsv4-series.md), [Eav4](../../sizes/memory-optimized/eav4-series.md), [Easv4](../../sizes/memory-optimized/easv4-series.md), [G](../../sizes/memory-optimized/m-family.md), [Gs](../../sizes/memory-optimized/m-family.md) |
| Storage optimized | [Ls](../../sizes/storage-optimized/l-family.md), [Lsv2](../../sizes/storage-optimized/lsv2-series.md) |

### Quota limitations
Existing subscriptions can continue to deploy the preceding VMs within already approved quota, subject to capacity availability. New subscriptions can't deploy the affected VM series. 

| Scenario | Outcome |
|---|---|
| New subscription | Can't deploy affected SKUs. |
| Existing subscription using already-approved quota | Can deploy or redeploy affected SKUs, subject to capacity availability. |
| Existing subscription requesting additional quota | Additional quota isn't approved. |
| Existing subscription with quota, but insufficient regional capacity | Deployment might fail even though quota is available. |
| Existing subscriptions using shared Capacity Reservations | No impact on deployment if existing subscription uses quota from shared pool and within approved limits. |

## Recommended migration paths

If you use impacted VM series, migrate to newer-generation VM families. Recommended migration targets generally include v5, v6, and v7 VM series, depending on workload requirements, performance objectives, and storage compatibility considerations.

### Migration recommendations

| Current VM category | Impacted VM series | Recommended target VM series |
|---|---|---|
| General purpose | [Dv3](../../sizes/general-purpose/dv3-series.md), [Dsv3](../../sizes/general-purpose/dsv3-series.md), [Dv4](../../sizes/general-purpose/dv4-series.md), [Dsv4](../../sizes/general-purpose/dsv4-series.md), [Ddsv4](../../sizes/general-purpose/ddsv4-series.md), [Ddav4](../../sizes/general-purpose/d-family.md), [Dav4](../../sizes/general-purpose/dav4-series.md), [Dasv4](../../sizes/general-purpose/dasv4-series.md) | [Dv5](../../sizes/general-purpose/dv5-series.md), [Dv6](../../sizes/general-purpose/d-family.md), [Dv7](../../sizes/general-purpose/d-family.md) (based on workload requirements) |
| Memory optimized | [Ev3](../../sizes/memory-optimized/ev3-esv3-series.md), [Esv3](../../sizes/memory-optimized/ev3-esv3-series.md), [Ev4](../../sizes/memory-optimized/ev4-series.md), [Esv4](../../sizes/memory-optimized/esv4-series.md), [Edv4](../../sizes/memory-optimized/edv4-series.md), [Edsv4](../../sizes/memory-optimized/edsv4-series.md), [Easv4](../../sizes/memory-optimized/easv4-series.md), [Eav4](../../sizes/memory-optimized/eav4-series.md) | [Ev5](../../sizes/memory-optimized/ev5-series.md), [Esv6](../../sizes/memory-optimized/esv6-series.md), [Esv7](../../sizes/memory-optimized/esv7-series.md) (based on workload requirements) |

### Migration resources

To help identify the most suitable replacement VM, review the following resources:

- [VM migration guidance](../../sizes/lifecycle/retirement/d-ds-dv2-dsv2-ls-series-migration-guide.md)
- [Migrate to the v6 and v7 VM series](sizes-v6-v7-migration-overview.md)

These resources provide detailed recommendations for alternative virtual machine series, migration planning considerations, and workload-specific guidance.

## Frequently asked questions

### What does "capacity limitations" mean?

Capacity limitations mean that Azure might not approve additional quota requests, new deployments, or capacity expansion requests for impacted VM series due to limited availability of the underlying hardware. These capacity controls don't affect existing running VMs.

### When do capacity limitations begin?

Capacity limitations for the impacted VM series begin in July 2026.

### Why are these VM series affected?

Azure prioritizes investment in newer-generation infrastructure that delivers improved performance, security, reliability, and platform capabilities. Older VM generations depend on hardware platforms that are no longer a focus for infrastructure expansion.

### Can an existing subscription continue deploying affected VM SKUs after July 31, 2026?

Yes. An existing subscription can continue to deploy, redeploy, and operate affected VM SKUs within its already-approved quota, subject to capacity availability.

> [!IMPORTANT]
> Quota is an approved limit; it isn't a capacity reservation or guarantee. A VM deployment might still encounter an [allocation failure](/troubleshoot/azure/virtual-machines/windows/allocation-failure) if capacity is unavailable in the requested region or zone.

### Can an existing subscription request more quota for an affected VM series?

No. You can't get more quota for affected VM series. Plan new growth on newer-generation VM series.

### Can a new subscription deploy an affected VM SKU?

No. New subscriptions are restricted from deploying the affected VM series.

### What happens to quota increase requests already submitted?

Requests submitted before enforcement took effect continue through the standard review process. Rollout timing might vary by region or system, so some customers might encounter the restriction before July 31. If you submitted the request, it proceeds through normal evaluation. If there's no option to request quota for an affected VM series, the restriction is active.

### Does this restriction mean the D(s)v3 and E(s)v3 or D(d)(s)v4 and E(d)(s)v4 VM series are retired?

No. This restriction is about capacity growth, not retirement. Retirement follows a separate process with dedicated customer communications, timelines, and migration guidance. This notice doesn't include a retirement announcement for the D(s)v3 and E(s)v3 or D(d)(s)v4 and E(d)(s)v4 series.

### Can customers continue normal operations on existing VMs and environments as long as they stay within their approved quota?

Existing subscriptions with previously approved quota aren't affected by this change. Customers can continue deploying impacted VM SKUs within their existing approved quota limits, subject to capacity availability. However, approved quota doesn't guarantee capacity availability in the region and deployments might still fail if sufficient capacity isn't available.

### Why aren't D(s)v3 and E(s)v3 or D(d)(s)v4 and E(d)(s)v4 VM series listed in the Retired VM Sizes Migration Guide?

The Retired VM Sizes Migration Guide only includes VM series with a published retirement plan. While [some of the VM series](/azure/virtual-machines/sizes/lifecycle/retirement/d-ds-dv2-dsv2-ls-series-migration-guide) affected by the capacity growth restrictions have retirement dates, the D(s)v3 and E(s)v3 or D(d)(s)v4 and E(d)(s)v4 VM series don't currently have a published retirement plan and remain supported until further notice. These VM series continue to be fully supported under Azure SLAs.  The recommended VM series for these VMs are D(d)sv5 or newer and E(d)s v5 and newer.

### Does reimaging one of the impacted VM series in a Virtual Machine Scale Set deallocate it?

No. Reimaging replaces the OS disk so that operating-system and configuration changes can be applied. It doesn't remove the VM or change its VM series. For details, see [Reimage a virtual machine in a scale set](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-reimage-virtual-machine).

### How do I identify the best replacement VM for my workload?

Azure provides VM migration guidance to help you identify the most appropriate replacement VM family and series based on workload requirements, performance needs, and storage compatibility considerations. Review the [VM migration guidance](/azure/virtual-machines/sizes/lifecycle/retirement/d-ds-dv2-dsv2-ls-series-migration-guide) before selecting a replacement VM.

### Are there pricing differences between previous-generation and newer-generation VM families?

Pricing, performance, storage capabilities, and hardware characteristics might vary across VM generations. Validate workload requirements and review the documentation for the recommended target VM families and series before migration.

### On-demand capacity reservations (ODCR)

#### Do existing capacity reservations for impacted VM series continue to function after the quota restriction takes effect?

Yes. Existing on-demand capacity reservations (ODCRs) continue to function because they consume existing quota allocations. You can continue using your reservations up to the limits of both your available quota and reserved capacity.

#### Can I continue deploying VMs against an existing capacity reservation for an impacted VM series?

Yes. You can continue deploying VMs against an existing capacity reservation within the reservation's allocated capacity. You don't need extra quota to deploy VMs that consume capacity already reserved within the same subscription.

> [!IMPORTANT]
> You might encounter problems when using shared capacity reservations across subscriptions. In this scenario, the consuming subscription might require extra quota to use the shared reservation. Because quota increases are restricted for the affected VM series, this scenario could prevent extra deployments even when reserved capacity is available.

#### Can I create new capacity reservations for impacted VM series after the quota restriction is implemented?
Yes, as long as quota is already available within an existing subscription. However, be aware that scenarios involving shared capacity reservations across subscriptions might be impacted because you can't grant extra quota to consuming subscriptions after the quota freeze takes effect.

