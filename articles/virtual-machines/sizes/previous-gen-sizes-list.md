---
title: Previous generation Azure VM size series
description: A list containing all previous generation and capacity limited VM size series.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 06/17/2025
ms.author: mattmcinnes
ms.reviewer: iamwilliew
# Customer intent: As a cloud architect, I want to understand the previous generation Azure VM size series, so that I can make informed decisions about migrating to newer VM sizes to optimize performance and capacity for my applications.
---

# Previous generation Azure VM size series

This article provides a list of all size series that are considered *previous-gen*. Status is listed as *next-gen available* or *capacity limited* based on capacity. For sizes that require it there are *migration guides* to help move to replacement sizes.

> [!NOTE]
> Series listed as *previous-gen* are **not currently retired** and can still be used. However, series announced for retirement will have restrictions when deploying through new subscriptions. To view retired size series, see [Retired Azure VM size series](./retired-sizes-list.md).

## What are previous-gen size series?
Previous generation virtual machine size series are running on older hardware. While they can still be used and are fully supported, there are newer generations available. It's recommended to migrate to the latest generation replacements.

To learn more about previous-gen sizes, retirement, and the status states of previous-gen size series, see the [size series retirement overview](./retirement-overview.md).

## General purpose previous-gen sizes

|Series name                 | Status                 | Migration guide   |
|----------------------------|------------------------|-------------------|
| B-series (V1)              | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard D-series          | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Preview DC-series          | [Capacity limited](./retirement-overview.md#capacity-limited) |
| DS-series                  | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Dv1 and Dsv1-series        | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Dv2 and Dsv2-series        | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Av2 and Amv2-series        | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |

For a list of general purpose sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired general purpose sizes](./retired-sizes-list.md#general-purpose-retired-sizes).

## Compute optimized previous-gen sizes

|Series name                | Status                  | Migration guide   |
|---------------------------|-------------------------|-------------------|
| F-series                  | [Capacity limited](./retirement-overview.md#next-gen-available) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Fs-series                 | [Capacity limited](./retirement-overview.md#next-gen-available) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Fsv2-series               | [Capacity limited](./retirement-overview.md#next-gen-available) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |

For a list of compute optimized sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired compute optimized sizes](./retired-sizes-list.md#compute-optimized-retired-sizes).

## Memory optimized previous-gen sizes

|Series name                | Replacement series      |Migration guide |
|---------------------------|-------------------------|----------------|
| Ev3 and Esv3-series       | [Next-gen available](./retirement-overview.md#next-gen-available)  |                |
| Ev4 and Esv4-series       | [Next-gen available](./retirement-overview.md#next-gen-available)  |                |
| Eav4 and Easv4-series     | [Next-gen available](./retirement-overview.md#next-gen-available)  |                |
| Edv4 and Edsv4-series     | [Next-gen available](./retirement-overview.md#next-gen-available)  |                |
| GS-series                 | [Capacity limited](./retirement-overview.md#capacity-limited)  | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) | 
| G-series                  | [Capacity limited](./retirement-overview.md#capacity-limited)  | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) | 
| Memory-optimized D-series | [Capacity limited](./retirement-overview.md#capacity-limited)  | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |   
| Memory-optimized DS-series| [Capacity limited](./retirement-overview.md#capacity-limited)  | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) | 

For a list of memory optimized sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired memory optimized sizes](./retired-sizes-list.md#memory-optimized-retired-sizes).

## Storage optimized previous-gen sizes

|Series name                | Replacement series   | Migration guide|
|---------------------------|----------------------|----------------|
| Lsv1-series               | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) | 
| Lsv2-series               | [Capacity limited](./retirement-overview.md#capacity-limited) | [Retirement Announced](https://learn.microsoft.com/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |  

For a list of storage optimized sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired storage optimized sizes](./retired-sizes-list.md#storage-optimized-retired-sizes).

## GPU accelerated previous-gen sizes

Currently there are no previous-gen or capacity limited GPU accelerated sizes.

For a list of GPU accelerated sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired GPU accelerated sizes](./retired-sizes-list.md#gpu-accelerated-retired-sizes).

## FPGA accelerated previous-gen sizes

Currently there are no previous-gen or capacity limited FPGA accelerated sizes.

For a list of FPGA accelerated sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired fpga accelerated sizes](./retired-sizes-list.md#fpga-accelerated-retired-sizes).

## HPC previous-gen sizes

Currently there are no previous-gen or capacity limited HPC sizes.

For a list of HPC sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired HPC sizes](./retired-sizes-list.md#hpc-retired-sizes).

## ADH previous-gen sizes

Currently there are no previous-gen or capacity limited ADH sizes.

For a list of ADH sizes listed as "retired" and "announced for retirement" (sizes that are no longer available or soon to be unavailable for use), see [retired ADH sizes](./retired-sizes-list.md#adh-retired-sizes).

## Next steps
- For a list of retired sizes, see [Retired Azure VM sizes](./retired-sizes-list.md).
- For more information on VM sizes, see [Sizes for virtual machines in Azure](../sizes.md).
