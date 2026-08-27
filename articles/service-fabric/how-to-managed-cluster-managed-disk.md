---
title: Select managed disk types for Service Fabric managed cluster nodes
description: Learn how to select managed disk types for Service Fabric managed cluster nodes and configure in an ARM template.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: devx-track-arm-template
services: service-fabric
ms.date: 08/27/2026
# Customer intent: As a cloud engineer, I want to select and configure the appropriate managed disk types for my Service Fabric managed cluster nodes using an ARM template, so that I can optimize storage performance and reliability for my applications.
---

# Select managed disk types for Service Fabric managed cluster nodes

Azure Service Fabric managed clusters use managed disks for all storage needs, including application data, for scenarios such as reliable collections and actors. Azure managed disks are block-level storage volumes managed by Azure and used with Azure Virtual Machines. Managed disks are like a physical disk in an on-premises server but, virtualized. With managed disks, all you have to do is specify the disk size, the disk type, and provision the disk. Once you provision the disk, Azure handles the rest. For more information about managed disks, see [Introduction to Azure managed disks
](../virtual-machines/managed-disks-overview.md).

**Disk size update:** Customers have the capability to update the disk size on current node type; however, it's important to note that only new nodes on the existing node type receive the new disk size. To implement this change, users can follow two approaches:
* Scale the node type by adding new nodes with the desired disk size, and then remove the old nodes with smaller disk sizes.
* Alternatively, create a new node type with the desired disk size and migrate their workload to the new node type using placement constraints.
 
**Disk type update:** Updating disk types in place for node types isn't supported. Therefore, the only viable option is to create a new node type with the desired disk type and migrate the workload accordingly. This process ensures a seamless transition to the updated disk type without disrupting the cluster's operation.

## Managed disk types

Azure Service Fabric managed clusters support the following values for the `dataDiskType` property:

| Storage account type | Managed disk type | Redundancy |
|---|---|---|
| `PremiumV2_LRS` | Premium SSD v2 | Locally redundant storage (LRS) |
| `Premium_LRS` | Premium SSD | Locally redundant storage (LRS) |
| `Premium_ZRS` | Premium SSD | Zone-redundant storage (ZRS) |
| `StandardSSD_LRS` (default) | Standard SSD | Locally redundant storage (LRS) |
| `StandardSSD_ZRS` | Standard SSD | Zone-redundant storage (ZRS) |
| `Standard_LRS` | Standard HDD | Locally redundant storage (LRS) |
| `UltraSSD_LRS` | Ultra Disk | Locally redundant storage (LRS) |

For a feature and performance comparison, see [Compare Azure managed disk types](../virtual-machines/disks-types.md#compare-azure-managed-disk-types).

>[!NOTE]
> - `PremiumV2_LRS` is supported only on zonal (cross-AZ) node types. Enable zonal resiliency or specify one or more availability zones for the node type.
> - Managed disk type availability and supported VM sizes vary by region. Confirm that the selected disk type is available for the cluster region and VM size.
> - Any temporary disk associated with the VM size isn't used for storing Service Fabric or application data by default. [Stateless node types](how-to-managed-cluster-stateless-node-type.md) support temporary disks if required.

## Specifying a Service Fabric managed cluster disk type

To specify a Service Fabric managed cluster disk type, set the `dataDiskType` property in the managed cluster resource definition to one of the [supported storage account types](#managed-disk-types).

```json
{
  "apiVersion": "2021-05-01",
  "type": "Microsoft.ServiceFabric/managedclusters",
  "dataDiskType": "StandardSSD_LRS"
}
```

Sample templates are available that include this specification: [Service Fabric managed cluster templates](https://github.com/Azure-Samples/service-fabric-cluster-templates).
