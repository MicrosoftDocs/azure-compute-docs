---
title: VM lifecycle overview
description: Learn about the Azure VM lifecycle stages (Current, Extended, End of Life, and Retired) and how to plan modernization to newer VM sizes.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: overview
ms.date: 09/24/2026
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud infrastructure administrator, I want to understand the lifecycle stages of virtual machine size series, so that I can plan deployments and modernize workloads to newer VM sizes before retirement.
---

# VM lifecycle overview

Azure virtual machine sizes are the amount of resources allocated to a virtual machine in the cloud. These resources are a portion of the physical server’s hardware capabilities. A *size series* is a collection of all sizes that are available within a single physical server’s hardware. As a size series' physical hardware ages and newer components are released, Microsoft stops deploying more of previously established series' hardware. Once users modernize their workloads off that hardware or the hardware becomes sufficiently outdated, it's retired to make room for new infrastructure.

![A diagram showing a greyed out Azure VM icon with an arrow pointing to a new sparkling Azure VM icon.](../media/size-retirement-new-vm.png "Moving from old to new VM sizes")

Every Azure VM size series moves through four lifecycle stages: *Current*, *Extended*, *End of Life*, and *Retired*. Each stage tells you what to expect from a size series and what action to take. Use the lifecycle stage of your VM sizes to plan new deployments, optimize existing workloads, and schedule modernization before retirement.

## Lifecycle stages

| Stage | Recommended action | What to expect |
|---|---|---|
| [Current](#current) | Adopt and expand | Recommended for new deployments and ongoing innovation. |
| [Extended](#extended) | Continue and evaluate | Run existing workloads while you evaluate newer options. |
| [End of Life](#end-of-life) | Plan modernization | Retirement is announced. Prepare and complete modernization plans before the retirement date. |
| [Retired](#retired) | Modernization completed | No longer available to create, resize into, or run. |

The VM generations in each stage vary by VM family. For example, for general purpose sizes, v6 and v7 series are Current, v4 and v5 series are Extended, and v1 through v3 series with an announced retirement are End of Life. For other families, check the family page for the lifecycle stage of each series.

Current and Extended sizes are both considered *modern* sizes because they're fully supported. Current sizes are the newest offering in each family. When you modernize, you can move to either stage.

### Current

Size series in the *Current* stage are the newest series in their VM family. Deploy new workloads, expand existing deployments, and optimize your Azure investments by using Current sizes.

To modernize general purpose workloads to Current sizes, see [Modernize to the v6 and v7 VM series](../../migration/sizes/sizes-v6-v7-migration-overview.md).

### Extended

Size series in the *Extended* stage are fully supported. Continue running existing workloads on Extended sizes while you evaluate Current sizes for new deployments and future growth.

If you need to modernize End of Life workloads with the least disruption, Extended sizes are a practical first step. For D-family and E-family workloads, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md).

### End of Life

Size series in the *End of Life* stage have an announced retirement. These series remain available until their retirement date, but VMs in these series have restrictions when you deploy through new subscriptions. Plan and complete your modernization to Current or Extended sizes before the retirement date.

For example, the Dv3, Dsv3, Ev3, and Esv3 series are in the End of Life stage and retire on November 15, 2029. After that date, you can't create, resize into, run, or purchase these sizes.

For a list of End of Life size series and their modernization guides, see [End of Life Azure VM size series](./end-of-life-sizes-list.md).

### Retired

You **can't use** size series in the *Retired* stage. At the retirement date, any remaining VMs in a retired series are deallocated, stop working, stop incurring charges, and no longer have an SLA or support.

For a list of retired size series and their retirement dates, see [Retired and retiring VM size series](./retirements-and-capacity-restrictions.md#retired-and-retiring-vm-size-series). For this year's retirement announcements, see [Azure Virtual Machine size retirements in 2026](/lifecycle/announcements/azure/virtual-machine-sizes-retirement-list-2026).

## Capacity restrictions

Capacity restrictions are separate from lifecycle stages. Some older size series have capacity growth restrictions, which means Azure doesn't guarantee capacity for new or scaled-out deployments in those series. A capacity restriction isn't a retirement announcement. A series can have capacity restrictions and a separate retirement date. For example, the Dv3, Dsv3, Ev3, and Esv3 series have capacity growth restrictions that began in July 2026, and they retire on November 15, 2029.

For affected series and recommended alternatives, see [End of Life VM size series capacity growth restrictions](./retirements-and-capacity-restrictions.md).

## Modernize to newer sizes

Modernize workloads to Current or Extended sizes. Use Current sizes for new deployments.

For the smoothest modernization of existing workloads, see [Modernize to the v5 VM series](./sizes-v5-modernization-overview.md). To access the latest features and performance, see [Modernize to the v6 and v7 VM series](../../migration/sizes/sizes-v6-v7-migration-overview.md).

Some size series have specific modernization instructions because of their unique hardware or software features. For modernization guides, see [Retired and retiring VM size series](./retirements-and-capacity-restrictions.md#retired-and-retiring-vm-size-series) and [End of Life Azure VM size series](./end-of-life-sizes-list.md).

For sizes without specific instructions, you can [resize your VM](../resize-vm.md) to a newer size by using the Azure portal, Azure PowerShell, Azure CLI, or Terraform. Make sure that the new size supports all features that your workload requires.

## Next steps
- For more information on VM sizes, see [Sizes for virtual machines in Azure](../overview.md).
- For a list of retired sizes, see [Retired and retiring VM size series](./retirements-and-capacity-restrictions.md#retired-and-retiring-vm-size-series).
- For a list of End of Life sizes, see [End of Life Azure VM size series](./end-of-life-sizes-list.md).
