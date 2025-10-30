---
title: General Purpose Sizes Migration Guide
description: Migration guide for general purpose VM size series
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 10/15/2025
ms.author: wwilliams
ms.reviewer: mattmcinnes
---

# General Purpose Sizes Migration Guide

This migration guide is designed for users of Azure General Purpose virtual machines (VMs), which are scheduled for retirement. To ensure minimal disruption and to continue optimizing cost and performance, this guide helps you transition to the latest series VMs.

This document covers:

 - Recommended replacement VM series
 - Detailed migration steps
 - Common questions and guidance on handling RIs.

By migrating to newer VM series, you gain access to improved price-performance ratios, broader regional availability, and the latest hardware capabilities.

## Recommended Replacement VM Series

|Current VM Family | Target VM Family| Differences in Specification in Target VM*| 
|--|--|--|
| D<br>Ds<br>Dv2<br>Dsv2 | Dsv5/Ddsv5/Dasv5/Dadsv5<br>Dasv6/Dadsv6/Dalsv6/Daldsv6<br>Dsv6/Ddsv6/Dlsv6/Dldsv6|  Local Storage: Supported - SCSI<br>Local Storage Throughput: 9000 IOPS / 125 MBps<br>Remote Storage Throughput: 3750 IOPS / 82 MBps<br> Disk Controller Type: SCSI|
| Ls | Lasv3<br>Lasv4 | Local Storage: Supported - NVMe<br>Remote Storage Throughput: 12800 IOPS / 200 MBps <br>Disk Controller Type: SCSI |
| Av2<br>Amv2 | Bsv2/Bpsv2/Basv2<br>Dsv5/Ddv5/Dasv5/Dpsv5<br>Esv5/Edv5/Easv5/Epsv5<br>Dsv6/Ddsv6/Dasv6/Dpsv6<br>Esv6/Edsv6/Easv6/Epsv6 | Local Storage: Not Supported<br>Remote Storage Throughput: 3750 IOPS / 85 MBps<br>Disk Controller Type: SCSI|
| B | Bsv2/Bpsv2/Basv2<br>Dlsv5/Dldsv5/Dalsv5/Daldsv5<br>Dlsv6/Dldsv6/Dalsv6/Daldsv6 | Local Storage: Not Supported<br>Remote Storage Throughput: 3750 IOPS / 85 MBps<br>Disk Controller Type: SCSI|
| F<br>Fs<br>Fsv2 | Dlsv6/Dldsv6<br>Falsv6<br>Dldsv5/Dlsv5/Dsv5/Ddsv5| Local Storage: Not Supported<br>Remote Storage Throughput: 4167 IOPS / 124 MBps<br>Disk Controller Type: NVMe|
| G<br>Gs | Lsv3/Lasv3<br>Lsv4/Lasv4| Local Storage: NVMe<br>Remote Storage Throughput: 12800 IOPS / 200 MBps<br>Disk Controller Type: SCSI|
| Lsv2 | Lasv3<br>Lasv4| Local Storage: NVMe<br>Remote Storage Throughput: 12800 IOPS / 200 MBps<br>Disk Controller Type: SCSI|

*Refers to the lowest VM size in the given target VM Family. For actual VM specifications, please refer to the VM product sizes page.

> [!IMPORTANT]  
> The following SKUs aren't available in the Sovereign clouds:
> Bsv2, Bpsv2, Basv2

For optimal performance and experience, we generally recommend using the newer v5 and v6 VM series. This ensures you have access to the latest features such as Premium Storage, Accelerated Networking, and Nested Virtualization. While the v6 VM series is preferred, there are certain scenarios where you might want to consider the v5 or even the v4 VM series. Here are some reasons why:
 - v6 VMs require [enabling NVMe](/azure/virtual-machines/nvme-overview) which means that you must have a [supported OS](/azure/virtual-machines/enable-nvme-interface).
 - v6 VMs support [Generation 2 VMs only](/azure/virtual-machines/generation-2).
 - v6 VMs require MANA ([Microsoft Azure Network Adapter](/azure/virtual-network/accelerated-networking-mana-overview)) and a MANA supported operating system.
 - v6 VMs may not have available capacity in the regions and zones you need.
 
Note that Lsv4 and Lasv4 series are the latest generation L-series VMs.

Use the [Azure VM size documentation](/azure/virtual-machines/sizes) to help identify suitable VM sizes.

## Migration Steps

#### Optional: For Reserved Instance (RI) customers only

- Review your current reservations using the [Azure Reservation Management](/azure/cost-management-billing/reservations/manage-reserved-vm-instance) page.
- If applicable, exchange existing reservations for newer VM series or trade in your reservations for an **Azure Savings Plan for compute**.

####  Identify the Target VM Size

- Evaluate your current VM's workload and performance requirements.
- Select a comparable size from the above table that meets your CPU, memory, and storage needs.

#### Check and Request Quota Increases

- Before resizing, verify that your subscription has sufficient quota for the target v6 VM series.
- Request more quota through the [Azure portal](/azure/azure-portal/supportability/per-vm-quota-requests) if needed.

#### Resize the Virtual Machine

You can resize your VM through the Azure portal, Azure CLI, or PowerShell. Follow these steps:
 1. **Stop (deallocate) the VM**.
 2. **Resize** the VM to your selected v6 series.
 3. **Start the VM** after resizing.

Refer to the full [Azure VM resizing guide](/azure/virtual-machines/sizes/resize-vm?tabs=portal) for more detailed instructions.

## FAQ
#### Q: Which Sizes Are Being Retired?
To review retired sizes, see [retired Azure VM sizes](https://learn.microsoft.com/azure/virtual-machines/sizes/retirement/retired-sizes-list).

#### Q: Why Should I Migrate?

If you are actively running any sizes listed in this article, these VMs are set to retire in 2028. Migration is mandatory to avoid unexpected shutdown. Additionally, migration yields the following benefits: 

 - **Performance**: Newer VM series offer better price-to-performance ratios.
 - **Regional Availability**: The v5 and v6 series has broader regional support across Azure data centers.
 - **Future-proofing**: Migrate ahead of the retirement schedule to avoid disruption.

#### Q: I am on pay-as-you-go (PayGo) or Savings Plan Pricing. Is There a Concern with Migration?

No. If you’re using PayGo or a savings plan, migrating to a newer VM type won't disrupt your current billing. The migration process remains seamless with no changes required in your subscription or payment plan.

#### Q: I'm on Reserved Instances (RIs) with an Older VM. How Do I Handle Migration?

If you have active Reserved Instances for D, Dv2, Dsv2, or L-series VMs, follow these steps:

Step 1: Review Current Reservations

 - Check your active RIs in the [Azure portal](/azure/cost-management-billing/reservations/manage-reserved-vm-instance).

Identify which RIs are expiring or will be affected by the VM retirement.

Step 2: Migrate and Manage Your RIs

Depending on your business needs, consider these options:

1. **Exchange Existing Reservations**:

   - Swap current RIs for a new VM series without any penalties.

   - Refer to the [RI Exchange Guide](/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations)

1. **Trade-In for Savings Plan**:

   - Convert your existing RIs into an **Azure Savings Plan for compute**.

   - This offers flexibility across VM families and regions.

   - Follow the [Azure RI Trade-In Tutorial](/azure/cost-management-billing/savings-plan/reservation-trade-in).

1. **Purchase New RIs**:

   - Buy new reservations that align with your new v6 VM series.

   - Consider shorter terms (1-year) for flexibility.
