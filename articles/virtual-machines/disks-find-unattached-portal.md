---
title: "Find and delete unattached Azure managed and unmanaged disks - Azure portal"
description: Find unattached managed disks in the Azure portal and use scripts to find unattached unmanaged disks.
author: roygara
ms.author: rogarana
ms.date: 09/16/2026
ms.service: azure-disk-storage
ms.topic: how-to
ms.custom:
  - ge-structured-content-pilot
---

# Find and delete unattached Azure managed and unmanaged disks - Azure portal

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

When you delete a virtual machine (VM) in Azure, by default, any disks that are attached to the VM aren't deleted. This helps to prevent data loss due to the unintentional deletion of VMs. After a VM is deleted, you will continue to pay for unattached disks. This article shows you how to find and delete unattached managed disks by using the Azure portal and links to scripts for finding and deleting unattached unmanaged disks. Deletions are permanent. You can't recover data after you delete a disk.

Unmanaged disks are VHD files that are stored as [page blobs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) in [Azure storage accounts](/azure/storage/common/storage-account-overview).

If you have unmanaged disks that aren't attached to a VM, no longer need the data on them, and would like to delete them, the best way to find and delete them is to use the scripts in either the [Azure CLI](linux/find-unattached-disks.md#unmanaged-disks-find-and-delete-unattached-disks) or [Azure PowerShell](windows/find-unattached-disks.md#unmanaged-disks-find-and-delete-unattached-disks) articles.

## Find and delete unattached managed disks in the Azure portal

If you have unattached managed disks and no longer need the data on them, the following process explains how to find them from the Azure portal:

> [!WARNING]
> Deleting an unattached managed disk is permanent. You can't recover its data after deletion.

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Search for and select **Disks**.
   
     On the **Disks** blade, you are presented with a list of all your disks.

1. Select the disk you'd like to delete, this brings you to the individual disk's blade.

1. On the individual disk's blade, confirm the disk state is unattached, then select **Delete**.
   
  :::image type="content" source="media/disks-find-unattached-portal/delete-managed-disk-unattached.png" alt-text="Screenshot of an unattached managed disk in the Azure portal with Disk state set to Unattached and Delete highlighted.":::

## Related content

- [Use the Azure CLI to find unattached disks](linux/find-unattached-disks.md)
- [Use the Azure PowerShell module to find unattached disks](windows/find-unattached-disks.md)
- [Identify Orphaned Disks Using PowerShell](/archive/blogs/ukplatforms/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell)
