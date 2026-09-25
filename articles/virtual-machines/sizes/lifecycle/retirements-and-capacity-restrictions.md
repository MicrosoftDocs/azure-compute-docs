---
title: VM size series retirements, capacity growth restrictions, and modernization guidance
description: Review retired and retiring Azure VM size series, capacity growth restrictions for End of Life series beginning July 2026, and modernization guidance to Current or Extended VM families.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 09/24/2026
ms.author: mattmcinnes
---

# VM size series retirements, capacity growth restrictions, and modernization guidance

As Azure continues to invest in higher-performing, more secure, and modern compute infrastructure, it moves away from older-generation hardware. Azure infrastructure expansion focuses on deploying newer-generation hardware that supports the latest virtual machine (VM) offerings.

As a result, certain VM series in the [End of Life](./lifecycle-overview.md#end-of-life) lifecycle stage have an announced retirement date, and some are also subject to capacity growth restrictions starting in July 2026. Plan to modernize to newer-generation VM families and series to ensure continued access to current platform capabilities, improved performance, enhanced security, and long-term capacity availability.

## Retired and retiring VM size series

> [!WARNING]
> Series with *Retirement Status* listed as *Retired* are **no longer available** and can't be provisioned.
>
> VMs in series announced for retirement have restrictions when you deploy through new subscriptions. Use Current VM sizes for new deployments to ensure the best price-performance and extended capacity availability over time.

Retired VM size series run on older hardware that's no longer supported. Series with *Retirement Status* listed as *Announced* are still available until the *Planned Retirement Date*. At the retirement date, any remaining VMs in a retired series are deallocated, stop working, stop incurring charges, and no longer have SLA or support. Plan your modernization to a replacement series well before the retirement date.

*End of Life* size series aren't retired yet and remain supported until their retirement date. For a list of End of Life sizes, see [End of Life Azure VM size series](./end-of-life-sizes-list.md). To learn more about the Current, Extended, End of Life, and Retired lifecycle stages, see the [VM lifecycle overview](./lifecycle-overview.md).

### General purpose retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| D-series | **Announced** | [03/31/25](https://azure.microsoft.com/updates?id=485569) | 05/01/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Ds-series | **Announced** | [03/31/25](https://azure.microsoft.com/updates?id=485569) | 05/01/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dv2-series | **Announced** | [03/31/25](https://azure.microsoft.com/updates?id=485569) | 05/01/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dsv2-series | **Announced** | [03/31/25](https://azure.microsoft.com/updates?id=485569) | 05/01/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Av2/Amv2-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| B-series (V1) | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dv3-series | **Announced** |  | 11/15/29 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dsv3-series | **Announced** |  | 11/15/29 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| DCsv2-series | **Retired** | - | 06/30/26 | [DCsv2-series retirement](./retirement/dcsv2-series-retirement.md) |
| DCas_cc_v5/DCads_cc_v5-series | **Retired** | - | 09/01/26 | [DCas_cc_v5 and DCads_cc_v5 retirement](./retirement/dcccv5-series-retirement.md) |
| DCsv3/DCdsv3-series | **Announced** | - | 10/31/29 | [DCsv3 and DCdsv3 retirement](../retirement/dcsv3-series-retirement.md) |

> [!NOTE]
> The Dv3, Dsv3, Ev3, and Esv3 retirement affects all 32 sizes in these series. After November 15, 2029, you can't create, resize into, run, or purchase these sizes. This retirement doesn't apply to Azure Government, Azure operated by 21Vianet, or sovereign cloud regions. For the smoothest transition, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md). For the latest features and performance, see [Modernize to the v6 and v7 VM series](../../migration/sizes/sizes-v6-v7-migration-overview.md).

### Compute optimized retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| F-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Fs-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Fsv2-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |

### Memory optimized retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| G-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Gs-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Ev3-series | **Announced** |  | 11/15/29 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Esv3-series | **Announced** |  | 11/15/29 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Standard_M192idms_v2 | **Announced** | [03/22/2024](https://azure.microsoft.com/updates?id=support-for-standardm192idmsv2-will-be-retired-on-31-march-2027) | 03/31/27 | [Msv2 and Mdsv2 retirement](./retirement/msv2-mdsv2-retirement.md) |
| Standard_M192ids_v2 | **Announced** | [03/22/2024](https://azure.microsoft.com/updates?id=community-support-for-standardm192idsv2-is-ending-on-31-march-2027) | 03/31/27 | [Msv2 and Mdsv2 retirement](./retirement/msv2-mdsv2-retirement.md) |
| Standard_M192ims_v2 | **Announced** | [03/22/2024](https://azure.microsoft.com/updates?id=community-support-for-standardm192imsv2-is-ending-on-31-march-2027) | 03/31/27 | [Msv2 and Mdsv2 retirement](./retirement/msv2-mdsv2-retirement.md) |
| Standard_M192is_v2 | **Announced** | [03/22/2024](https://azure.microsoft.com/updates?id=community-support-for-standardm192isv2-is-ending-on-31-march-2027) | 03/31/27 | [Msv2 and Mdsv2 retirement](./retirement/msv2-mdsv2-retirement.md) |
| ECas_cc_v5/ECads_cc_v5-series | **Retired** | - | 09/01/26 | [ECas_cc_v5 and ECads_cc_v5 retirement](./retirement/ecccv5-series-retirement.md) |

### Storage optimized retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| Ls-series | **Announced** | [03/31/25](https://azure.microsoft.com/updates?id=485569) | 05/01/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Lsv2-series | **Announced** | [10/15/25](https://azure.microsoft.com/updates?id=500682) | 11/15/28 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |

### GPU accelerated retired sizes

NVv3-series and NVv4-series have announced retirements planned for September 30, 2026. Review the following entries and modernize to a supported alternative before that date.

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| NCv3-NC24rs Series | **Retired** | - | 09/30/25 | [NCv3-NC24rs-series Retirement](./retirement/ncv3-nc24rs-retirement.md) |
| NCv3-Series | **Retired** | - | 09/30/25 | [NCv3-series Retirement](./retirement/ncv3-retirement.md) |
| NVv3-series | **Announced** | [04/15/25](https://azure.microsoft.com/updates?id=516070) | 09/30/26 | [NVv3-series Retirement](./retirement/nvv3-series-retirement.md) |
| NVv4-series | **Announced** | [04/15/25](https://azure.microsoft.com/updates?id=516070) | 09/30/26 | [NVv4-series Retirement](./retirement/nvv4-retirement.md) |

### FPGA accelerated retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| NP-series | **Announced** | [04/02/2026](https://azure.microsoft.com/updates?id=548497) | 05/31/27 | [NP-series retirement](./retirement/np-series-retirement.md) |

> [!NOTE]
> After the retirement date, any remaining NP-series VMs (Standard_NP10s, Standard_NP20s, Standard_NP40s) are deallocated, stop working, stop incurring charges, and no longer have SLA or support. Managed disk data is preserved. Plan to modernize well in advance, because purchases of 1-year and 3-year Azure Reserved VM Instances for NP-series ended on April 2, 2026.

### HPC retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| HC-series | **Announced** | - | 05/31/27 | [HC-series retirement](./retirement/hc-series-retirement.md) |
| HBv2-series | **Announced** | - | 05/31/27 | [HBv2-series retirement](./retirement/hbv2-series-retirement.md) |

### ADH retired sizes

| Series name | Retirement Status | Retirement Announcement | Planned Retirement Date | Modernization guide |
|---|---|---|---|---|
| Dsv3-Type1, Dsv3-Type2, Esv3-Type1, Esv3-Type2 | **Retired** | 03/15/22 | 06/30/23 | [Dedicated Host SKU retirement](./retirement/dedicated-host-retirement.md) |

## Impacted VM series

The following VM series are affected by capacity growth restrictions:

| VM category | Impacted VM series |
|---|---|
| Compute optimized | [F](../compute-optimized/f-family.md), [Fs](../compute-optimized/f-family.md), [Fsv2](../compute-optimized/fsv2-series.md) |
| General purpose | [D](../general-purpose/d-family.md), [Ds](../general-purpose/d-family.md), [Dv2](../general-purpose/dv2-series.md), [Dsv2](../general-purpose/dsv2-series.md), [Dv3](../general-purpose/dv3-series.md), [Dsv3](../general-purpose/dsv3-series.md), [B](../general-purpose/b-family.md), [Bs](../general-purpose/b-family.md), [Av2](../general-purpose/av2-series.md), [Amv2](../general-purpose/a-family.md) |
| Memory optimized | [Ev3](../memory-optimized/e-family.md), [Esv3](../memory-optimized/e-family.md), [G](../memory-optimized/m-family.md), [Gs](../memory-optimized/m-family.md) |
| Storage optimized | [Ls](../storage-optimized/l-family.md), [Lsv2](../storage-optimized/lsv2-series.md) |

### Quota limitations
Existing subscriptions can continue to deploy the preceding VMs within already approved quota, subject to capacity availability. New subscriptions can't deploy the affected VM series.

| Scenario | Outcome |
|---|---|
| New subscription | Can't deploy affected SKUs. |
| Existing subscription using already-approved quota | Can deploy or redeploy affected SKUs, subject to capacity availability. |
| Existing subscription requesting additional quota | Additional quota isn't approved. |
| Existing subscription with quota, but insufficient regional capacity | Deployment might fail even though quota is available. |
| Existing subscriptions using shared Capacity Reservations | No impact on deployment if existing subscription uses quota from shared pool and within approved limits. |

## Recommended modernization paths

If you use impacted VM series, modernize to Current or Extended VM families. Recommended modernization targets generally include v5, v6, and v7 VM series in the Current or Extended stage, depending on workload requirements, performance objectives, and storage compatibility considerations. Use Current sizes for new deployments.

### Modernization recommendations

| Current VM category | Impacted VM series | Recommended target VM series |
|---|---|---|
| General purpose | [Dv3](../general-purpose/dv3-series.md), [Dsv3](../general-purpose/dsv3-series.md)| [Dv5](../general-purpose/dv5-series.md), [Dv6](../general-purpose/d-family.md), [Dv7](../general-purpose/d-family.md) (based on workload requirements) |
| Memory optimized | [Ev3](../memory-optimized/ev3-esv3-series.md), [Esv3](../memory-optimized/ev3-esv3-series.md) | [Ev5](../memory-optimized/ev5-series.md), [Esv6](../memory-optimized/esv6-series.md), [Esv7](../memory-optimized/esv7-series.md) (based on workload requirements) |

### Modernization resources

To help identify the most suitable replacement VM, review the following resources:

- [VM modernization guidance](./retirement/retired-sizes-modernization-guide.md)
- [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md)
- [Modernize to the v6 and v7 VM series](../../migration/sizes/sizes-v6-v7-migration-overview.md)

These resources provide detailed recommendations for alternative virtual machine series, modernization planning considerations, and workload-specific guidance.

## Frequently asked questions

### What do "capacity growth restrictions" mean?

Capacity growth restrictions mean that Azure might not approve additional quota requests, new deployments, or capacity expansion requests for impacted VM series due to limited availability of the underlying hardware. These capacity controls don't affect existing running VMs.

### When do capacity growth restrictions begin?

Capacity growth restrictions for the impacted VM series begin in July 2026.

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

### Does this restriction mean the D(s)v3 and E(s)v3 VM series are retired?

No. This restriction is about capacity growth, not retirement. Retirement follows a separate process with dedicated customer communications, timelines, and transition guidance.

Separately from these capacity growth restrictions, the Dv3, Dsv3, Ev3, and Esv3 series are in the End of Life stage and retire on November 15, 2029. Until that date, these series remain supported under Azure SLAs, subject to the capacity growth restrictions in this article. For retirement details, see [Retired and retiring VM size series](#retired-and-retiring-vm-size-series).

### Can customers continue normal operations on existing VMs and environments as long as they stay within their approved quota?

Existing subscriptions with previously approved quota aren't affected by this change. Customers can continue deploying impacted VM SKUs within their existing approved quota limits, subject to capacity availability. However, approved quota doesn't guarantee capacity availability in the region and deployments might still fail if sufficient capacity isn't available.

### Where can I find retirement and transition guidance for the D(s)v3 and E(s)v3 VM series?

The Dv3, Dsv3, Ev3, and Esv3 series retire on November 15, 2029. After that date, you can't create, resize into, run, or purchase these sizes. For retirement dates and recommended replacements, see the [Retired VM sizes modernization guide](./retirement/retired-sizes-modernization-guide.md).

For the smoothest transition, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md). For the latest features and performance, see [Modernize to the v6 and v7 VM series](../../migration/sizes/sizes-v6-v7-migration-overview.md).

### Does reimaging one of the impacted VM series in a Virtual Machine Scale Set deallocate it?

No. Reimaging replaces the OS disk so that operating-system and configuration changes can be applied. It doesn't remove the VM or change its VM series. For details, see [Reimage a virtual machine in a scale set](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-reimage-virtual-machine).

### How do I identify the best replacement VM for my workload?

Azure provides VM modernization guidance to help you identify the most appropriate replacement VM family and series based on workload requirements, performance needs, and storage compatibility considerations. Review the [VM modernization guidance](./retirement/retired-sizes-modernization-guide.md) before selecting a replacement VM.

### Are there pricing differences between End of Life and newer VM families?

Pricing, performance, storage capabilities, and hardware characteristics might vary across VM generations. Validate workload requirements and review the documentation for the recommended target VM families and series before you modernize.

### Reserved instances

#### Will existing reserved instances (RIs) continue to work after the quota growth restrictions, including during redeployment, disaster recovery, or failover?

Yes. Existing RIs remain valid through their purchased term and continue providing discounts for matching usage after quota growth restrictions take effect. Capacity growth restrictions don't invalidate existing RIs.

RIs are billing benefits, not capacity reservations. An RI discount might apply to matching usage, but it doesn't guarantee VM creation, redeployment, disaster recovery, failover, or scaling if capacity is unavailable.

Existing subscriptions can deploy or redeploy affected VM sizes within approved quota, subject to capacity availability. New subscriptions, additional quota, and capacity expansion might be restricted.

Review disaster recovery and restore plans, modernize to newer VM generations where appropriate, and update RIs, Savings Plans, and capacity strategies to align with the target environment.

#### Do RIs provide protection from growth restrictions, and how do they work with on-demand capacity reservations?

Purchasing an RI doesn't reserve capacity or protect against capacity growth restrictions. RIs provide a discount for eligible usage but don't guarantee capacity availability.

Customers who require capacity assurance should evaluate on-demand capacity reservations (ODCRs). Customers who need flexibility across VM families or regions should consider Azure Savings Plan for Compute.

RIs and ODCRs serve different purposes: RIs provide billing discounts, while ODCRs reserve capacity for a specific VM size, region, and zone.

#### What happens to existing RIs and costs when I modernize from applicable A, B, D, E, F, or L variants of v1 through v3 series to v5 or later?

RIs don't automatically transfer when you modernize to a different VM series. Exchange an eligible RI to match the new VM series or transition to Azure Savings Plan for Compute. If you don't make an update, the RI might no longer apply, which could result in additional costs.

RI exchanges are available until February 1, 2027. After that date, each eligible active reservation purchased before the deadline retains one final exchange.

#### What commitment discounts remain available, and how should I plan future purchases or renewals?

- For affected applicable A, B, D, E, F, or L variants of v1 through v3 VM series, new or renewed one-year and three-year RIs are no longer available.
- For newer VM generations, RIs remain available for eligible VM series.
- Azure Savings Plan for Compute remains available as a flexible commitment option across eligible VM generations.

Review the reservation portfolio before February 1, 2027, and plan how to use any remaining exchange rights for eligible active RIs. For more information, see [Manage Azure Reservations](/azure/cost-management-billing/reservations/manage-reserved-vm-instance).

#### What options are available for customers with large usage of applicable A, B, D, E, F, or L variants of v1 through v3 when existing RIs expire? Is there an exception process for RI renewals?

Existing RIs remain valid through their committed term and continue providing discounts for eligible usage. However, no exceptions or extensions are planned for RI purchase or renewal end dates. Work with your Microsoft account team to evaluate modernization and alternative commitment options.

Affected applicable A, B, D, E, F, or L variants of v1 through v3 RIs can't be renewed or repurchased after expiration. Once expired, usage moves to pay-as-you-go rates unless you select another commitment option.

You can:

- Continue running affected VMs at pay-as-you-go rates.
- Transition to Azure Savings Plan for Compute.
- Modernize to newer VM generations and purchase new RIs, an Azure Savings Plan for Compute, or both.

### On-demand capacity reservations (ODCR)

#### Do existing capacity reservations for impacted VM series continue to function after the quota restriction takes effect?

Yes. Existing on-demand capacity reservations (ODCRs) continue to function because they consume existing quota allocations. You can continue using your reservations up to the limits of both your available quota and reserved capacity.

#### Can I continue deploying VMs against an existing capacity reservation for an impacted VM series?

Yes. You can continue deploying VMs against an existing capacity reservation within the reservation's allocated capacity. You don't need extra quota to deploy VMs that consume capacity already reserved within the same subscription.

> [!IMPORTANT]
> You might encounter problems when using shared capacity reservations across subscriptions. In this scenario, the consuming subscription might require extra quota to use the shared reservation. Because quota increases are restricted for the affected VM series, this scenario could prevent extra deployments even when reserved capacity is available.

#### Can I create new capacity reservations for impacted VM series after the quota restriction is implemented?
Yes, as long as quota is already available within an existing subscription. However, be aware that scenarios involving shared capacity reservations across subscriptions might be impacted because you can't grant extra quota to consuming subscriptions after the quota freeze takes effect.
