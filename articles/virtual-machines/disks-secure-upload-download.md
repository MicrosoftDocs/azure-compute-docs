---
title: Secure downloads and uploads of Azure managed disks
description: Protect your Azure managed disks by using RBAC roles to limit who has the ability to import or export with disks
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 01/07/2026
ms.author: rogarana
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a cloud engineer, I want to configure shared disks for Azure managed disks, so that I can enable simultaneous access from multiple virtual machines to support clustered applications.
---

# Secure downloads and uploads of Azure managed disks

If you use [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can also use it to restrict uploads and downloads of Azure managed disks. When a user attempts to upload or download a disk, Azure validates the identity of the requesting user in Microsoft Entra ID, and confirms that user has the required permissions. If a user doesn't have the required permissions, they can't upload or download managed disks.

At a higher level, a system administrator can set a policy at the Azure account or subscription level, to ensure that all disks and snapshots must use Microsoft Entra ID for uploads or downloads. If you have any questions on securing uploads or downloads by using Microsoft Entra ID, reach out to: azuredisks@microsoft .com.

## Restrictions
[!INCLUDE [disks-azure-ad-upload-download-restrictions](includes/disks-azure-ad-upload-download-restrictions.md)]

## Prerequisites
[!INCLUDE [disks-azure-ad-upload-download-prereqs](includes/disks-azure-ad-upload-download-prereqs.md)]

## Assign RBAC role

To access managed disks secured with Microsoft Entra ID, requesting users must have either the [Data Operator for Managed Disks](/azure/role-based-access-control/built-in-roles#data-operator-for-managed-disks) role, or a [custom role](/azure/role-based-access-control/custom-roles-portal) with the following permissions: 

- **Microsoft.Compute/disks/download/action**
- **Microsoft.Compute/disks/upload/action**
- **Microsoft.Compute/snapshots/download/action**
- **Microsoft.Compute/snapshots/upload/action**

For detailed steps on assigning a role, see the following articles for [portal](/azure/role-based-access-control/role-assignments-portal), [PowerShell](/azure/role-based-access-control/role-assignments-powershell), or [CLI](/azure/role-based-access-control/role-assignments-cli). To create or update a custom role, see the following articles for [portal](/azure/role-based-access-control/custom-roles-portal), [PowerShell](/azure/role-based-access-control/role-assignments-powershell), or [CLI](/azure/role-based-access-control/role-assignments-cli).

## Restrict access to an individual disk

To restrict access to an individual disk, enable **data access authentication mode** on that disk. 

# [Portal](#tab/azure-portal)

You can either enable it when creating the disk, or you can enable it on the **Disk Export** page under **Settings** for existing disks.

:::image type="content" source="./media/disks-upload-download-portal/disks-data-access-auth-mode.png" alt-text="Screenshot of a disk's data access authentication mode checkbox, tick the checkbox to restrict access to the disk, and save your changes." lightbox="./media/disks-upload-download-portal/disks-data-access-auth-mode.png":::

# [PowerShell](#tab/azure-powershell)

Set `dataAccessAuthMode` to `"AzureActiveDirectory"` on your disk to download it when it's secured. Use the following script to update an existing disk. Replace the values for `-ResourceGroupName` and `-DiskName` before running the script:

```azurepowershell
New-AzDiskUpdateConfig -DataAccessAuthMode "AzureActiveDirectory" | Update-AzDisk -ResourceGroupName 'yourResourceGroupName' -DiskName 'yourDiskName"
```

# [Azure CLI](#tab/azure-cli)

Set `dataAccessAuthMode` to `"AzureActiveDirectory"` on your disk to download it when it's secured. Use the following script to update an existing disk. Replace the values for `--resource-group` and `--Name` before running the script:

```azurecli
az disk update --name yourDiskName --resource-group yourResourceGroup --data-access-auth-mode AzureActiveDirectory
```

---

## Assign Azure policy

You can also assign an Azure policy with a remediation task. A policy with a remediation task continously audits your resources and notifies you when any of them don't comply. The built-in policy definition you'd assign is **Protect your data with authentication requirements when exporting or uploading to a disk or snapshot**. To learn how to assign an Azure policy see the [Azure portal](/azure/governance/policy/assign-policy-portal), [Azure CLI](/azure/governance/policy/assign-policy-azurecli), or [Azure PowerShell module](/azure/governance/policy/assign-policy-powershell) articles.

## Next steps

- [Upload a VHD to Azure or copy a managed disk to another region - Azure CLI](linux/disks-upload-vhd-to-managed-disk-cli.md)
- [Upload a VHD to Azure or copy a managed disk to another region - Azure PowerShell](windows/disks-upload-vhd-to-managed-disk-powershell.md)
- [Download a Linux VHD from Azure](linux/download-vhd.md)
- [Download a Windows VHD from Azure](windows/download-vhd.md)
