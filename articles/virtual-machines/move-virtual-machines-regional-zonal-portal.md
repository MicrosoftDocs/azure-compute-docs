---
title: Move Azure single-instance virtual machines from regional to zonal availability
description: Learn how to move single-instance Azure virtual machines from a regional deployment to an Availability Zone in the same region.
ms.service: azure-virtual-machines
ms.topic: tutorial
ms.date: 02/17/2026
ms.custom: sfi-image-nochange
# Customer intent: "As a cloud administrator, I want to move Azure single instance virtual machines from a regional configuration to zonal availability zones, so that I can improve the reliability and performance of my applications within the same region."
---

This tutorial explains how to move a single-instance Azure virtual machine (VM) from a regional deployment to a zonal deployment in the same Azure region.

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

Use the following steps to move a VM from regional to zonal deployment in the same region.

### Select the VM

1. In the [Azure portal](https://ms.portal.azure.com/#home), go to your VM.
1. In the VM resource pane, select **Availability + scaling** > **Edit**.
   :::image type="content" source="./media/tutorial-move-regional-zonal/scaling-pane.png" alt-text="Screenshot of Availability + scaling pane.":::

You can also open **Availability + scale** from the VM overview and then select **Availability + scaling**.

### Select the target availability zones

1. Under **Target availability zone**, select the zone you want to use (for example, Zone 1).
   :::image type="content" source="./media/tutorial-move-regional-zonal/availability-scaling-home.png" alt-text="Screenshot of Availability + scaling homepage.":::

If the selected VM configuration isn't supported for move, validation fails and you must restart with a supported VM configuration. See the [support matrix](/azure/reliability/migrate-vm#support-matrix).

1. If Azure recommends a different VM size to improve deployment success in the selected zone, choose the recommended size, or choose a different zone and keep the current size.
   :::image type="content" source="./media/tutorial-move-regional-zonal/aure-recommendation.png" alt-text="Screenshot showing Azure recommendation to increase virtual machine size.":::

1. Select the consent statement for the **System Assigned Managed Identity** process, then select **Next**.

   :::image type="content" source="./media/tutorial-move-regional-zonal/move-virtual-machine-availability-zone.png" alt-text="Screenshot of select target availability zone.":::

The managed identity setup can take a few minutes. Progress updates are shown in the portal.

### Review VM properties before move

1. On **Review properties**, review the target VM configuration.

#### VM properties

The following source VM properties are retained by default in the target zonal VM:

| Property | Description |
| --- | --- |
| VM name | The source VM name is retained in the target zonal VM. |
| VNET | The source virtual network is retained, and the target zonal VM is created in the same VNET by default. You can select a different VNET. |
| Subnet | The source subnet is retained by default. You can select a different subnet. |
| NSG | The source NSG is retained by default. You can select or create a different NSG. |
| Load balancer (Standard SKU) | Standard SKU load balancers support zonal deployment and are retained. |
| Public IP (Standard SKU) | Standard SKU public IP addresses support zonal deployment and are retained. |

The following properties are newly created by default in the target zonal VM:

| Property | Description |
| --- | --- |
| VM | A copy of the source VM is created in the target zonal configuration. The source VM remains intact and is stopped after the move. Source VM ARM ID isn't retained. |
| Resource group | A new resource group is created by default. The source resource group can't be reused when the target VM uses the same name in the same region. You can select an existing target resource group. |
| NIC | A new NIC is created for the target zonal VM. The source NIC remains intact and is stopped after move. Source NIC ARM ID isn't retained. |
| Disks | New disks are created in the target zonal configuration and attached to the new zonal VM. |
| Load balancer (Basic SKU) | Basic SKU load balancers don't support zonal deployment and aren't retained. A new Standard SKU load balancer is created by default. You can edit this or select an existing target load balancer. |
| Public IP (Basic SKU) | Basic SKU public IP addresses don't support zonal deployment and aren't retained. A new Standard SKU public IP is created by default. You can edit this or select an existing target public IP. |

1. Resolve any validation errors.
1. Select the consent statement at the bottom of the page, then continue with the move.
   :::image type="content" source="./media/tutorial-move-regional-zonal/migrate-vms.png" alt-text="Screenshot of migrating virtual machine page.":::

### Move the VM

Select **Move** to complete the move to Availability Zones.

:::image type="content" source="./media/tutorial-move-regional-zonal/move-completed.png" alt-text="Screenshot of move completion page.":::

During move:

- The source VM is stopped, so brief downtime occurs.
- A target zonal VM is created and started.

### Configure settings after move

Review source VM settings and reconfigure as needed (for example, extensions, RBAC, Public IP configuration, backup, and disaster recovery settings).

### Delete or keep the source VM

After move completion, the source VM remains stopped. You can delete it or keep it for another purpose.

## Delete move resources created during migration

After the move, you can manually delete the move collection.

To remove the move collection:

1. Enable hidden resources (the move collection is hidden by default).
1. Go to the move collection resource group by searching for *ZonalMove-MC-RG-SourceRegion*.
1. Delete the move collection (for example, *ZonalMove-MC-RG-UKSouth*).

## Next steps

To perform the same workflow using scripting, see [Move Azure single-instance VMs from regional to zonal deployment using PowerShell or CLI](./move-virtual-machines-regional-zonal-powershell.md).
