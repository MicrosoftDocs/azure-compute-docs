---
title: Customer-selected availability zones for Virtual Machine Scale Sets
description: Learn how to select the availability zones that Azure Virtual Machine Scale Sets uses when deploying virtual machines.
author: hilaryw29
ms.author: hilarywang
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 08/10/2026
ms.reviewer: fisteele
ms.custom: devx-track-azurecli, devx-track-azurepowershell, devx-track-arm-template
# Customer intent: As a cloud architect, I want to select the availability zones that my Virtual Machine Scale Set uses, so that I can meet my workload's placement and resiliency requirements.
---

# Configure customer-selected availability zones for a scale set

By using customer-selected availability zones, you specify the zones that a Virtual Machine Scale Set can use by setting the `zones` property. You can select one zone for a zonal deployment or multiple zones for a zone-spanning deployment.

Use customer-selected zones when your workload must use specific availability zones. If you want Azure to select zones based on capacity and SKU availability, consider [automatic zone placement](virtual-machine-scale-sets-automatic-zone-placement.md).

## Prerequisites

- Create the scale set in an [Azure region that supports availability zones](/azure/reliability/availability-zones-region-support).
- Verify that your VM size and disk types are available in the zones that you want to use. Use the [Compute Resource SKUs API](/rest/api/compute/resource-skus/list?tabs=HTTP) to determine which sizes are available in each zone.

## Create zone-spanning or zonal scale sets

When you deploy a Virtual Machine Scale Set, you can choose to use a single availability zone in a region, or multiple zones.

You can create a scale set that uses availability zones by using one of the following methods:

- [Azure portal](#use-the-azure-portal)
- [Azure CLI](#use-azure-cli)
- [Azure PowerShell](#use-azure-powershell)
- [Azure Resource Manager templates](#use-an-azure-resource-manager-template)

### Use the Azure portal

The process to create a scale set that uses an availability zone is the same as detailed in the [getting started article](quick-create-portal.md). When you select a supported Azure region, you can create a scale set in one or more available zones, as shown in the following example:

![Screenshot of how to create a scale set in a single availability zone.](media/virtual-machine-scale-sets-use-availability-zones/vmss-az-portal.png)

The scale set and supporting resources, such as the Azure load balancer and public IP address, are created in the single zone that you specify.

### Use Azure CLI

The process to create a scale set that uses an availability zone is the same as detailed in the [getting started article](quick-create-cli.md). To use availability zones, create your scale set in a supported Azure region.

Add the `--zones` parameter to the [az vmss create](/cli/azure/vmss) command and specify which zone to use (such as zone *1*, *2*, or *3*).

```azurecli
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image <SKU Image> \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --zones 1 2 3
```

It takes a few minutes to create and configure all the scale set resources and VMs in the zone(s) that you specify. For a complete example of a zone-redundant scale set and network resources, see [this sample CLI script](scripts/cli-sample-zone-redundant-scale-set.md#sample-script)

### Use Azure PowerShell

To use availability zones, create your scale set in a supported Azure region. Add the `-Zone` parameter to the [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig) command and specify which zone or zones to use (such as zone *1*, *2*, or *3*).

```powershell
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS2" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicy "Automatic" `
  -Zone "1", "2", "3"
```

### Use an Azure Resource Manager template

The process to create a scale set that uses an availability zone is the same as detailed in the getting started article for [Linux](quick-create-template-linux.md) or [Windows](quick-create-template-windows.md).

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "name": "myScaleSet",
  "location": "East US 2",
  "apiVersion": "2017-12-01",
  "zones": [
        "1",
        "2",
        "3"
      ]
}
```

If you create a public IP address or a load balancer, specify the `"sku": {"name":"Standard"}` property to create zone-redundant network resources. You also need to create a Network Security Group and rules to permit any traffic. For more information, see [Azure Load Balancer Standard Overview](/azure/load-balancer/load-balancer-overview) and [Standard Load Balancer and Availability Zones](/azure/load-balancer/load-balancer-standard-availability-zones).

## Update a scale set to add availability zones

You can modify a scale set to expand the set of zones over which to spread VM instances. Expanding the set of zones allows you to take advantage of a higher availability SLA (99.99%) versus regional (nonzonal) availability SLA (99.95%). Or expand your scale set to take advantage of new availability zones that weren't available when you created the scale set.

This feature can be used with API version 2023-03-01 or greater.

### Expand scale set to use availability zones

You can update the scale set to scale out instances to one or more additional availability zones, up to the number of availability zones supported by the region. For regions that support zones, the minimum number of zones is three.

> [!IMPORTANT]
> When you expand the scale set to additional zones, the original instances are not migrated or changed. When you scale out, new instances will be created and spread evenly across the selected availability zones. Data from the original instances are not migrated to the new zones. When you scale in the scale set, any regional (nonzonal) instances will be priorized for removal first. After that, instances will be removed based on the [scale in policy](virtual-machine-scale-sets-scale-in-policy.md).

Expanding to a zone-spanning scale set is done in three steps:

1. Prepare for zone expansion
1. Update zones parameter on the scale set
1. Add new zonal instances and remove original instances

#### Prepare for zone expansion

> [!WARNING]
> This feature allows you to add zones to the scale set. You can't go back to a regional (nonzonal) scale set or remove zones once you add them.

To prepare for zone expansion:

- [Check that you have enough quota](../virtual-machines/quotas.md) for the VM size in the selected region to handle more instances.
- Check that the VM size and disk types you use are available in all the desired zones. Use the [Compute Resources SKUs API](/rest/api/compute/resource-skus/list?tabs=HTTP) to determine which sizes are available in which zones.
- Validate that the scale set configuration is valid for zonal and zone-spanning scale sets:
  - Set `platformFaultDomainCount` to 1 or 5. Fixed spreading with 2 or 3 fault domains isn't supported for zonal and zone-spanning scale sets.
  - Capacity reservations are not supported during zone expansion. Once the scale set is fully zone-spanning or zonal (no more regional (nonzonal) instances), you can add a capacity reservation group to the scale set.
  - Azure Dedicated Host deployments are not supported.

#### Update the zones parameter on the scale set

Update the scale set to change the zones parameter.

### [Azure portal](#tab/portal-2)

1. Navigate to the scale set you want to update
1. On the **Availability** tab of the scale set landing page, find the **Availability zone** property and select **Edit**.
1. In the **Edit Location** dialog box, select the zones you want.
1. Select **Apply**.

### [Azure CLI](#tab/cli-2)

```azurecli
az vmss update --set zones=["1","2","3"] -n < myScaleSet > -g < myResourceGroup >
```

### [Azure PowerShell](#tab/powershell-2)

```azurepowershell
# Get the Virtual Machine Scale Set object
$vmss = Get-AzVmss -ResourceGroupName < resource-group-name > -VMScaleSetName < vmss-name >

# Update the zones parameter
$vmss.Zones = [Collections.Generic.List[string]]('1','2','3')

# Apply the changes
Update-AzVmss -ResourceGroupName < resource-group-name > -VMScaleSetName < vmss-name > -VirtualMachineScaleSet $vmss
```

### [REST API](#tab/template-2)

```json
PATCH /subscriptions/subscriptionid/resourceGroups/resourcegroupo/providers/Microsoft.Compute/virtualMachineScaleSets/myscaleset?api-version=2023-03-01

```javascript
{
  "zones": [
    "1",
    "2",
    "3"
  ]
}
```

---

### Add new zonal instances and remove original instances

You can manually balance your scale set across zones by triggering a scale-out operation and then scaling in. For more details, see [How to manually balance your scale set](./virtual-machine-scale-sets-zone-balancing.md#how-to-manually-balance-your-scale-set).

### Known issues and limitations

- The original instances are not migrated to the newly added zones. Your workload must handle any required data migration or replication.

- Scale sets running Service Fabric RP or Azure Kubernetes Service are not supported.

- You can't remove or replace zones, only add zones

- You can't update from a zone spanning or zonal scale set to a regional (nonzonal) scaleset.

- `platformFaultDomainCount` must be set to 1 or 5. Fixed spreading with 2 or 3 fault domains isn't supported for zone-spanning or zonal deployments.

- Capacity reservations are not supported during zone expansion. Once the scale set is fully zone-spanning or zonal (no more regional (nonzonal) instances), you can add a capacity reservation group to the scale set.

- Azure Dedicated Host deployments are not supported

## Next steps

Now that you have created a scale set in an availability zone, you can learn how to [Deploy applications on Virtual Machine Scale Sets](tutorial-install-apps-cli.md) or [Use autoscale with Virtual Machine Scale Sets](tutorial-autoscale-cli.md).