---
title: Move Azure single-instance virtual machines from regional to zonal availability
description: Learn how to move single-instance Azure virtual machines from a regional deployment to an Availability Zone in the same region by using the Azure portal, PowerShell, or CLI.
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 02/20/2026
ms.custom: sfi-image-nochange portal
# Customer intent: "As a cloud administrator, I want to move Azure single instance virtual machines from a regional configuration to zonal availability zones, so that I can improve the reliability and performance of my applications within the same region."
---

# Move Azure single-instance virtual machines from regional to zonal availability

This article explains how to move a single-instance Azure virtual machine (VM) from a regional deployment to a zonal deployment in the same Azure region.

## Prerequisites

Before you begin, verify the following:

- **Availability Zone support**: Confirm that your target region supports Availability Zones. [Learn more](/azure/reliability/availability-zones-region-support).

- **VM SKU availability**: VM size (SKU) availability can vary by region and zone. Check SKU availability before you start. [Learn more](../virtual-machines/windows/create-powershell-availability-zone.md#check-vm-sku-availability).

- **Subscription permissions**: Verify that you have *Owner* access to the subscription that contains the VMs you want to move.
  The first time you move a VM to a zonal deployment, Azure requires a trusted [system-assigned managed identity](/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types). The account you use must have Owner permissions so Azure can create the identity and assign the required role (Contributor or User Access Administrator) in the source subscription. [Learn more](/azure/role-based-access-control/rbac-and-directory-admin-roles#azure-roles).

- **VM support**: Confirm that the VM configuration is supported for this move workflow. [Learn more](/azure/reliability/migrate-vm).

- **Subscription quota**: Ensure the subscription has enough quota to create the target zonal VM and related networking resources. If needed, [request additional limits](/azure/azure-resource-manager/management/azure-subscription-service-limits).
- **Regional or zonal capacity**: Quota approval doesn't guarantee that capacity is available in the target zone. If the move validation reports capacity constraints, select a different target zone or VM size.
- **VM health**: Ensure the VM is healthy and that required reboots or mandatory updates are complete.

## Select and move VMs

Choose a tool to move a VM from regional to zonal deployment in the same region.


## [Portal](#tab/portal)

### Portal: Select the VM

1. In the [Azure portal](https://ms.portal.azure.com/#home), go to your VM.
1. In the VM overview, select **Availability + scaling** > **Edit**.
   :::image type="content" source="./media/tutorial-move-regional-zonal/scaling-pane.png" alt-text="Screenshot of Availability + scaling pane.":::


### Portal: Select the target availability zone

1. Under **Target availability zone**, select the zone you want to use (for example, Zone 1).
   :::image type="content" source="./media/tutorial-move-regional-zonal/availability-scaling-home.png" alt-text="Screenshot of Availability + scaling homepage.":::

   If the selected VM configuration isn't supported for move, validation fails and you must restart with a supported VM configuration. See the [support matrix](/azure/reliability/migrate-vm#support-matrix).

1. If Azure recommends a different VM size to improve deployment success in the selected zone, choose the recommended size, or choose a different zone and keep the current size.
   :::image type="content" source="./media/tutorial-move-regional-zonal/aure-recommendation.png" alt-text="Screenshot showing Azure recommendation to increase virtual machine size.":::

1. Select the consent statement for the **System Assigned Managed Identity** process, then select **Next**.

   

The managed identity setup can take a few minutes. Progress updates are shown in the portal.

### Portal: Review VM properties before move

On **Review properties**, review the target VM configuration.


The following source VM properties are retained by default in the target zonal VM:

| Property | Description |
| --- | --- |
| VM name | The source VM name is retained in the target zonal VM. |
| Virtual network/subnet| The source virtual network (VNET) and subnet are retained, and the target zonal VM is created in the same VNET and subnet by default. You can also select or create a different VNET and subnet. |
| Network Security Group (NSG) | The source NSG is retained by default. You can select or create a different NSG. |
| Load balancer (Standard SKU) | Standard SKU load balancers support zonal deployment and are retained. |
| Public IP (Standard SKU) | Standard SKU public IP addresses support zonal deployment and are retained. |

The following properties are newly created by default in the target zonal VM as needed:

| Property | Description |
| --- | --- |
| VM | A copy of the source VM is created in the target zonal configuration. The source VM remains intact and is stopped after the move. Source VM ARM ID isn't retained. |
| Resource group | A new resource group is created by default. The source resource group can't be reused when the target VM uses the same name in the same region. You can select an existing target resource group. |
| NIC | A new NIC is created for the target zonal VM. The source NIC remains intact and is stopped after move. Source NIC ARM ID isn't retained. |
| Disks | New disks are created in the target zonal configuration and attached to the new zonal VM. |
| Load balancer (Basic SKU) | Basic SKU load balancers don't support zonal deployment and aren't retained. A new Standard SKU load balancer is created by default. You can edit this or select an existing target load balancer. |
| Public IP (Basic SKU) | Basic SKU public IP addresses don't support zonal deployment and aren't retained. A new Standard SKU public IP is created by default. You can edit this or select an existing target public IP. |


After reviewing the target configuration:
1. Resolve any validation errors.
1. Select the consent statement at the bottom of the page to show you understand that the source VM(s) will be stopped during the process. 
1. Select **Move** to complete the move.


During the move:

- The source VM is **stopped**, so brief downtime occurs.
- A target zonal VM is created and started.


### Configure settings after move

- Review source VM settings and reconfigure as needed (for example, extensions, RBAC, Public IP configuration, backup, and disaster recovery settings).

- After move completion, the source VM remains stopped. You can delete it or keep it for another purpose.

- After the move, you can manually delete the move collection. To remove the move collection:

    1. Enable hidden resources (the move collection is hidden by default).
    1. Go to the move collection resource group by searching for *ZonalMove-MC-RG-SourceRegion*.
    1. Delete the move collection (for example, *ZonalMove-MC-RG-UKSouth*).


## [PowerShell](#tab/powershell)

### PowerShell: Sign in and set context

```azurepowershell-interactive
Set-AzContext -SubscriptionId "<subscription-id>"
Connect-AzAccount -Subscription "<subscription-id>"
```

### PowerShell: Create move collection and identity

```azurepowershell-interactive
New-AzResourceGroup -Name "RegionToZone-DemoMCRG" -Location "EastUS"
Register-AzResourceProvider -ProviderNamespace Microsoft.Migrate
New-AzResourceMoverMoveCollection -Name "RegionToZone-DemoMC" -ResourceGroupName "RegionToZone-DemoMCRG" -MoveRegion "eastus" -Location "eastus2euap" -IdentityType "SystemAssigned" -MoveType "RegionToZone"
```

### PowerShell: Add VM move resource and resolve dependencies

```azurepowershell-interactive
$targetResourceSettingsObj = @{ 
  "resourceType" = "Microsoft.Compute/virtualMachines"
  "targetAvailabilityZone" = "2"
  "targetResourceName" = "<target-vm-name>"
  "targetVmSize" = "<target-vm-size>"
}

Add-AzResourceMoverMoveResource -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC" -SourceId "/subscriptions/<subscription-id>/resourcegroups/<rg>/providers/Microsoft.Compute/virtualMachines/<source-vm-name>" -Name "demoVM-MoveResource" -ResourceSetting $targetResourceSettingsObj
Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC"
```

### PowerShell: Validate recommendations (SKU, quota, capacity)

Quota and capacity are validated separately:

- **Quota** is your subscription limit for vCPUs and virtual machines.
- **Capacity** is available infrastructure in the selected region and zone.

If recommendations require a different zone or size, update the move resource and resolve dependencies again.

```azurepowershell-interactive
$targetResourceSettingsObj.TargetVmSize = "Standard_DC1ds_v3"
$targetResourceSettingsObj.TargetAvailabilityZone = "3"
Add-AzResourceMoverMoveResource -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC" -SourceId "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/<source-vm-name>" -Name "demoVM-MoveResource" -ResourceSetting $targetResourceSettingsObj
Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC"
```

### PowerShell: Initiate and commit move

```azurepowershell-interactive
Invoke-AzResourceMoverInitiateMove -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC" -MoveResource $("demoVM-MoveResource")
Invoke-AzResourceMoverCommit -ResourceGroupName "RegionToZone-DemoMCRG" -MoveCollectionName "RegionToZone-DemoMC" -MoveResource $("demoVM-MoveResource")
```

## [CLI](#tab/cli)

### CLI: Sign in and set subscription

```azurecli-interactive
az login
az account set --subscription <subscription-id>
```

### CLI: Create move collection and identity

```azurecli-interactive
az group create --location eastus2 --name clidemo-RG
az resource-mover move-collection create --identity type=SystemAssigned --location eastus2 --move-region eastus --name cliDemo-zonalMC --resource-group clidemo-RG --move-type RegionToZone
```

### CLI: Add VM move resource and resolve dependencies

```azurecli-interactive
az resource-mover move-resource add --resource-group clidemo-RG --move-collection-name cliDemo-zonalMC --name vm-demoMR --source-id "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/<source-vm-name>" --resource-settings '{"resourceType":"Microsoft.Compute/virtualMachines","targetResourceName":"<target-vm-name>","targetAvailabilityZone":"2","targetVmSize":"<target-vm-size>"}'
az resource-mover move-collection resolve-dependency --name cliDemo-zonalMC --resource-group clidemo-RG
```

### CLI: Validate recommendations (SKU, quota, capacity)

Quota and capacity are validated separately. If move recommendations require a different zone or VM size, update the move resource settings and resolve dependencies again before initiating the move.

### CLI: Initiate and commit move

```azurecli-interactive
az resource-mover move-collection initiate-move --move-resources "/subscriptions/<subscription-id>/resourceGroups/clidemo-RG/providers/Microsoft.Migrate/moveCollections/cliDemo-zonalMC/moveResources/vm-demoMR" --validate-only false --name cliDemo-zonalMC --resource-group clidemo-RG
az resource-mover move-collection commit --move-resources "/subscriptions/<subscription-id>/resourceGroups/clidemo-RG/providers/Microsoft.Migrate/moveCollections/cliDemo-zonalMC/moveResources/vm-demoMR" --validate-only false --name cliDemo-zonalMC --resource-group clidemo-RG
```

---

## Delete source regional VMs

After you commit the move and verify that the resources work as expected in the target region, you can delete each source resource using:

- [Azure portal](/azure/azure-resource-manager/management/manage-resources-portal#delete-resources)
- [PowerShell](/azure/azure-resource-manager/management/manage-resources-powershell#delete-resources)
- [Azure CLI](/azure/azure-resource-manager/management/manage-resource-groups-cli#delete-resource-groups)


## Related content

- For deployment guidance and limits, see [Move to Availability Zones (Azure portal)](/azure/availability-zones/migrate-vm?toc=/azure/virtual-machines/toc.json&bc=/azure/virtual-machines/breadcrumb/toc.json).
- See [Regional to zonal move FAQ](./move-virtual-machines-regional-zonal-faq.md).
