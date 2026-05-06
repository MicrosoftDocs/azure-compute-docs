---
title: NP family VM size series
description: List of sizes in the NP family.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 04/13/2026
ms.author: mattmcinnes
ai-usage: ai-assisted
# Customer intent: "As a cloud architect, I want to explore the NP family VM sizes, so that I can select the appropriate FPGA-accelerated VMs for my workloads."
---

# 'NP' family FPGA-accelerated VM size series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

> [!IMPORTANT]
> NP-series VM sizes (Standard_NP10s, Standard_NP20s, Standard_NP40s) are scheduled for retirement on **May 31, 2027**. After this date, remaining NP-series VMs are deallocated, stop working, stop incurring charges, and no longer have SLA or support. Managed disk data is preserved.
>
> Purchases of 1-year and 3-year Azure Reserved VM Instances for NP-series ended on **April 2, 2026**. Existing reservations are honored until expiration, but no new NP-series reservations can be purchased. Avoid deploying new NP-series VMs and plan your migration before the retirement date.
>
> For migration guidance, see [NP-series virtual machine migration guidance](../retirement/np-series-retirement.md).

[!INCLUDE [np-family-summary](./includes/np-family-summary.md)]

## Workloads and use cases

[!INCLUDE [np-family-workloads](./includes/np-family-workloads.md)]

## Series in family

### NP-series
[!INCLUDE [np-series-summary](./includes/np-series-summary.md)]

> [!WARNING]
> The NP-series sizes (Standard_NP10s, Standard_NP20s, Standard_NP40s) are retiring on **May 31, 2027**. Avoid deploying new NP-series VMs. See the [Retirement and migration guidance](#retirement-and-migration-guidance) section below for recommended alternatives.

[View the full np-series page](../../np-series.md).

[!INCLUDE [np-series-specs](./includes/np-series-specs.md)]

### Previous-generation NP family series
For older sizes, see [previous generation sizes](../previous-gen-sizes-list.md#fpga-accelerated-previous-gen-sizes).

## Retirement and migration guidance

NP-series VMs are FPGA-accelerated (Xilinx/AMD Alveo U250) and are used for custom hardware acceleration scenarios such as ML inference, video transcoding, and database search. As these VMs retire on May 31, 2027, customers should migrate to alternative Azure GPU or accelerator VM families that best match their workload needs. Note that feature parity may differ from FPGA-based acceleration, and customers should validate performance and compatibility before finalizing a migration target.

Recommended alternatives:

- **[NDv2 VMs](../gpu-accelerated/ndv2-series.md)** – Designed for demanding GPU-accelerated AI, machine learning, simulation, and HPC workloads. Powered by NVIDIA V100 GPUs with NVLink interconnect.
- **[NCads_H100_v5 VMs](../gpu-accelerated/ncadsh100v5-series.md)** – Ideal for real-world Azure Applied AI training and batch inference workloads. Powered by NVIDIA H100 GPUs.
- **[NCasT4_v3 VMs](../gpu-accelerated/ncast4v3-series.md)** – Suitable for deploying AI inference services, interactive graphics, and visualization workloads. Powered by NVIDIA T4 GPUs.

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
