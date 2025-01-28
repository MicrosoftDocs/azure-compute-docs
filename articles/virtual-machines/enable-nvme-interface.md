---
title: Supported OS Images
description: Get a list of supported operating system images for remote NVMe.
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 06/25/2024
ms.topic: how-to
ms.custom: template-how-to-pattern
---

# Supported OS images for remote NVMe

The following lists provide up-to-date information on which OS images are tagged as supported for remote NVM Express (NVMe).

> [!IMPORTANT]
> Only Gen2 VM images with security type "Standard" support NVMe. For more information, see [FAQ for NVMe](/azure/virtual-machines/enable-nvme-faqs#will-generation-1-vms-be-supported-with-nvme-disks-). Older operating systems cannot support NVMe due to OS driver limitations. If an older version of the OS is not included in the list below, it means that NVMe support is not present for that OS.

For specifics about which virtual machine (VM) generations support which storage types, check the [documentation about VM sizes in Azure](/azure/virtual-machines/sizes).

For more information about enabling the NVMe interface on virtual machines created in Azure, review the [FAQ for remote NVMe disks](/azure/virtual-machines/enable-nvme-remote-faqs).

## Supported Linux OS images

- Almalinux 8.x
- Almalinux 9.x
- Debian 11
- Debian 12
- Oracle Linux 7.9
- Oracle Linux 8.5 and newer
- Oracle Linux 9.0 and newer
- RHEL 7.9
- RHEL 8.6 and newer
- RHEL 9.0 and newer
- SLES 15 SP4 and newer
- SLES-for-SAP 15 SP3 and newer
- Ubuntu 18.04
- Ubuntu 20.04
- Ubuntu 22.04

## Supported Windows OS images
- Windows Server 2019*
- Windows Server 2022
- Windows 10
- Windows 11

*Shared Disks on NVME are not compatible with Windows Server 2019

To download an image, go to [Azure Marketplace](https://ms.portal.azure.com/#view/Microsoft_Azure_Marketplace/MarketplaceOffersBlade/selectedMenuItemId/home).
