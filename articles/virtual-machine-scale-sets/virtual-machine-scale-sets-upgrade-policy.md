---
title: Upgrade policy modes for Virtual Machine Scale Sets
description: Learn about the different upgrade policies available for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: overview
ms.service: azure-virtual-machine-scale-sets
ms.date: 03/21/2025
ms.reviewer: ju-shim
ms.custom: upgradepolicy, ignite-2024
# Customer intent: As a cloud administrator, I want to understand the different upgrade policy modes for Virtual Machine Scale Sets, so that I can choose the most appropriate approach to manage updates while ensuring service uptime and instance availability.
---
# Upgrade policy modes for Virtual Machine Scale Sets

The upgrade policy mode you choose can impact the overall service uptime of your Virtual Machine Scale Set. The available upgrade policy modes are: **automatic**, **manual**, and **rolling**. 



## Upgrade policy modes

Each Virtual Machine Scale Set has an **upgrade policy mode** which determines how instances are brought up to date with the latest scale set model. Changes to the scale set model include updates that need to be applied to each individual instance. Some examples include: 
- Image reference upgrades
- SKU changes
- Add/ remove a data disk
- Add/ remove extensions
- Adding availability zones

Additionally, there can be situations where you might want specific instances in your scale set to be treated differently from the rest. For example, certain instances in the scale set could be needed to perform different tasks than the other members of the scale set. In these situations, [Instance Protection](virtual-machine-scale-sets-instance-protection.md) or [custom metrics for rolling upgrade policy (preview)](virtual-machine-scale-sets-rolling-upgrade-custom-metrics.md) provide the controls needed to protect these instances from being upgraded along side the other instances. 

### Automatic upgrade policy mode

With an automatic upgrade policy mode, the scale set makes no guarantees about the order of virtual machines being brought down. The scale set might take down all virtual machines at the same time to perform upgrades. 

:::image type="content" source="./media/upgrade-policy/automatic-upgrade.png" alt-text="Diagram that shows a high level diagram of what happens when using an automatic upgrade policy.":::

Automatic upgrade policy mode is best suited for DevTest scenarios where you aren't concerned about the uptime of your instances while making changes to configurations and settings. 

If your scale set is part of a Service Fabric cluster, *Automatic* mode is the only available mode. For more information, see [Service Fabric application upgrades](../service-fabric/service-fabric-application-upgrade.md).

### Manual upgrade policy mode

With a manual upgrade policy mode, you choose when to update the scale set instances. Nothing happens automatically to the existing virtual machines when changes occur to the scale set model. New instances added to the scale set use the most update-to-date model available. 

:::image type="content" source="./media/upgrade-policy/manual-upgrade.png" alt-text="Diagram that shows a high level diagram of what happens when using a manual upgrade policy.":::

Manual upgrade policy mode is best suited for workloads where you require more control over when and how instances are updated.  

### Rolling upgrade policy mode


With a rolling upgrade policy mode, the scale set performs updates in batches. You also get more control over the upgrades with settings like batch size, max healthy percentage, prioritizing unhealthy instances and enabling upgrades across availability zones. 

:::image type="content" source="./media/upgrade-policy/rolling-upgrade.png" alt-text="Diagram that shows a high level diagram of what happens when using a rolling upgrade policy.":::

Rolling upgrade policy mode is best suited for production workloads that require a set number of instances always be available. Rolling upgrades is safest way to upgrade instances to the latest model without compromising availability and uptime. 

When using a rolling upgrade policy mode on Virtual Machine Scale Sets with Flexible Orchestration, the scale set must also use the [Application Health Extension](virtual-machine-scale-sets-health-extension.md) to monitor application health.

When using a rolling upgrade policy mode on Virtual Machine Scale Sets with Uniform Orchestration, the scale set must also have a [health probe](/azure/load-balancer/load-balancer-custom-probe-overview) or use the [Application Health Extension](virtual-machine-scale-sets-health-extension.md) to monitor application health. 

## Upgrades that require a restart, reimage, or redeploy
Some upgrades require a virtual machine restart while others can be completed without disrupting scale set instances. Updates that require restarting, reimaging or redeploying the virtual machine instance include: 

- Password or SSH keys updates
- Custom Data changes
- Image Reference updates
- Virtual machine size changes
- Adding Availability Zones
- Proximity Placement Group changes

> [!NOTE]
> While Password and Custom Data changes can be made without a restart, in order for the upgrades to be applied to the virtual machine instances, you must reimage the virtual machine. For more information, see [Reimage a virtual machine](virtual-machine-scale-sets-reimage-virtual-machine.md)

## Next steps
Learn how to [set the upgrade policy mode](virtual-machine-scale-sets-set-upgrade-policy.md) of your Virtual Machine Scale Set.
