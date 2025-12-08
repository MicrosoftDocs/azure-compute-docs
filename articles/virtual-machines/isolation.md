---
title: Isolation for VMs in Azure
description: Learn about VM isolation works in Azure.
author: mimckitt
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 04/20/2023
ms.author: briannali
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to understand how VM isolation works in Azure, so that I can effectively design secure and efficient cloud infrastructure for my organization."
---

# Virtual machine isolation in Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets


Azure Compute offers virtual machine sizes that are isolated to a specific hardware type and dedicated to a single customer. Isolated sizes run on specific hardware generations. They're deprecated when the hardware generation is retired or a new generation becomes available.

Isolated virtual machine sizes are best suited for workloads that require a high degree of isolation from other customers’ workloads. This requirement sometimes applies to compliance and regulatory standards. Utilizing an isolated size guarantees that your virtual machine is the only one running on that specific server instance. 


Additionally, as the isolated size virtual machines are large, customers may choose to subdivide the resources of these virtual machines by using [Azure support for nested virtual machines](https://azure.microsoft.com/blog/nested-virtualization-in-azure/).

The current isolated virtual machine offerings include:

| Family | Size | 
| --- | --- |
| [E](./sizes/memory-optimized/e-family.md) | Standard_E192is_v6 <br>Standard_E192ids_v6 <br>Standard_E104i_v5 <br>Standard_E104id_v5 <br>Standard_E104is_v5 <br>Standard_E104ids_v5 <br>Standard_E112ias_v5 <br>Standard_E112iads_v5 <br>Standard_E80is_v4 <br>Standard_E80ids_v4 <br>Standard_E96ias_v4 |
| Eb | Standard_E112ibs_v5 <br>Standard_E112ibds_v5 |
| EC | Standard_EC96ias_v5 <br>Standard_EC96iads_v5 |
| M | Standard_M832is_16_v3 <br>Standard_M832ids_16_v3 <br>Standard_M192is_v2<sup>1</sup> <br>Standard_M192ids_v2<sup>1</sup> <br>Standard_M192ims_v2<sup>1</sup> <br>Standard_M192idms_v2<sup>1</sup> |
| NC | Standard_NC80adis_H100_v5 |
| ND | Standard_ND96is_H100_v5 <br>Standard_ND96isr_H100_v5 <br>Standard_ND96isr_H200_v5 <br>Standard_ND96is_MI300X_v5 <br>Standard_ND96isr_MI300X_v5 |

<sup>1</sup> These sizes are announced for retirement

> [!NOTE]
> Isolated VM sizes have a limited lifespan that is tied to the lifetime of the hardware they run on. 

## Retired Isolated VM Sizes

Isolated VM sizes have a hardware limited lifespan. Azure issues reminders 12 months in advance of the official deprecation date of the sizes and provides an updated isolated offering for your consideration. The following sizes are retired or announced for retirement.

| Size | Retirement Announcement | Isolation Retirement Date | Migration Guide |
| --- | --- | --- | --- |
| Standard_DS15_v2  | [05/15/2020](https://azure.microsoft.com/en-us/updates?id=the-d15-v2-ds15-v2-azure-virtual-machines-may-no-longer-be-isolated-starting-february-15-2020) | 05/15/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_D15_v2   | [05/15/2020](https://azure.microsoft.com/en-us/updates?id=the-d15-v2-ds15-v2-azure-virtual-machines-may-no-longer-be-isolated-starting-february-15-2020) | 05/15/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_G5       | [02/15/2021](https://azure.microsoft.com/en-us/updates?id=the-g5-and-gs5-azure-vms-will-no-longer-be-hardwareisolated-on-28-february-2022) | 02/28/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_GS5      | [02/15/2021](https://azure.microsoft.com/en-us/updates?id=the-g5-and-gs5-azure-vms-will-no-longer-be-hardwareisolated-on-28-february-2022) | 02/28/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_E64i_v3  | [02/15/2021](https://azure.microsoft.com/en-us/updates?id=the-e64iv3-e64isv3-azure-vms-will-not-be-hardwareisolated-on-28-february-2022) | 02/28/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_E64is_v3 | [02/15/2021](https://azure.microsoft.com/en-us/updates?id=the-e64iv3-e64isv3-azure-vms-will-not-be-hardwareisolated-on-28-february-2022) | 02/28/2021 | [Migration Guide](/azure/virtual-machines/migration/sizes/d-ds-dv2-dsv2-ls-series-migration-guide) |
| Standard_M192is_v2| - | 03/31/2027 | - |
| Standard_M192ims_v2| - | 03/31/2027 | - |
| Standard_M192ids_v2| - | 03/31/2027 | - |
| Standard_M192idms_v2| -| 03/31/2027 | - |


## FAQ
### Q: What specific scenarios benefit most from isolated VM sizes? 
**A**: Isolated VM sizes run on dedicated physical servers for maximum security and consistency. These sizes are best for regulated workloads, highly sensitive data, and performance-critical systems. 

### Q: Is the size going to be retired or only its "isolation" feature?
**A**: If a VM sizes is published as isolated but doesn't have an "i" in its name, only the isolation feature is being retired (unless stated otherwise). VM sizes with an "i" in the name are deprecated entirely.

### Q: Is there downtime if my VM moves to nonisolated hardware?
**A**: For sizes where only isolation is retired (not the size), no action is needed and there's no downtime. If isolated is required, the announcement will include a recommended replacement size. Moving to that sizes requires resizing your VM.  

### Q: Is there any cost difference when moving to a nonisolated VM?
**A**: No, there's no cost difference.

### Q: When are the other isolated sizes going to retire?
**A**: We provide reminders 12 months in advance of the official deprecation of the isolated size.

###Q: What should I do if the isolated VM size I need is not available in my region? 
**A**: Choose an alternative SKU with similar specs first, and if none meet your needs or deploy the isolated VM size in a supported region. 
 
## Next steps

Customers can also choose to further subdivide the resources of these isolated virtual machines by using [Azure support for nested virtual machines](https://azure.microsoft.com/blog/nested-virtualization-in-azure/).

