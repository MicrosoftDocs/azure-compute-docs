---
title: Retired Azure VM size series 
description: A list containing all retired and soon to be retired VM size series and their replacement series.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 06/13/2025
ms.author: mattmcinnes
ms.reviewer: iamwilliew
# Customer intent: As a cloud administrator, I want to review the list of retired virtual machine size series and their migration guides, so that I can ensure my systems are updated and transition to supported VM sizes before any planned retirements.
---

# Retired Azure VM size series

This article provides a list of all sizes that are retired or have been announced for retirement. For sizes that require it there are migration guides to help move to replacement sizes.

> [!WARNING]
> Series with *Retirement Status* listed as *Retired* are **no longer available** and can't be provisioned.
> 
> Virtual Machines (VMs) announced for retirement will have restrictions when deploying through new subscriptions. We recommend using the latest generation VMs to ensure best price-performance and to benefit from extended capacity availability over time for new deployments.

## What are retired size series?
Retired virtual machine size series are running on older hardware which is no longer supported. The hardware is with newer generations of hardware.

Series with *Retirement Status* listed as *Announced* are still available, but will be retired on the *Planned Retirement Date*. It's recommended that you plan your migration to a replacement series well before the listed retirement date.

To learn more about size series retirement, previous-gen sizes, and the retirement process, see the [size series retirement overview](./retirement-overview.md).

> [!IMPORTANT] 
> If you're currently using one of the size series listed as *Retired*, view the migration guide to switch to a replacement series as soon as possible.

*Previous-gen* size series aren't retired and still fully supported, but they have limitations similar to series that are announced for retirement. For a list of previous-gen sizes, see [previous generation Azure VM sizes](../previous-gen-sizes-list.md).

## General purpose retired sizes

|Series name        | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide |
|-------------------|-------------------|-------------------------------------------------------------------|-------------------------|-----------------|
| D-series          | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |
| Ds-series         | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |
| Dv2-series        | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)                |
| Dsv2-series       | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |
| Av2/Amv2-series   | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)            |
| B-series          | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)         |

## Compute optimized retired sizes

|Series name        | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide |
|-------------------|-------------------|-------------------------------------------------------------------|-------------------------|-----------------|
| F-series          | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |
| Fs-series         | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |
| Fsv2-series       | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)               |

## Memory optimized retired sizes

|Series name        | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide |
|-------------------|-------------------|-------------------------------------------------------------------|-------------------------|-----------------|
| G-series          | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)             |
| Gs-series         | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)     |

## Storage optimized retired sizes

|Series name        | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide |
|-------------------|-------------------|-------------------------------------------------------------------|-------------------------|-----------------|
| Ls-series         | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)             |
| Lsv2-series       | **Announced**     | [10/15/25](https://azure.microsoft.com/updates?id=500682)         | 11/15/28                | [Migration Guide](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide)            |

## GPU accelerated retired sizes

| Series name        | Retirement Status |Retirement Announcement      | Planned Retirement Date | Migration Guide |
|--------------------|-------------------|-----------------------------|-------------------------|-----------------|
| NCv3-NC24rs Series | **Retired**       | -                           | 30/9/25                 | [NCv3-NC24rs-series Retirement](../../ncv3-nc24rs-retirement.md) |
| NCv3-Series        | **Retired**       | -                           | 30/9/25                 | [NCv3-series Retirement](../../ncv3-retirement.md)     |

## FPGA accelerated retired sizes

Currently there are no retired FPGA accelerated series retired or announced for retirement.

## HPC retired sizes

Currently there are no retired HPC series retired or announced for retirement.


## ADH retired sizes

Currently there are no retired ADH series retired or announced for retirement.


## Next steps
- For a list of older and capacity limited sizes, see [Previous generation Azure VM sizes](../previous-gen-sizes-list.md).
- For more information on VM sizes, see [Sizes for virtual machines in Azure](../overview.md).
