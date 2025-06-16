---
title: Retired Azure VM size series 
description: A list containing all retired and soon to be retired VM size series and their replacement series.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 05/28/2025
ms.author: mattmcinnes
ms.reviewer: iamwilliew
---

# Retired Azure VM size series

This article provides a list of all sizes that are retired or have been announced for retirement. For sizes that require it there are migration guides to help move to replacement sizes.

> [!WARNING]
> Series with *Retirement Status* listed as *Retired* are **no longer available** and can't be provisioned.

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
| D-series          | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | -               |
| Ds-series         | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | -               |
| Dv2-series        | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | -               |
| Dsv2-series       | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | -               |
| Av1-series        | **Retired**       | 11/02/23                                                          | 8/31/24                 | [Av1-series Retirement](./av1-series-retirement.md)  |

## Compute optimized retired sizes

Currently there are no compute optimized series retired or announced for retirement.

## Memory optimized retired sizes

Currently there are no memory optimized series retired or announced for retirement.

## Storage optimized retired sizes

|Series name        | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide |
|-------------------|-------------------|-------------------------------------------------------------------|-------------------------|-----------------|
| Ls-series         | **Announced**     | [03/31/25](https://azure.microsoft.com/updates?id=485569)         | 05/01/28                | -               |

## GPU accelerated retired sizes

| Series name       | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide
|-------------------|-------------------|-----------------------------|-------------------------|-----------------|
| NV-Series         | **Retired**       | -                           | 9/6/23                  | [NV-series Retirement](./nv-series-retirement.md)    |
| NC-Series         | **Retired**       | -                           | 9/6/23                  | [NC-series Retirement](./nc-series-retirement.md)    |
| NCv2-Series       | **Retired**       | -                           | 9/6/23                  | [NCv2-series Retirement](./ncv2-series-retirement.md)  |
| ND-Series         | **Retired**       | -                           | 9/6/23                  | [ND-series Retirement](./nd-series-retirement.md)    |

## FPGA accelerated retired sizes

Currently there are no retired FPGA accelerated series retired or announced for retirement.

## HPC retired sizes

| Series name       | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide
|-------------------|-------------------|-----------------------------|-------------------------|-----------------|
| HB-Series         | **Announced**     | 12/07/23                    | 8/31/24                  | [NV-series Retirement](./nv-series-retirement.md)    |

## ADH retired sizes

| Series name       | Retirement Status |Retirement Announcement                                            | Planned Retirement Date | Migration Guide
|-------------------|-------------------|-----------------------------|-------------------------|-----------------|
| Dsv3-Type1        | **Retired**       | -                           | 6/30/23                  | [Dedicated Host SKU Retirement](./dedicated-host-retirement.md)    |
| Dsv3-Type2        | **Retired**       | -                           | 6/30/23                  | [Dedicated Host SKU Retirement](./dedicated-host-retirement.md)    |
| Esv3-Type1        | **Retired**       | -                           | 6/30/23                  | [Dedicated Host SKU Retirement](./dedicated-host-retirement.md)    |
| Esv3-Type2        | **Retired**       | -                           | 6/30/23                  | [Dedicated Host SKU Retirement](./dedicated-host-retirement.md)    |


## Next steps
- For a list of older and capacity limited sizes, see [Previous generation Azure VM sizes](../previous-gen-sizes-list.md).
- For more information on VM sizes, see [Sizes for virtual machines in Azure](../overview.md).
