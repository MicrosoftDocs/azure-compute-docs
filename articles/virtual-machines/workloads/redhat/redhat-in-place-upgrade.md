---
title: In-place upgrade of Red Hat Enterprise Linux images on Azure
description: Learn how to do an in-place upgrade from Red Hat Enterprise images to the latest version.
ms.author: cynthn
author: cynthn
ms.service: azure-virtual-machines
ms.subservice: redhat
ms.custom: linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 12/01/2025
ms.update-cycle: 1095-days
# Customer intent: "As a system administrator managing RHEL VMs on Azure, I want to perform in-place upgrades from RHEL 7 to RHEL 8, and RHEL 8 to RHEL 9, so that I can keep the operating systems up to date while retaining applications and configurations."
---

# Red Hat Enterprise Linux in-place upgrades

**Applies to:** :heavy_check_mark: Linux VMs


During an in-place upgrade, the earlier RHEL OS major version will be replaced with the new RHEL OS major version without removing the earlier version first. The installed applications and utilities, along with the configurations and preferences, are incorporated into the new version.

> [!CAUTION]
> Following the process in this article will cause a disconnection between the data plane and the **[control plane](/azure/architecture/guide/multitenant/considerations/control-planes)** of the virtual machine (VM). Azure capabilities such as **[Auto guest patching](/azure/virtual-machines/automatic-vm-guest-patching)**, **[Auto OS image upgrades](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade)**, **[Hotpatching](/windows-server/get-started/hotpatch?toc=%2Fazure%2Fvirtual-machines%2Ftoc.json)**, and **[Azure Update Manager](/azure/update-manager/overview)** won't be available. To utilize these features, it's recommended to create a new VM using your preferred operating system instead of performing an in-place upgrade.

## Prerequisites
Before you begin:
- Offerings of SQL Server on Red Hat Enterprise Linux don't support in-place upgrades on Azure.
- Take a snapshot of the image before you start the upgrade as a precaution.

## Upgrade RHEL VMs
In general, Red Hat only supports upgrading from version to the next; you cannot skip versions. See the Red Hat's documentation on in-place upgrades:

[Upgrading from RHEL 7 to RHEL 8.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index)

[Upgrading from RHEL 8 to RHEL 9.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/upgrading_from_rhel_8_to_rhel_9/index)

[Upgrading from RHEL 9 to RHEL 10.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/10/html/upgrading_from_rhel_9_to_rhel_10/index)

[Upgrading from RHEL 7 SAP to RHEL 8 SAP.](https://access.redhat.com/solutions/5154031)

[Upgrading from RHEL 8 SAP to RHEL 9 SAP.](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_for_sap_solutions/9/html/upgrading_sap_environments_from_rhel_8_to_rhel_9/index)


## Next steps
* Learn more about [Red Hat images in Azure](./redhat-images.md).
* Learn more about [Red Hat update infrastructure](./redhat-rhui.md).
* Learn more about the [RHEL BYOS offer](./byos.md).
* To learn more about the Red Hat in-place upgrade processes, see [Upgrading from RHEL 7 TO RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index) in the Red Hat documentation.
* To learn more about Red Hat support policies for all versions of RHEL, see [Red Hat Enterprise Linux life cycle](https://access.redhat.com/support/policy/updates/errata) in the Red Hat documentation.
