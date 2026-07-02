---
title: Deploy Service Fabric node types with managed data disks
description: Learn how to create and deploy Service Fabric node types with attached managed data disks.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ai-usage: ai-assisted
ms.date: 03/22/2026
# Customer intent: As a cloud architect, I want to configure Azure Service Fabric node types with managed data disks, so that I can ensure persistent and scalable data storage for my applications.
---

# Deploy an Azure Service Fabric cluster node type with managed data disks

Azure Service Fabric node types, by default, use the temporary disk on each virtual machine (VM) in the underlying virtual machine scale set for data storage. However, because the temporary disk isn't persistent, and the size of the temporary disk is bound to a given VM SKU, this can be too restrictive for some scenarios. 

This article provides the steps for how to use native support from Service Fabric to configure and use managed data disks as the default data path. Service Fabric will automatically configure managed data disks at node type creation and handle situations where VMs or the virtual machine scale set is reimaged.

> [!IMPORTANT]
> VMSS `_v6` SKU sizes introduce [NVMe disks](/azure/virtual-machines/nvme-overview), which replace the older SCSI-based protocol for disk storage. This change affects disk characteristics and can break existing disk initialization logic for both managed data disks and temporary disks. As a result, required partitions or file systems might not be created, and Service Fabric node bootstrap can stall.
>
> Service Fabric managed data disk support doesn't initialize NVMe disks automatically. You must initialize and format NVMe disks yourself by using a Custom Script Extension, including when you use the temporary disk for `dataPath`.
>
> Although NVMe disks are commonly associated with `_v6` SKUs, some earlier or non-v6 series also use NVMe. For example, [Fxmsv2-series](/azure/virtual-machines/sizes/compute-optimized/fxmsv2-series?tabs=sizebasic#feature-support) uses NVMe-based local storage. Any VM SKU with NVMe disks is subject to the same initialization considerations in this article. For more information, see [Enable NVMe interface on your VMs and VM scale sets (FAQ)](/azure/virtual-machines/enable-nvme-temp-faqs).
>
> [Service Fabric managed clusters](/azure/service-fabric/overview-managed-cluster) support NVMe scenarios. If you want a simpler managed experience for NVMe-based SKUs, consider migrating to Service Fabric managed clusters.

## Prerequisites

* The required minimum disk size for the managed data disk is 50 GB.
* Data disk drive letter should be set to character lexicographically greater than all drives present in the virtual machine scale set SKU. 
* Only one managed data disk per VM is supported. For scenarios involving more than 1 data disks, user needs to manage the data disks on their own.

## Configure the virtual machine scale set to use managed data disks in Service Fabric
To use managed data disks on a node type, configure the underlying virtual machine scale set resource with the following:

* Add a managed disk in data disks section of the template for the virtual machine scale set. 
* Update the Service Fabric extension for the virtual machine scale set with following settings: 
    * For Windows: **useManagedDataDisk: true** and **dataPath: 'K:\\\\SvcFab'**. Note that drive K is just a representation. You can use any drive letter lexicographically greater than all the drive letters present in the virtual machine scale set SKU.
    * For Linux: Not supported.

Here's an Azure Resource Manager template for a Service Fabric extension:

```json
{
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(parameters('vmNodeType1Name'),'_ServiceFabricNode')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "enableAutomaticUpgrade": true,
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                            "nodeTypeRef": "[parameters('vmNodeType1Name')]",
                            "dataPath": "K:\\\\SvcFab",
                            "useManagedDataDisk": true,
                            "durabilityLevel": "Bronze",
                            "certificate": {
                                "thumbprint": "[parameters('certificateThumbprint')]",
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            },
                            "systemLogUploadSettings": {
                                "Enabled": true
                            },
                        },
                        "typeHandlerVersion": "1.1"
                    }
                },
            ]
        },
        "storageProfile": 
        {
            "datadisks": [
                {
                    "lun": "1",
                    "createOption": "empty",
                    "diskSizeGB": "100",
                    "managedDisk": { "storageAccountType": "Standard_LRS" }
                }
            ]
        }
    }
}
```

## Migrate to using managed data disks for Service Fabric node types
For all migration scenarios, new node types with managed data disks need to be added. Existing node types can't be converted to use managed data disks.

1. Add a new node type that's configured to use managed data disks as specified earlier.
2. Migrate any required workloads to the new node type.
3. Disable and remove the old node type from the cluster.


## Next steps 
* [Service Fabric overview](service-fabric-reliable-services-introduction.md)
* [Node types and virtual machine scale sets](service-fabric-cluster-nodetypes.md)
* [Service Fabric capacity planning](service-fabric-best-practices-capacity-scaling.md)
