---
title: Constrained vCPU sizes 
description: Lists the Vm sizes that are capable of having a constrained vCPU count. Useful for per-core licensed database workloads.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 10/16/2025
ms.author: mattmcinnes
ms.reviewer: mimckitt, archatc
# Customer intent: "As a database administrator, I want to select virtual machine sizes with constrained vCPU counts, so that I can optimize licensing costs while maintaining necessary performance for my database workloads."
---

# Constrained vCPU sizes for database workloads

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

> [!TIP]
> Try the **[Virtual Machine selector tool](https://aka.ms/vm-selector)** to find other sizes that best fit your workload.

Some database workloads require high memory, storage, and I/O bandwidth, but don't benefit from a high number of cores. Products designed for these workloads are often licensed on a per-CPU-core basis. This licensing system means that a VM size with otherwise ideal specifications but an excessive vCPU count can cause a substantial increase in licensing costs. To help manage your costs, Azure offers predefined VM sizes with a lowered vCPU count to help to reduce the cost of software licensing, while maintaining the same memory, storage, and I/O bandwidth.

The original size's available vCPU count can be reduced to either one half or one quarter of the original size specification. These new VM sizes have a suffix that specifies the number of available vCPUs to make them easier for you to identify. Learn more about [VM size naming conventions](vm-naming-conventions.md).

## Example workload
The licensing fees charged for SQL Server are based on the available vCPU count. Third party products should count the available vCPUs and report it as the amount to be used and licensed. This constraint results in a 50% to 75% increase in the ratio of the VM specs to available (billable) vCPUs while maintaining the benefits of the original VM size's specifications. 

## Specification comparison
The Standard_E32s_v5 VM size comes with 32 vCPUs, 256 GiB of RAM, 32 disks, and 80,000 IOPs of I/O bandwidth. The pre-defined Standard_E32-16s_v5 and Standard_E32-8s_v5 VM sizes comes with 16 and 8 active vCPUs respectively, while maintaining the memory, storage, and I/O bandwidth specifications of the Standard_E32s_v5.

| Size type       | Size name              | Disabled vCPUs | Active vCPUs      | Memory, Storage, I/O           |
|-----------------|------------------------|----------------|-------------------|--------------------------------|
| Base (Original) | Standard_E32s_v5       | 0              | **32**            | 256 GiB, 32 Disks, 80,000 IOPs |
| Constrained     | Standard_E32-**16**s_v5| 16             | **16**            | 256 GiB, 32 Disks, 80,000 IOPs |
| Constrained     | Standard_E32-**8**s_v5 | 24             | **8**             | 256 GiB, 32 Disks, 80,000 IOPs |

> [!NOTE]
> Disabled vCPUs are not available for use by the VM. If your VM has multiple workloads assigned to it then it may require more vCPUs. If possible, relocate secondary tasks to another virtual machine to avoid increased licensing costs.

## Pricing
At this time, the VM pricing, remains the same as the original size. For more information, see [Azure VM sizes for more cost-effective database workloads](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/).

## List of available sizes with constrained vCPUs

#### [M-family sizes](#tab/family-M)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_M8-2ms         | 2    | M8ms       |
| Standard_M8-4ms         | 4    | M8ms       |
| Standard_M16-4ms        | 4    | M16ms      |
| Standard_M16-8ms        | 8    | M16ms      |
| Standard_M32-8ms        | 8    | M32ms      |
| Standard_M32-16ms       | 16   | M32ms      |
| Standard_M64-32ms       | 32   | M64ms      |
| Standard_M64-16ms       | 16   | M64ms      |
| Standard_M128-64ms      | 64   | M128ms     |
| Standard_M128-32ms      | 32   | M128ms     |
| Standard_M416-208s_v2   | 208  | M416s_v2   |
| Standard_M416-208ms_v2  | 208  | M416ms_v2  |
| Standard_M64-32bds_1_v3 | 32   | M64bds_1_v3|
| Standard_M96-48bds_2_v3 | 48   | M96bds_2_v3|
| Standard_M128-64bds_3_v3| 64   | M128bds_3_v3|
| Standard_M176-88bds_4_v3| 88   | M176bds_4_v3|
| Standard_M128-64bds_v3  | 64   | M128bds_v3 |
| Standard_M176-88bds_v3  | 88   | M176bds_v3 |
| Standard_M128-64bs_v3   | 64   | M128bs_v3  |
| Standard_M176-88bs_v3   | 88   | M176bs_v3  |



#### [E-family sizes](#tab/family-E)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_E4-2s_v3       | 2    | E4s_v3     |
| Standard_E8-4s_v3       | 4    | E8s_v3     |
| Standard_E8-2s_v3       | 2    | E8s_v3     |
| Standard_E16-8s_v3      | 8    | E16s_v3    |
| Standard_E16-4s_v3      | 4    | E16s_v3    |
| Standard_E32-16s_v3     | 16   | E32s_v3    |
| Standard_E32-8s_v3      | 8    | E32s_v3    |
| Standard_E64-32s_v3     | 32   | E64s_v3    |
| Standard_E64-16s_v3     | 16   | E64s_v3    |
| Standard_E4-2s_v4       | 2    | E4s_v4     |
| Standard_E8-4s_v4       | 4    | E8s_v4     |
| Standard_E8-2s_v4       | 2    | E8s_v4     |
| Standard_E16-8s_v4      | 8    | E16s_v4    |
| Standard_E16-4s_v4      | 4    | E16s_v4    |
| Standard_E32-16s_v4     | 16   | E32s_v4    |
| Standard_E32-8s_v4      | 8    | E32s_v4    |
| Standard_E64-32s_v4     | 32   | E64s_v4    |
| Standard_E64-16s_v4     | 16   | E64s_v4    |
| Standard_E4-2ds_v4      | 2    | E4ds_v4    |
| Standard_E8-4ds_v4      | 4    | E8ds_v4    |
| Standard_E8-2ds_v4      | 2    | E8ds_v4    |
| Standard_E16-8ds_v4     | 8    | E16ds_v4   |
| Standard_E16-4ds_v4     | 4    | E16ds_v4   |
| Standard_E32-16ds_v4    | 16   | E32ds_v4   |
| Standard_E32-8ds_v4     | 8    | E32ds_v4   |
| Standard_E64-32ds_v4    | 32   | E64ds_v4   |
| Standard_E64-16ds_v4    | 16   | E64ds_v4   |
| Standard_E4-2s_v5       | 2    | E4s_v5     |
| Standard_E8-4s_v5       | 4    | E8s_v5     |
| Standard_E8-2s_v5       | 2    | E8s_v5     |
| Standard_E16-8s_v5      | 8    | E16s_v5    |
| Standard_E16-4s_v5      | 4    | E16s_v5    |
| Standard_E32-16s_v5     | 16   | E32s_v5    |
| Standard_E32-8s_v5      | 8    | E32s_v5    |
| Standard_E64-32s_v5     | 32   | E64s_v5    |
| Standard_E64-16s_v5     | 16   | E64s_v5    |
| Standard_E96-48s_v5     | 48   | E96s_v5    |
| Standard_E96-24s_v5     | 24   | E96s_v5    |
| Standard_E4-2ds_v5      | 2    | E4ds_v5    |
| Standard_E8-4ds_v5      | 4    | E8ds_v5    |
| Standard_E8-2ds_v5      | 2    | E8ds_v5    |
| Standard_E16-8ds_v5     | 8    | E16ds_v5   |
| Standard_E16-4ds_v5     | 4    | E16ds_v5   |
| Standard_E32-16ds_v5    | 16   | E32ds_v5   |
| Standard_E32-8ds_v5     | 8    | E32ds_v5   |
| Standard_E64-32ds_v5    | 32   | E64ds_v5   |
| Standard_E64-16ds_v5    | 16   | E64ds_v5   |
| Standard_E96-48ds_v5    | 48   | E96ds_v5   |
| Standard_E96-24ds_v5    | 24   | E96ds_v5   |
| Standard_E4-2as_v4      | 2    | E4as_v4    |
| Standard_E8-4as_v4      | 4    | E8as_v4    |
| Standard_E8-2as_v4      | 2    | E8as_v4    |
| Standard_E16-8as_v4     | 8    | E16as_v4   |
| Standard_E16-4as_v4     | 4    | E16as_v4   |
| Standard_E32-16as_v4    | 16   | E32as_v4   |
| Standard_E32-8as_v4     | 8    | E32as_v4   |
| Standard_E64-32as_v4    | 32   | E64as_v4   |
| Standard_E64-16as_v4    | 16   | E64as_v4   |
| Standard_E96-48as_v4    | 48   | E96as_v4   |
| Standard_E96-24as_v4    | 24   | E96as_v4   |
| Standard_E4-2ads_v5     | 2    | E4ads_v5   |
| Standard_E8-4ads_v5     | 4    | E8ads_v5   |
| Standard_E8-2ads_v5     | 2    | E8ads_v5   |
| Standard_E16-8ads_v5    | 8    | E16ads_v5  |
| Standard_E16-4ads_v5    | 4    | E16ads_v5  |
| Standard_E32-16ads_v5   | 16   | E32ads_v5  |
| Standard_E32-8ads_v5    | 8    | E32ads_v5  |
| Standard_E64-32ads_v5   | 32   | E64ads_v5  |
| Standard_E64-16ads_v5   | 16   | E64ads_v5  |
| Standard_E96-48ads_v5   | 48   | E96ads_v5  |
| Standard_E96-24ads_v5   | 24   | E96ads_v5  |
| Standard_E4-2as_v5      |	2    | E4as_v5    |  
| Standard_E8-4as_v5      |	4    | E8as_v5    | 
| Standard_E8-2as_v5      |	2    | E8as_v5    |  
| Standard_E16-8as_v5     |	8    | E16as_v5   |
| Standard_E16-4as_v5     |	4    | E16as_v5   |
| Standard_E32-16as_v5    |	16   | E32as_v5   |
| Standard_E32-8as_v5     |	8    | E32as_v5   |
| Standard_E64-32as_v5    |	32   | E64as_v5   |
| Standard_E64-16as_v5    | 16   | E64as_v5   |
| Standard_E96-48as_v5    | 48   | E96as_v5   |
| Standard_E96-24as_v5    | 24   | E96as_v5   |
| Standard_E4-2s_v6	      | 4    | E4s_v6     |
| Standard_E8-2s_v6		    | 8   | E8s_v6     |
| Standard_E8-4s_v6	    | 8    | E8s_v6     |
| Standard_E16-4s_v6	  | 16    | E16s_v6     |
| Standard_E16-8s_v6	  | 16    | E16s_v6     |
| Standard_E32-8s_v6	  | 32    | E32s_v6     |
| Standard_E32-16s_v6	 | 32    | E32s_v6     |
| Standard_E64-16s_v6	 | 64    | E64s_v6     |
| Standard_E64-32s_v6	 | 64    | E64s_v6     |
| Standard_E96-24s_v6	 | 96    | E96s_v6     |
| Standard_E96-48s_v6	 | 96    | E96s_v6     |
| Standard_E128-32s_v6	 | 128    | E128s_v6     |
| Standard_E128-64s_v6	 | 128   | E128s_v6     |
| Standard_E4-2ds_v6	4 | 4  | E4ds_v6  |
| Standard_E8-2ds_v6	| 8 | E8ds_v6 |
| Standard_E8-4ds_v6	| 8 | E8ds_v6 |
| Standard_E16-4ds_v6	| 16 | E16ds_v6 |
| Standard_E16-8ds_v6	| 16 | E16ds_v6 |
| Standard_E32-8ds_v6	| 32 | E32ds_v6 |
| Standard_E32-16ds_v6 | 32 | E32ds_v6 |
| Standard_E64-16ds_v6	| 16 | E64ds_v6 |
| Standard_E64-32ds_v6	| 64 | E64ds_v6 |
| Standard_E96-24ds_v6	| 96 | E96ds_v6 |
| Standard_E96-48ds_v6	| 96 | E96ds_v6 |
| Standard_E128-32ds_v6	| 128 | E128ds_v6 |
| Standard_E128-64ds_v6	| 128 | E128ds_v6 |
| Standard_E4-2ads_v7	| 2  | E4ads_v7  |
| Standard_E8-2ads_v7	| 2 | E8ads_v7 |
| Standard_E8-4ads_v7	| 4 | E8ads_v7 |
| Standard_E16-4ads_v7	| 4 | E16ads_v7 |
| Standard_E16-8ads_v7	| 8 | E16ads_v7 |
| Standard_E32-8ads_v7	| 8 | E32ads_v7 |
| Standard_E32-16ads_v7 | 16 | E32ads_v7 |
| Standard_E64-16ads_v7	| 16 | E64ads_v7 |
| Standard_E64-32ads_v7	| 32 | E64ads_v7 |
| Standard_E96-24ads_v7	| 24 | E96ads_v7 |
| Standard_E96-48ads_v7	| 48 | E96ads_v7 |
| Standard_E128-32ads_v7	| 32 | E128ads_v7 |
| Standard_E128-64ads_v7	| 64 | E128ads_v7 |
| Standard_E4-2as_v7	| 2  | E4as_v7  |
| Standard_E8-2as_v7	| 2 | E8as_v7 |
| Standard_E8-4as_v7	| 4 | E8as_v7 |
| Standard_E16-4as_v7	| 4 | E16as_v7 |
| Standard_E16-8as_v7	| 8 | E16as_v7 |
| Standard_E32-8as_v7	| 8 | E32as_v7 |
| Standard_E32-16as_v7 | 16 | E32as_v7 |
| Standard_E64-16as_v7	| 16 | E64as_v7 |
| Standard_E64-32as_v7	| 32 | E64as_v7 |
| Standard_E96-24as_v7	| 24 | E96as_v7 |
| Standard_E96-48as_v7	| 48 | E96as_v7 |
| Standard_E128-32as_v7	| 32 | E128as_v7 |
| Standard_E128-64as_v7	| 64 | E128as_v7 |


#### [F-family sizes](#tab/family-F)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_F2-1amds_v7	| 1  | F2amds_v7  |
| Standard_F4-1amds_v7	| 1  | F4amds_v7  |
| Standard_F4-2amds_v7	| 2  | F4amds_v7  |
| Standard_F8-2amds_v7	| 2 | F8amds_v7 |
| Standard_F8-4amds_v7	| 4 | F8amds_v7 |
| Standard_F16-4amds_v7	| 4 | F16amds_v7 |
| Standard_F16-8amds_v7	| 8 | F16amds_v7 |
| Standard_F32-8amds_v7	| 8 | F32amds_v7 |
| Standard_F32-16amds_v7 | 16 | F32amds_v7 |
| Standard_F64-16amds_v7	| 16 | F64amds_v7 |
| Standard_F64-32amds_v7	| 32 | F64amds_v7 |
| Standard_F2-1ams_v7	| 1  | F2ams_v7  |
| Standard_F4-1ams_v7	| 1  | F4ams_v7  |
| Standard_F4-2ams_v7	| 2  | F4ams_v7  |
| Standard_F8-2ams_v7	| 2 | F8ams_v7 |
| Standard_F8-4ams_v7	| 4 | F8ams_v7 |
| Standard_F16-4ams_v7	| 4 | F16ams_v7 |
| Standard_F16-8ams_v7	| 8 | F16ams_v7 |
| Standard_F32-8ams_v7	| 8 | F32ams_v7 |
| Standard_F32-16ams_v7 | 16 | F32ams_v7 |
| Standard_F64-16ams_v7	| 16 | F64ams_v7 |
| Standard_F64-32ams_v7	| 32 | F64ams_v7 |



#### [FX-family sizes](#tab/family-fx)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_FX4-2mds_v2          | 2    | FX4mds_v2        |
| Standard_FX8-4mds_v2          | 4    | FX8mds_v2       |
| Standard_FX8-2mds_v2         | 2   | FX8mds_v2        |
| Standard_FX12-6mds_v2         | 6    | FX12mds_v2        |
| Standard_FX16-8mds_v2          | 8    | FX16mds_v2        |
| Standard_FX16-4mds_v2         | 4    | FX16mds_v2        |
| Standard_FX24-12mds_v2         | 12   | FX24mds_v2        |
| Standard_FX24-6mds_v2        | 6    | FX24mds_v2        |
| Standard_FX32-16mds_v2     | 16    | FX32mds_v2        |
| Standard_FX32-8mds_v2          | 8    | FX32mds_v2       |
| Standard_FX48-24mds_v2         | 24   | FX48mds_v2        |
| Standard_FX48-12mds_v2         | 12    | FX48mds_v2        |
| Standard_FX64-32mds_v2       | 32    | FX64mds_v2        |
| Standard_FX64-16mds_v2       | 16   | FX64mds_v2        |
| Standard_FX96-48mds_v2        | 48    | FX96mds_v2        |
| Standard_FX96-24mds_v2        | 24    | FX96mds_v2        |
| Standard_FX4-2ms_v2          | 2    | FX4ms_v2        |
| Standard_FX8-4ms_v2          | 4    | FX8ms_v2       |
| Standard_FX8-2ms_v2         | 2   | FX8ms_v2        |
| Standard_FX12-6ms_v2         | 6    | FX12ms_v2        |
| Standard_FX16-8ms_v2          | 8    | FX16ms_v2        |
| Standard_FX16-4ms_v2         | 4    | FX16ms_v2        |
| Standard_FX24-12ms_v2         | 12   | FX24ms_v2        |
| Standard_FX24-6ms_v2        | 6    | FX24ms_v2        |
| Standard_FX32-16ms_v2     | 16    | FX32ms_v2        |
| Standard_FX32-8ms_v2          | 8    | FX32ms_v2       |
| Standard_FX48-24ms_v2         | 24   | FX48ms_v2        |
| Standard_FX48-12ms_v2         | 12    | FX48ms_v2        |
| Standard_FX64-32ms_v2       | 32    | FX64ms_v2        |
| Standard_FX64-16ms_v2       | 16   | FX64ms_v2        |
| Standard_FX96-48ms_v2        | 48    | FX96ms_v2        |
| Standard_FX96-24ms_v2        | 24    | FX96ms_v2        |


#### [G-family sizes](#tab/family-G)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_GS4-8          | 8    | GS4        |
| Standard_GS4-4          | 4    | GS4        |
| Standard_GS5-16         | 16   | GS5        |
| Standard_GS5-8          | 8    | GS5        |


#### [D-family sizes](#tab/family-D)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_DS11-1_v2      | 1    | DS11_v2    |
| Standard_DS12-2_v2      | 2    | DS12_v2    |
| Standard_DS12-1_v2      | 1    | DS12_v2    |
| Standard_DS13-4_v2      | 4    | DS13_v2    |
| Standard_DS13-2_v2      | 2    | DS13_v2    |
| Standard_DS14-8_v2      | 8    | DS14_v2    |
| Standard_DS14-4_v2      | 4    | DS14_v2    |


#### [H-family sizes](#tab/family-H)

| Size name               | Active vCPUs | Base size |
|-------------------------|------|------------|
| Standard_HB368rs_v5     | 368  | HB368rs_v5 | 
| Standard_HB368_336rsv5  | 336  | HB368rs_v5 |
| Standard_HB368_288rsv5  | 288  | HB368rs_v5 |
| Standard_HB368-240rs_v5 | 240  | HB368rs_v5 |
| Standard_HB368-192rs_v5 | 192  | HB368rs_v5 |
| Standard_HB368-144rs_v5 | 144  | HB368rs_v5 |
| Standard_HB368-96rs_v5  | 96   | HB368rs_v5 |
| Standard_HB368-48rs_v5  | 48   | HB368rs_v5 |
| Standard_HB176rs_v4     | 176  | HB176rs_v4 | 
| Standard_HB176-144rs_v4 | 144  | HB176rs_v4 | 
| Standard_HB176-96rs_v4  | 96   | HB176rs_v4 | 
| Standard_HB176-48rs_v4  | 48   | HB176rs_v4 | 
| Standard_HB176-24rs_v4  | 24   | HB176rs_v4 | 
| Standard_HX176rs        | 176  | HX176rs    |
| Standard_HX176-144rs    | 144  | HX176rs    |
| Standard_HX176-96rs     | 96   | HX176rs    |
| Standard_HX176-48rs     | 48   | HX176rs    |
| Standard_HX176-24rs     | 24   | HX176rs    |
| Standard_HB120rs_v3     | 120  | HB120rs_v3 |
| Standard_HB120-96rs_v3  | 96   | HB120rs_v3 |
| Standard_HB120-64rs_v3  | 64   | HB120rs_v3 |
| Standard_HB120-32rs_v3  | 32   | HB120rs_v3 |
| Standard_HB120-16rs_v3  | 16   | HB120rs_v3 |
| Standard_HB120rs_v2     | 120  | HB120rs_v2 |
| Standard_HB120-96rs_v2  | 96   | HB120rs_v2 |
| Standard_HB120-64rs_v2  | 64   | HB120rs_v2 |
| Standard_HB120-32rs_v2  | 32   | HB120rs_v2 |
| Standard_HB120-16rs_v2  | 16   | HB120rs_v2 |
| Standard_HC44rs         | 44   | HC44rs     | 
| Standard_HC44-16rs      | 16   | HC44rs     | 
| Standard_HC44-32rs      | 32   | HC44rs     |

---

## Other standard sizes
- [Compute optimized](./sizes-compute.md)
- [Memory optimized](./sizes-memory.md)
- [Storage optimized](./sizes-storage.md)
- [GPU](./sizes-gpu.md)
- [High performance compute](./sizes-hpc.md) 

## Next steps
Learn more about how [Azure compute units (ACU)](./acu.md) can help you compare compute performance across Azure SKUs.
