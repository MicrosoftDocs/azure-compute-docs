---
title: Retired VM Sizes Migration Guide
description: Migration guide for retired VM size series
author: iamwilliew
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 01/30/2026
ms.author: wwilliams
ms.reviewer: mattmcinnes
ai-usage: ai-assisted
---

# Retired VM Sizes Migration Guide

This migration guide is designed for users of Azure virtual machines (VMs) scheduled for retirement. This guide helps you transition to the latest VM series, helping you minimize disruptions while optimizing cost and performance. The guide covers General Purpose, Storage Optimized, and other VM series. It also surfaces critical information for HPC HBv2-series VMs undergoing retirement; specialized workload validation is recommended when migrating HPC workloads.

This guide covers:

 - Recommended replacement VM series
 - Detailed migration steps
 - Common questions and guidance on handling RIs.

By migrating to newer VM series, you gain access to improved price-performance ratios, broader regional availability, and the latest hardware capabilities.

## Recommended Replacement VM Series

|Current VM Series | Target VM Series| Differences in Specification in Target VM*| 
|--|--|--|
| D<br>Ds<br>Dv2<br>Dsv2 | Dsv5/Ddsv5/Dasv5/Dadsv5<br>Dasv6/Dadsv6/Dsv6/Ddsv6<br>Dasv7/Dadsv7<br>Esv6/Edsv6/Easv6/Eadsv6<br>Easv7/Eadsv7| D/Ev5 disk controller type: SCSI <br> D/Ev6, D/Ev7 disk controller type: NVMe<br>Local Storage Throughput: 9000 IOPS / 125 MBps<br>Remote Storage Throughput: 3750 IOPS / 82 MBps|
| Ls | Lsv3/Lasv3<br>Lsv4/Lasv4 | Local Storage: Supported - NVMe<br>Remote Storage Throughput: 12800 IOPS / 200 MBps <br>Disk Controller Type: SCSI and NVMe |
| Av2<br>Amv2 | Bsv2/Basv2<br>Dsv5/Ddv5/Dasv5<br>Esv5/Edv5/Easv5<br>Dsv6/Ddsv6/Dasv6<br>Esv6/Edsv6/Easv6 | B/Bav2, D/Ev5 disk controller type: SCSI <br> D/Ev6 disk controller type: NVMe<br>Remote Storage Throughput: 3750 IOPS / 85 MBps
| Bv1 | Bsv2/Basv2<br>Dlsv5/Dldsv5/Dalsv5/Daldsv5<br>Dlsv6/Dldsv6/Dalsv6/Daldsv6 | B/Bav2, D/Ev5 disk controller type: SCSI <br> D/Ev6 disk controller type: NVMe<br>Remote Storage Throughput: 3750 IOPS / 85 MBps<br>Disk Controller Type: SCSI|
| F<br>Fs<br>Fsv2 | Dlsv6/Dldsv6/Dalsv6/Daldsv6<br>Falsv6<br>Dldsv5/Dlsv5/Dsv5/Ddsv5| D/Ev5 disk controller type: SCSI <br> D/Ev6 disk controller type: NVMe <br> Remote Storage Throughput: 4167 IOPS / 124 MBps|
| G<br>Gs | Lsv3/Lasv3<br>Lsv4/Lasv4| Lv3/Lv4 controller type: SCSI and NVMe <br>Remote Storage Throughput: 12800 IOPS / 200 MBps|
| Lsv2 | Lsv3/Lasv3<br>Lasv4/Lasv4| Local Storage: NVMe<br>Remote Storage Throughput: 12800 IOPS / 200 MBps<br>Disk Controller Type: SCSI and NVMe|
| HBv2 | HBv5<br>HX<br>HBv4<br>HBv3 | Transition to newer AMD EPYC generations (Milan/Genoa/Zen4) and newer InfiniBand fabrics (HDR on HBv3 → NDR on HBv4/HBv5). Validate MPI workload performance, memory bandwidth requirements, and RDMA compatibility on the target series. |
| NP-series<br>(Standard_NP10s<br>Standard_NP20s<br>Standard_NP40s) | [NDv2](../../sizes/gpu-accelerated/ndv2-series.md)<br>[NCads_H100_v5](../../sizes/gpu-accelerated/ncadsh100v5-series.md)<br>[NCasT4_v3](../../sizes/gpu-accelerated/ncast4v3-series.md) | GPU type: NVIDIA V100 (NDv2), H100 (NCads_H100_v5), or T4 (NCasT4_v3) vs. AMD Xilinx Alveo U250 FPGA<br>NVLink interconnect: Available on NDv2; not applicable on FPGA-based NP-series<br>Disk controller: NVMe/SCSI (GPU families) vs. SCSI (NP-series)<br>Note: Workloads must be ported from FPGA-based acceleration (XRT/Vitis) to CUDA/GPU-based frameworks. |

*Refers to the smallest VM size in the given target VM series. Full VM specifications are available on each target VM series' product sizes page.

*Refers to the smallest VM size in the given target VM series. Full VM specifications are available on each target VM series' product sizes page.

> [!IMPORTANT]  
> The following SKUs aren't available in the Sovereign clouds:
> Bsv2, Bpsv2, Basv2

## Recommended Replacement Isolated VM Sizes

|Current VM Size | Target VM Sizes| Differences in Specification in Target VM*| 
| --- | --- | --- |
| Standard_E64i_v3<br>Standard_E64is_v3| Standard_E192is_v6<br>Standard_E192ids_v6<br>Standard_E104i_v5<br>Standard_E104id_v5<br>Standard_E104is_v5<br>Standard_E104ids_v5<br>Standard_E80is_v4<br>Standard_E80ids_v4|  Local Storage: Supported - NVMe<br>Local Storage Throughput: 37,500 IOPS / 180 MBps<br>Remote Storage Throughput: 3,750 IOPS / 106 MBps<br> Disk Controller Type: NVMe|

*Refers to the smallest VM size in the given target VM series. Full VM specifications are available on each target VM series' product sizes page.

For optimal performance and experience, we recommend using the newer v5 and v6 VM series. This ensures you have access to the latest features such as Premium Storage, Accelerated Networking, and Nested Virtualization. While the v6 VM series is preferred, there are certain scenarios where you might want to consider the v5 or even the v4 VM series. Here are some reasons why:
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
- **HBv2 customers**: 1-year and 3-year HBv2 Reserved Instance purchases ended on April 2, 2026. If you have active HBv2 RIs, consider exchanging them for supported HPC series RIs (such as HBv3, HBv4, HBv5, or HX) or trading them in for an Azure Savings Plan for compute before the retirement date of May 31, 2027.
- **NP-series customers**: 1-year and 3-year NP-series Reserved Instance purchases ended on April 2, 2026. If you have active NP-series RIs, consider exchanging them for supported GPU series RIs or trading them in for an Azure Savings Plan for compute before the retirement date of May 31, 2027.
- For customers using Reserved Instances (RIs) on VM series such as
One-year and three-year RIs for the VM series Dv3, Dsv3, Ev3, and Esv3.
One-year RIs for the VM series Av2, Amv2, Bv1, D, Ds, Dv2, Dsv2, F, Fs, Fsv2, G, Gs, Ls, and Lsv2.
Existing RIs will remain valid through the end of their original term. However, once an RI expires after July 1, 2026, it cannot be purchased or renewed for these VM series. At that point, workloads will transition to pay as you go pricing unless another cost optimization option is selected. Customers are encouraged to either transition to Azure Savings Plan for compute or migrate workloads to newer VM generations, which continue to support both Reserved Instances and Savings Plans. Customers should plan their migration ahead of RI expiration to avoid unintended cost increases.


####  Identify the Target VM Size

- Evaluate your current VM's workload and performance requirements.
- Select a comparable size from the above table that meets your CPU, memory, and storage needs.

#### GPU workload migration (NP-series customers)

NP-series customers should validate their workload GPU requirements (CUDA cores, memory bandwidth, and interconnect needs) before selecting a target VM family. Consider the following recommended alternatives and their key characteristics:

- **[NDv2 VMs](../../sizes/gpu-accelerated/ndv2-series.md)** – Best for training and large-scale AI/HPC workloads. Features NVIDIA V100 GPUs with NVLink for GPU-to-GPU communication and high memory bandwidth.
- **[NCads_H100_v5 VMs](../../sizes/gpu-accelerated/ncadsh100v5-series.md)** – Best for modern AI training and batch inference requiring the latest GPU generation. Features NVIDIA H100 GPUs.
- **[NCasT4_v3 VMs](../../sizes/gpu-accelerated/ncast4v3-series.md)** – Best for inference, interactive graphics, and cost-sensitive workloads. Features NVIDIA T4 GPUs.

> [!IMPORTANT]
> NP-series VMs use FPGA-based acceleration (Xilinx/AMD Alveo U250). Migrating to GPU-based VM families requires porting your workloads from FPGA frameworks (such as Vitis/XRT) to GPU-based frameworks (such as CUDA). Perform thorough test validation before resizing production workloads.

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
#### Q: Which sizes are being retired?
To review retired sizes, see [retired Azure VM sizes](/azure/virtual-machines/sizes/retirement/retired-sizes-list). View retired isolated sizes at [Isolation for VMs in Azure](/azure/virtual-machines/isolation).

> [!NOTE]
> HPC HBv2-series VMs are also retiring. See the HBv2 row in the table below for the applicable dates.

| VM Series | 3 YR RI expiration date | 1 YR RI expiration date | Retirement Date|
|----|----|----|----|
| Av2       | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| Amv2      | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| Bv1       | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| D         | 05/01/2025 | 07/01/2026 | 05/01/2028 |
| Ds        | 05/01/2025 | 07/01/2026 | 05/01/2028 |
| Dsv2      | 05/01/2025 | 07/01/2026 | 05/01/2028 |
| Dsv3      | 07/01/2026 | 07/01/2026 | Product active |
| Dv2       | 05/01/2025 | 07/01/2026 | 05/01/2028 |
| Dv3       | 07/01/2026 | 07/01/2026 | Product active |
| Esv3      | 07/01/2026 | 07/01/2026 | Product active |
| Ev3       | 07/01/2026 | 07/01/2026 | Product active |
| F         | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| Fs        | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| Fsv2      | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| G         | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| Gs        | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| HBv2      | 04/02/2026 | 04/02/2026 | 05/31/2027 |
| Ls        | 05/01/2025 | 07/01/2026 | 05/01/2028 |
| Lsv2      | 11/15/2025 | 07/01/2026 | 11/15/2028 |
| NP-series | 04/02/2026 | 04/02/2026 | 05/31/2027 |


> [!NOTE]
> Purchases of 1-year and 3-year Azure Reserved VM Instances for NP-series ended on 04/02/2026.


#### Q: Why should I migrate my VM?

Migration is mandatory to avoid unexpected shutdown. Additionally, migration yields the following benefits: 

 - **Performance**: Newer VM series offer better price-to-performance ratios.
 - **Regional Availability**: The v5 and v6 series has broader regional support across Azure data centers.
 - **Future-proofing**: Migrate ahead of the retirement schedule to avoid disruption.

#### Q:How are Reserved Instance (RI) purchases and renewals changing for Dv3, Dsv3, Ev3, and Esv3?
 -Dv3, Dsv3, Ev3, and Esv3 are not being retired until further notice. However, three- and one-year reserved instances discounts will no longer be available for new purchases or renewals after July 1, 2026. Azure savings plan is the primary recommendation if you plan to continue using these VM series. If you are looking to migrate from Dv3/Dsv3, recommended VM series include Dsv5, Ddsv5, Dasv5, Dadsv5, Dsv6, Ddsv6, Dasv6, and Dadsv6. For Ev3/Esv3, recommended replacements include Esv5, Edsv5, Easv5, Eadsv5, Esv6, Edsv6, Easv6, and Eadsv6. Customers should refer to the migration steps above for guidance

#### Q: What will happen to my VM if I do not resize my VM to a target size within the retirement timeline?

After retirement, VMs using this size will be deallocated and stop incurring charges. The size is no longer supported or covered by an SLA; in‑memory and temporary disk data is lost, but managed disk data is preserved. To resume service, you may resize to a supported size and restart the VM.

Specifically for HBv2-series: after May 31, 2027, HBv2-series VMs (Standard_HB120rs_v2 and derived sizes) will be automatically set to a deallocated state. After that date, HBv2 VMs will stop working, lose SLA and support, and stop incurring billing charges.

Specifically for NP-series: after May 31, 2027, NP-series VMs (Standard_NP10s, Standard_NP20s, Standard_NP40s) will be automatically set to a deallocated state. They'll stop working, lose SLA and support, and stop incurring billing charges. Managed disk data is preserved, but workloads won't resume until you resize to a supported VM size.

#### Q: Can I recover my VM after it has been deallocated?
Yes, you can resize and restart your deallocated VM following the [Azure VM resizing guide](/azure/virtual-machines/sizes/resize-vm?tabs=portal).

#### Q: Will VM migration disrupt pay-as-you-go or Savings Plan Pricing billing?
No. If you’re using pay-as-you-go or a savings plan, migrating to a newer VM type won't disrupt your current billing. The migration process remains seamless with no changes required in your subscription or payment plan.

#### Q: How can I migrate my VM if I am on Reserved Instances (RIs) with a retired VM?
If you have active Reserved Instances for any of listed series in FAQ chart, follow these steps:

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
