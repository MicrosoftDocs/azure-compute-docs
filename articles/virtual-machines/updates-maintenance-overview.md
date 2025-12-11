---
title: Guest updates and host maintenance overview
description: Learn about the updates and maintenance options that are available with virtual machines in Azure.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machines
ms.topic: overview
ms.date: 03/20/2024
ms.reviewer: cynthn
# Customer intent: "As a cloud administrator, I want to manage guest updates and host maintenance for Azure virtual machines so that I can ensure reliability, security, and performance of our applications while minimizing downtime during updates."
---

# Guest updates and host maintenance overview

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article provides an overview of guest update and host maintenance options for Azure virtual machines (VMs).

Azure periodically updates its infrastructure to improve reliability, performance, and security or to launch new features. Most updates are transparent to you. To incorporate these updates, Azure uses a robust infrastructure that includes region pairs and availability zones combined with multiple tools and features.

Azure also offers you the ability to control updates on Azure machines like virtual machine scale sets, host machines, guest VMs, and extensions attached to VMs. This control is possible through maintenance configurations, which you can use to set up recurring schedules for times when you want available platform updates to occur.

With Azure infrastructure updates, you can:

- Upgrade network components.
- Decommission hardware on networks.
- Patch software components in hosting environments.
- Update guest operating system (OS) software on VMs.

To perform these updates, use the tools that are available in Azure.

The maintenance platform aims to provide you with a *unified maintenance experience* for all Azure resources that are affected during maintenance. The maintenance experience is available for host ([Azure Dedicated Host](dedicated-hosts.md) and [isolated](isolation.md) VMs) and guest (VMs and Azure Arc VMs) resources, Azure Kubernetes Service, Salesforce Marketing Cloud (SFMC), and network gateways (Azure VPN Gateway, Azure ExpressRoute, and virtual network gateways in Azure). To deploy, use either the Azure portal, PowerShell, or the Azure CLI.

[Maintenance control](maintenance-configurations.md) provides you with an option to skip or defer certain updates and schedule them only during your preferred maintenance window. In exceptional circumstances where a high-severity security issue arises that could potentially compromise customer services, Azure reserves the right to bypass these schedules to implement urgent changes. This measure is taken solely to help ensure the safety and integrity of customer services. It's employed only when the updates have no effect on customer resources. Such instances are infrequent and are invoked only as a last resort to safeguard resources.

## Host maintenance

Host maintenance is performed on the physical hosts where VMs are located and is usually transparent to you. Some updates might have an effect on hosts that you can tolerate. During these updates, the VMs that are allocated on the hosts might freeze (*non-rebootful updates*), reboot (*rebootful updates*), or migrate live to another updated host. Azure chooses the update mechanism that affects your VMs the least.

### Dedicated hosts, isolated VMs, and shared hosts
 
The host maintenance experience is available for [dedicated](dedicated-hosts.md) hosts, [isolated](isolation.md) VMs, and shared hosts. Dedicated hosts are hosts in which one customer owns all VMs. Shared hosts are hosts in which VMs from multiple customers reside together. Isolated VMs are large machines that are isolated to a specific hardware type and dedicated to a single customer. 

On [dedicated](dedicated-hosts.md) hosts, you have the host maintenance experience available for all updates. You can opt into maintenance control and schedule a maintenance window based on your needs within 35 days from the last maintenance date. [Isolated](isolation.md) VMs have the maintenance control experience available like dedicated hosts.

You can use [maintenance control](maintenance-configurations.md) to:

- Apply all updates together.
- Wait up to 35 days to apply updates for host machines.
- Set up a maintenance schedule or use Azure Functions to automate platform updates.
- Configure maintenance across subscriptions and resource groups.

On shared hosts, the maintenance experience is available for rebootful updates or high-impact updates. Currently, the maintenance control experience isn't available for updates that take less than 30 seconds.

### Maintenance notifications
 
Azure provides notifications before, during, and after maintenance operations. [Scheduled events](./windows/scheduled-events.md) provide notifications before an event starts and while it's in progress so that your application can react automatically. [Flash Health events](flash-overview.md) provide information that you can consume to analyze alerts and trends in VM availability for reporting and root cause analysis.
 
#### Scheduled events

[Scheduled events](./windows/scheduled-events.md) provide advance notification of upcoming availability impacts so that you can prepare your application ahead of time. They're delivered directly to the affected VM and to all VMs in the same placement group for automated resilience. For information on scheduled events, see [Scheduled events for Windows VMs](./windows/scheduled-events.md) and [Scheduled events for Linux](./linux/scheduled-events.md).

#### Flash Health events

[Flash Health events](flash-overview.md) provide near real-time information about past availability impacts so that you can react to events and easily mitigate incidents. Flash information is available in Azure Monitor, Azure Resource Graph, or Azure Event Grid to integrate with your systems and processes.

## Guest updates

This section explains guest update options.

### OS image upgrade

[Automatic OS upgrades](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md?context=/azure/virtual-machines/context/context) are available for virtual machine scale sets. An upgrade works by replacing the OS disk of a VM with a new disk that was created by using the latest image version. Any configured extensions and custom data scripts are run on the OS disk, while data disks are retained. To minimize the application downtime, upgrades take place in batches. No more than 20% of the scale set upgrades at any time.

Maintenance control is also available for OS image upgrades. You can opt in to this experience by using maintenance configurations to schedule when to apply these image upgrades. To use this experience, scale sets need to have automatic OS upgrades enabled. You can schedule recurrence for up to a week (seven days). A minimum of five hours is required for the maintenance window.

### Guest VM patching

[Automatic VM guest patching](automatic-vm-guest-patching.md) is integrated with Azure Update Manager. You can save recurring deployment schedules to install updates for your Windows Server and Linux machines in Azure, in on-premises environments, and in other cloud environments that are connected by using Azure Arc-enabled servers.

### Guest extension upgrades

[Automatic Extension Upgrade](automatic-extension-upgrade.md) is available for Azure Virtual Machines and Azure Virtual Machine Scale Sets. When Automatic Extension Upgrade is enabled on a VM or scale set, the extension is automatically upgraded whenever the extension publisher releases a new version for that extension. The extension upgrade process replaces the existing extension version on a VM with the new version of the same extension. 

The health of the VM is monitored after the new extension is installed. If the VM isn't in a healthy state within five minutes of the upgrade completion, the extension version rolls back to the previous version. Maintenance control on extensions is currently available only via the CLI and PowerShell. You can schedule recurrence for up to a week (seven days). A minimum of five hours is required for the maintenance window.

### Hotpatch

[Hotpatching](/azure/automanage/automanage-hotpatch?context=/azure/virtual-machines/context/context) is a new way to install updates on new Windows Server Azure Edition VMs that doesn't require a reboot after installation. Hotpatching for Windows Server Azure Edition VMs has the following benefits:

- Lower workload effect with fewer reboots.
- Faster deployment of updates because the packages are smaller, install faster, and have easier patch orchestration with Update Manager.
- Better protection because the hotpatch update packages are scoped to Windows security updates that install faster without rebooting.

### Azure update management

You can use [Update Manager in Azure Automation](/azure/automation/update-management/overview?context=/azure/virtual-machines/context/context) to manage system updates for your Windows and Linux VMs in Azure, in on-premises environments, and in other cloud environments. You can quickly assess the status of available updates on all agent machines and manage the process of installing required updates for servers.

### Update Manager

[Update Manager](/azure/update-center/overview) is a unified service in Azure that helps you manage and govern updates (Windows and Linux), both on-premises and with other cloud platforms, across hybrid environments from a single dashboard. The new functionality provides a native and an out-of-the-box experience with granular access controls. You have the flexibility to create schedules or act now and the ability to check updates automatically. The enhanced functionality ensures that administrators have visibility into the health of all systems in the environment. For more information, see [Key benefits](/azure/update-center/overview#key-benefits).

## Related content

For more ways to increase the uptime of your applications and services, see [Availability options for Azure virtual machines](availability.md).
