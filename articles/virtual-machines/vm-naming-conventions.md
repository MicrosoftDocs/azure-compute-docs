---
title: Azure VM sizes naming conventions
description: Explains the naming conventions used for Azure VM sizes
ms.service: azure-virtual-machines
subservice: sizes
author: mimckitt
ms.topic: concept-article
ms.date: 05/01/2023
ms.author: mimckitt
ms.reviewer: mattmcinnes
ms.custom: sttsinar
# Customer intent: "As an IT administrator, I want to understand the naming conventions for Azure VM sizes, so that I can select the appropriate VM configuration based on features and specifications for my infrastructure needs."
---

# Azure virtual machine sizes naming conventions

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This page outlines the naming conventions used for Azure VMs. VMs use these naming conventions to denote varying features and specifications.

## Naming convention explanation

**[Family]** + **[Sub-family*]** + **[# of vCPUs]** + **[Constrained vCPUs*]** + **[Additive Features]** + **[Accelerator Type*]** + **[Memory Capacity*]** + **[Version]**

|Value | Explanation|
|---|---|
| Family | Indicates the VM Family Series| 
| *Subfamily | Used for specialized VM differentiations such as: <br> B = memory bandwidth optimized <br>C = confidential (for DC, EC, NCC series)<br>C = compute intensive (for HC, NC, NCC series) <br>D = AI training and inference optimized <br>G = cloud gaming and remote desktop optimized <br>V = visualization and graphics optimized <br>X = extra memory|
| # of vCPUs| Denotes the number of vCPUs of the VM |
| *Constrained vCPUs| Used for certain VM sizes only. Denotes the number of vCPUs for the [constrained vCPU capable size](./constrained-vcpu.md) |
| Additive Features | Lower case letters denote additive features, such as: <br>a = AMD-based processor <br>b = remote storage bandwidth optimized <br>d = includes a local temp disk <br>e = encrypted; contains confidential TDX capabilities <br>f = flat ratio (1:1) of vCPU to memory size <br>i = isolated size <br>l = low memory; decreased vCPU to memory ratio <br>m = memory intensive; highest vCPU to memory ratio in a particular series <br>n = network optimized; increased vCPU to network bandwidth ratio <br>o = increased vCPU to local SSD storage capacity ratio<br>p = ARM-based processor <br>r = includes RDMA (InfiniBand) secondary network <br>s = compatible with any premium SSD type <br>t = tiny memory; smallest vCPU to memory ratio in a particular size 
| *Accelerator Type | Denotes the type of hardware accelerator in the specialized/GPU SKUs. Only new specialized/GPU SKUs launched from Q3 2020 have the hardware accelerator in the name. |
| *Memory Capacity | Denotes M-series memory capacity, rounded to the nearest TiB.
| Version | Denotes the version number of the VM Family Series |

## Example breakdown

**[Family]** + **[Sub-family*]** + **[# of vCPUs]** + **[Constrained vCPUs*]** + **[Additive Features]** + **[Accelerator Type*]** + **[Memory Capacity*]** + **[Version]**

### Example 1: M48ds_1_v3

|Value | Explanation|
|---|---|
| Family | M | 
| # of vCPUs | 48 |
| Additive Features | d = local temp disk <br> s = Premium Storage capable |
| Memory Capacity | 974 GiB ≈ 1 TiB |
| Version | v3 |

### Example 2: NV16as_v4

|Value | Explanation|
|---|---|
| Family | N | 
| Subfamily | V |
| # of vCPUs | 16 |
| Additive Features | a = AMD-based processor <br> s = Premium Storage capable |
| Version | v4 |

### Example 3: NC4as_T4_v3

|Value | Explanation|
|---|---|
| Family | N | 
| Subfamily | C |
| # of vCPUs | 4 |
| Additive Features | a = AMD-based processor <br> s = Premium Storage capable |
| Accelerator Type | T4 |
| Version | v3 |

### Example 4: M8-2ms_v2 (Constrained vCPU)

|Value | Explanation|
|---|---|
| Family | M | 
| # of vCPUs | 8 |
| Constrained vCPUs | 2 |
| Additive Features | m = memory intensive <br> s = Premium Storage capable |
| Version | v2 |

## Next steps

Learn more about available [VM Sizes](./sizes.md) in Azure.
