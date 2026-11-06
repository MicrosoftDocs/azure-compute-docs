---
title: End of Life Azure VM size series
description: A list of Azure VM size series in the End of Life lifecycle stage and their modernization guides.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 09/25/2026
ms.author: mattmcinnes
ms.reviewer: iamwilliew
# Customer intent: As a cloud architect, I want to see which Azure VM size series are in the End of Life stage, so that I can plan modernization to Current or Extended VM sizes before retirement.
---

# End of Life Azure VM size series

This article lists the size series in the *End of Life* lifecycle stage. End of Life series have an announced retirement. For series that need them, *modernization guides* help you move to replacement sizes.

> [!NOTE]
> End of Life series **aren't retired yet** and can still be used until their retirement date. However, series with an announced retirement have restrictions when you deploy through new subscriptions. To view retired size series, see [Retired and retiring VM size series](./retirements-and-capacity-restrictions.md#retired-and-retiring-vm-size-series).

## What are End of Life size series?
End of Life virtual machine size series run on older hardware and have an announced retirement date. While they're still supported until retirement, plan and complete your modernization to Current or Extended sizes. Use Current sizes for new deployments.

To learn more about the Current, Extended, End of Life, and Retired lifecycle stages, see the [VM lifecycle overview](./lifecycle-overview.md).

## General purpose End of Life sizes

|Series name                 | Modernization guide   |
|----------------------------|--------------------|
| B-series (V1)              | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Standard D-series          | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| DS-series                  | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dv1 and Dsv1-series        | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dv2 and Dsv2-series        | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Dv3 and Dsv3-series        | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Av2 and Amv2-series        | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| DCsv3 and DCdsv3-series    | [Modernization guide](../retirement/dcsv3-series-retirement.md) |

For general purpose sizes that are retired or have an announced retirement date, see [retired general purpose sizes](./retirements-and-capacity-restrictions.md#general-purpose-retired-sizes).

## Compute optimized End of Life sizes

|Series name                | Modernization guide   |
|---------------------------|--------------------|
| F-series                  | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Fs-series                 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Fsv2-series               | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |

For compute optimized sizes that are retired or have an announced retirement date, see [retired compute optimized sizes](./retirements-and-capacity-restrictions.md#compute-optimized-retired-sizes).

## Memory optimized End of Life sizes

|Series name                | Modernization guide |
|---------------------------|------------------|
| Ev3 and Esv3-series       | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| GS-series                 | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| G-series                  | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Memory-optimized D-series | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Memory-optimized DS-series| [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Msv2 and Mdsv2 isolated sizes | [Modernization guide](./retirement/msv2-mdsv2-retirement.md) |

For memory optimized sizes that are retired or have an announced retirement date, see [retired memory optimized sizes](./retirements-and-capacity-restrictions.md#memory-optimized-retired-sizes).

## Storage optimized End of Life sizes

|Series name                | Modernization guide |
|---------------------------|------------------|
| Lsv1-series               | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |
| Lsv2-series               | [Modernization guide](./retirement/retired-sizes-modernization-guide.md) |

For storage optimized sizes that are retired or have an announced retirement date, see [retired storage optimized sizes](./retirements-and-capacity-restrictions.md#storage-optimized-retired-sizes).

## GPU accelerated End of Life sizes

|Series name                | Modernization guide |
|---------------------------|------------------|
| NVv3-series               | [Modernization guide](./retirement/nvv3-series-retirement.md) |
| NVv4-series               | [Modernization guide](./retirement/nvv4-retirement.md) |

For GPU accelerated sizes that are retired or have an announced retirement date, see [retired GPU accelerated sizes](./retirements-and-capacity-restrictions.md#gpu-accelerated-retired-sizes).

## FPGA accelerated End of Life sizes

|Series name                | Modernization guide |
|---------------------------|------------------|
| NP-series                 | [Modernization guide](./retirement/np-series-retirement.md) |

For FPGA accelerated sizes that are retired or have an announced retirement date, see [retired FPGA accelerated sizes](./retirements-and-capacity-restrictions.md#fpga-accelerated-retired-sizes).

## HPC End of Life sizes

|Series name                | Modernization guide |
|---------------------------|------------------|
| HC-series                 | [Modernization guide](./retirement/hc-series-retirement.md) |
| HBv2-series               | [Modernization guide](./retirement/hbv2-series-retirement.md) |

For HPC sizes that are retired or have an announced retirement date, see [retired HPC sizes](./retirements-and-capacity-restrictions.md#hpc-retired-sizes).

## ADH End of Life sizes

For ADH sizes that are retired or have an announced retirement date, see [retired ADH sizes](./retirements-and-capacity-restrictions.md#adh-retired-sizes).

## Next steps
- To modernize existing workloads, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md) or [Modernize to the v6 and v7 VM series](./sizes-v6-v7-modernization-overview.md).
- For a list of retired sizes, see [Retired and retiring VM size series](./retirements-and-capacity-restrictions.md#retired-and-retiring-vm-size-series).
- For more information on VM sizes, see [Sizes for virtual machines in Azure](../overview.md).
