---
title: Red Hat Update Infrastructure | Microsoft Docs
description: Learn about Red Hat Update Infrastructure for on-demand Red Hat Enterprise Linux instances in Microsoft Azure.
author: ju-shim
ms.service: azure-virtual-machines
ms.subservice: redhat
ms.custom: linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.date: 10/28/2024
ms.reviewer: cynthn
ms.author: jushiman
# Customer intent: As a system administrator managing Red Hat Enterprise Linux VMs in Azure, I want to utilize the Red Hat Update Infrastructure, so that I can efficiently access and manage software updates for my PAYG instances without incurring double billing or configuration issues.
---
# Red Hat Update Infrastructure for on-demand Red Hat Enterprise Linux VMs in Azure

**Applies to:** :heavy_check_mark: Linux VMs

[Red Hat Update Infrastructure (RHUI)](https://access.redhat.com/products/red-hat-update-infrastructure) allows cloud providers, such as Azure, to:

- Mirror Red Hat-hosted repository content
- Create custom repositories with Azure-specific content
- Make the content available to end-user Virtual Machines (VMs)

Red Hat Enterprise Linux (RHEL) Pay-As-You-Go (PAYG) images come preconfigured to access Azure RHUI. No other configuration is needed. To get the latest updates, run `sudo yum update` after your RHEL instance is ready. This service is included as part of the RHEL PAYG software fees. For more information on RHEL images in Azure, including publishing and retention policies, see [Overview of Red Hat Enterprise Linux images in Azure](./redhat-images.md).

For more information on Red Hat support policies for all versions of RHEL, see [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata).

> [!IMPORTANT]
> RHUI is intended only for pay-as-you-go (PAYG) images. For golden images, also known as bring your own subscription (BYOS), the system needs to be attached to Red Hat Subscription Manager (RHSM) or Satellite in order to receive updates. For more information, see [How to register and subscribe a RHEL system](https://access.redhat.com/solutions/253273).

## Important information about Azure RHUI

- Azure RHUI is the update infrastructure that supports all RHEL PAYG VMs created in Azure. This infrastructure doesn't prevent you from registering your PAYG RHEL VMs with Subscription Manager, Satellite, or another source of updates. Registering with a different source with a PAYG VM results in indirect double-billing. See the following point for details.

- Access to the Azure-hosted RHUI is included in the RHEL PAYG image price. Unregistering a PAYG RHEL VM from the Azure-hosted RHUI doesn't convert the virtual machine into a BYOS type of VM. If you register the same VM with another source of updates, you might incur *indirect* double charges. You're charged the first time for the Azure RHEL software fee. You're charged the second time for Red Hat subscriptions that were purchased previously. If you consistently need to use an update infrastructure other than Azure-hosted RHUI, consider registering to use [RHEL BYOS images](./byos.md).

- RHEL SAP PAYG images in Azure are connected to dedicated RHUI channels that remain on the specific RHEL minor version as required for SAP certification. RHEL SAP PAYG images in Azure include RHEL for SAP, RHEL for SAP HANA, and RHEL for SAP Business Applications.

- Access to Azure-hosted RHUI is limited to the VMs within the [Azure datacenter IP ranges](https://www.microsoft.com/download/details.aspx?id=56519). If you proxy all VM traffic by using an on-premises network infrastructure, you might need to set up user-defined routes for the RHEL PAYG VMs to access the Azure RHUI. If that is the case, user-defined routes need to be added for *all* RHUI IP addresses.

## Image update behavior

The Red Hat images provided in Azure Marketplace are connected by default to one of two different types of life-cycle repositories:

- Non-EUS: Has the latest available software published by Red Hat for their particular Red Hat Enterprise Linux (RHEL) repositories.

- Extended Update Support (EUS): Updates for a specific RHEL minor release.

> [!NOTE]
> For more information on RHEL EUS, see [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata) and [Red Hat Enterprise Linux Extended Update Support Overview](https://access.redhat.com/articles/rhel-eus).

The packages contained in the Red Hat Update Infrastructure repositories are published and maintained by Red Hat. Extra packages to support custom Azure services are published in independent repositories maintained by Microsoft.

For a full image list, run `az vm image list --offer RHEL --all -p RedHat --output table` using the Azure CLI.

### Images connected to non-EUS repositories

For RHEL VM images connected to non-EUS repositories, running `sudo yum update` will upgrade to the latest RHEL minor version. For example, if you provision a VM from a RHEL 8.4 PAYG image and run `sudo yum update`, you end up with a VM that has installed all updates through the latest minor version in the RHEL8 family.

Images that are connected to non-EUS repositories don't contain a minor version number in the SKU. The SKU is the third element in the image name. For example, all of the following images come attached to non-EUS repositories:

```output
RedHat:RHEL:7-LVM:7.9.2023032012
RedHat:RHEL:8-LVM:8.7.2023022813
RedHat:RHEL:9-lvm:9.1.2022112101
RedHat:rhel-raw:7-raw:7.9.2022040605
RedHat:rhel-raw:8-raw:8.6.2022052413
RedHat:rhel-raw:9-raw:9.1.2022112101
```

The SKUs are either X-LVM or X-RAW. The minor version is indicated in the version of these images, which is the fourth element in the name.

### Images connected to EUS repositories

If you provision a VM from a RHEL image that is connected to EUS repositories, it isn't upgraded to the latest RHEL minor version when you run `sudo yum update`. This situation happens because the images connected to EUS repositories are also version-locked to their specific minor version.

Images connected to EUS repositories contain a minor version number in the SKU. For example, all of the following images come attached to EUS repositories:

```output
RedHat:RHEL:7.7:7.7.2022051301
RedHat:RHEL:8_4:latest
RedHat:RHEL:9_0:9.0.2023061412
```

## RHEL EUS and version-locking RHEL VMs

Extended Update Support (EUS) repositories are available to customers who might want to lock their RHEL VMs to a certain RHEL minor release after provisioning the VM. You can version-lock your RHEL VM to a specific minor version by updating the repositories to point to the Extended Update Support repositories. You can also undo the EUS version-locking operation.

> [!NOTE]
> The RHEL Extras channel does not follow the EUS lifecycle. This means that if you install a package from the RHEL Extras channel, it will not be specific to the EUS release you are on. Red Hat does not support installing content from the RHEL Extras channel while on an EUS release. For more information, see [Red Hat Enterprise Linux Extras Product Life Cycle](https://access.redhat.com/support/policy/updates/extras/).

Support for RHEL 7 EUS ended in June 30, 2024. Support for RHEL 8 EUS ended May 31, 2025. For more information, see [Red Hat Enterprise Linux Extended Maintenance](https://access.redhat.com/support/policy/updates/errata/#Long_Support).

- RHEL 9.4 EUS support ends April 30, 2026
- RHEL 9.6 EUS support ends May 31, 2027

---
### Switch a RHEL Server to EUS Repositories.

>[!NOTE]
> Support for RHEL 7 EUS ended on June 30, 2024.
> Support for RHEL 8 EUS ended on May 31, 2025.
> It isn't recommended to switch to EUS repositories on RHEL 7 or 8 anymore.

Use the following procedure to lock a RHEL VM to a particular minor release.

>[!NOTE]
>This procedure only applies for RHEL versions for which EUS is available. At the time of writing the list of versions includes RHEL 9.4, 9.6, and 10.0. For more information, see [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata#Extended_Update_Support).

1. Save your RHEL major version in a variable for use in the commands below.

   ```bash
   major_version=$(rpm -q --queryformat '%{RELEASE}' rpm | grep -o "[0-9]*\(_[0-9]*\)\?\$" | cut -d "_" -f 1)
   echo $major_version
   ```
   
1. Disable non-EUS repositories.

   ```bash
   sudo dnf --disablerepo='*' remove "rhui-azure-rhel${major_version}"
   ```

1. Create a `config` file by using this command or a text editor:

   ```bash
   cat <<EOF > rhel${major_version}-eus.config
   [rhui-microsoft-azure-rhel${major_version}]
   name=Microsoft Azure RPMs for Red Hat Enterprise Linux ${major_version} (rhel${major_version}-eus)
   baseurl=https://rhui4-1.microsoft.com/pulp/repos/unprotected/microsoft-azure-rhel${major_version}-eus
   enabled=1
   gpgcheck=1
   sslverify=1
   gpgkey=/etc/pki/rpm-gpg/RPM-GPG-KEY-microsoft-azure-release
   EOF
   ```

1. Add non-EUS repository.

   ```bash
   sudo dnf --config rhel${major_version}-eus.config install rhui-azure-rhel${major_version}-eus
   ```

1. Lock the `releasever` level, at the time of writing it has to be one of 9.4, 9.6, or 10.0.

   ```bash
   sudo sh -c 'echo 9.6 > /etc/dnf/vars/releasever'
   ```

   If there are permission issues to access the `releasever`, you can edit the file using a text editor, add the image version details, and save the file.

   >[!NOTE]
   >This instruction locks the RHEL minor release to the current minor release. Enter a specific minor release if you're looking to upgrade and lock to a later minor release that isn't the latest. For example, `echo 9.6 > /etc/yum/vars/releasever` locks your RHEL version to RHEL 9.6.

1. Update your RHEL VM.

   ```bash
   sudo dnf update
   ```
---
### Switch a RHEL Server to non-EUS Repositories.

To remove the version lock, use the following commands.

1. Remove the `releasever` file.

   ```bash
   sudo rm /etc/dnf/vars/releasever
   ```

1. Save your RHEL major version in a variable for use in the commands below.

   ```bash
   major_version=$(rpm -q --queryformat '%{RELEASE}' rpm | grep -o "[0-9]*\(_[0-9]*\)\?\$" | cut -d "_" -f 1)
   echo $major_version
   ```
   
1. Disable EUS repositories.

   ```bash
   sudo dnf --disablerepo='*' remove "rhui-azure-rhel${major_version}-eus"
   ```

1. Create a `config` file by using this command or a text editor:

   ```bash
   cat <<EOF > rhel${major_version}.config
   [rhui-microsoft-azure-rhel${major_version}]
   name=Microsoft Azure RPMs for Red Hat Enterprise Linux ${major_version}
   baseurl=https://rhui4-1.microsoft.com/pulp/repos/unprotected/microsoft-azure-rhel${major_version}
   enabled=1
   gpgcheck=1
   sslverify=1
   gpgkey=/etc/pki/rpm-gpg/RPM-GPG-KEY-microsoft-azure-release
   EOF
   ```
   >[!NOTE]
   >You might encounter an error that gpgkey must be in either http, ftp , file or https and not "". In that case, try editing the config file to use: `gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-microsoft-azure-release`.
1. Add non-EUS repository.

   ```bash
   sudo dnf --config rhel${major_version}.config install rhui-azure-rhel${major_version}
   ```

1. Update your RHEL VM.

   ```bash
   sudo dnf update
   ```

---
## The IPs for the RHUI content delivery servers

RHUI is available in all regions where RHEL on-demand images are available. Availability currently includes all public regions listed in the [Azure status dashboard](https://azure.microsoft.com/status/), Azure US Government, and Microsoft Azure Germany regions.

If you're using a network configuration (custom Firewall or user-defined routes (UDR) configuration) to further restrict `https` access from RHEL PAYG VMs, make sure the following IPs are allowed for `dnf update` to work depending on your environment:

```output
# Azure Global - RHUI 4
West Europe - 52.136.197.163
South Central US - 20.225.226.182
East US - 52.142.4.99
Australia East - 20.248.180.252
Southeast Asia - 20.24.186.80
```

---
## Azure RHUI Infrastructure

### Update expired RHUI client certificate on a VM

If you experience RHUI certificate issues from your Azure RHEL PAYG VM, see [Troubleshoot RHUI certificate issues in Azure](/troubleshoot/azure/virtual-machines/troubleshoot-linux-rhui-certificate-issues).

### Troubleshoot connection problems to Azure RHUI

If you experience problems connecting to Azure RHUI from your Azure RHEL PAYG VM, follow these steps:

1. Inspect the VM configuration for the Azure RHUI endpoint:

   - Check whether the `/etc/yum.repos.d/rh-cloud.repo` file contains a reference to `rhui-[1-4].microsoft.com` in the `baseurl` of the `[rhui-microsoft-azure-rhel*]` section of the file. If it does, you're using the new Azure RHUI.

   - If the reference points to a location with the following pattern, `mirrorlist.*cds[1-4].cloudapp.net`, a configuration update is required. You're using the old VM snapshot, and you need to update it to point to the new Azure RHUI.

1. Verify that access to Azure-hosted RHUI is limited to VMs within the [Azure datacenter IP ranges](https://www.microsoft.com/download/details.aspx?id=56519).

1. If you're still having issues using the new configuration and the VM connects from the Azure IP range, file a support case with Microsoft or Red Hat.

### Infrastructure update

In September 2016, Azure deployed an updated Azure RHUI. In April 2017, the old Azure RHUI was shut down. If you have been using the RHEL PAYG images or their snapshots from September 2016 or later, you're automatically connecting to the new Azure RHUI. If, however, you have older snapshots on your VMs, you need to manually update their configuration to access the Azure RHUI as described in a following section.

The new Azure RHUI servers are deployed with [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). In Traffic Manager, any VM can use a single endpoint, rhui-1.microsoft.com, and rhui4-1.microsoft.com, regardless of region.

### Manual update procedure to use the Azure RHUI servers

This procedure is provided for reference only. RHEL PAYG images already have the correct configuration to connect to Azure RHUI. To manually update the configuration to use the Azure RHUI servers, complete the following steps:

1. Save your RHEL major version in a variable for use in the commands below.

   ```bash
   major_version=$(rpm -q --queryformat '%{RELEASE}' rpm | grep -o "[0-9]*\(_[0-9]*\)\?\$" | cut -d "_" -f 1)
   echo $major_version
   ```
   
1. Create a `config` file by using this command or a text editor:

   ```bash
   cat <<EOF > rhel${major_version}.config
   [rhui-microsoft-azure-rhel${major_version}]
   name=Microsoft Azure RPMs for Red Hat Enterprise Linux ${major_version}
   baseurl=https://rhui4-1.microsoft.com/pulp/repos/unprotected/microsoft-azure-rhel${major_version}
   enabled=1
   gpgcheck=1
   sslverify=1
   gpgkey=/etc/pki/rpm-gpg/RPM-GPG-KEY-microsoft-azure-release
   EOF
   ```

1. Install the latest `rhui-azure` package.

   ```bash
   sudo dnf --config rhel${major_version}.config install rhui-azure-rhel${major_version}
   ```

1. Update your VM.

   ```bash
   sudo dnf update
   ```

## Next steps

- To create a Red Hat Enterprise Linux VM from an Azure Marketplace PAYG image and to use Azure-hosted RHUI, go to the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.rhel-20190605).
- To learn more about the Red Hat images in Azure, see [Overview of Red Hat Enterprise Linux images](./redhat-images.md).
- To learn more about Red Hat's support policies, see [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata).
