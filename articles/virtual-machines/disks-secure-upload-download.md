---
title: Secure Azure managed disk downloads and uploads
description: Learn how to secure managed disk downloads and uploads with Microsoft Entra ID and RBAC roles. Control access and protect your data with authentication requirements.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/17/2026
ms.author: rogarana
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a cloud engineer, I want to secure managed disk downloads and uploads with Microsoft Entra ID and RBAC roles, so that I can control who has access to import or export disk data and protect my data from unauthorized access.
---

# Secure downloads and uploads of Azure managed disks

If you use [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can also use it to restrict uploads and downloads of Azure managed disks. When a user tries to upload or download a disk, Azure validates the identity of the requesting user in Microsoft Entra ID, and confirms that user has the required permissions. If a user doesn't have the required permissions, they can't upload or download managed disks.

At a higher level, a system administrator can set a policy at the Azure account or subscription level, to ensure that all disks and snapshots must use Microsoft Entra ID for uploads or downloads. If you have any questions on securing uploads or downloads by using Microsoft Entra ID, reach out to: azuredisks@microsoft .com.

## Restrictions
[!INCLUDE [disks-azure-ad-upload-download-restrictions](includes/disks-azure-ad-upload-download-restrictions.md)]

## Prerequisites
[!INCLUDE [disks-azure-ad-upload-download-prereqs](includes/disks-azure-ad-upload-download-prereqs.md)]

## Assign an Azure RBAC role

To access managed disks secured by using Microsoft Entra ID, users must have either the [Data Operator for managed disks](/azure/role-based-access-control/built-in-roles#data-operator-for-managed-disks) role assigned through Azure role-based access control (Azure RBAC) or a [custom role](/azure/role-based-access-control/custom-roles-portal) with the following permissions:

- `Microsoft.Compute/disks/download/action`
- `Microsoft.Compute/disks/upload/action`
- `Microsoft.Compute/snapshots/download/action`
- `Microsoft.Compute/snapshots/upload/action`

For detailed steps on assigning a role, see the following articles for [Azure portal](/azure/role-based-access-control/role-assignments-portal), [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell), or [Azure CLI](/azure/role-based-access-control/role-assignments-cli). To create or update a custom role, see the following articles for [Azure portal](/azure/role-based-access-control/custom-roles-portal), [Azure PowerShell](/azure/role-based-access-control/custom-roles-powershell), or [Azure CLI](/azure/role-based-access-control/custom-roles-cli).

## Restrict access to an individual disk

To restrict access to an individual disk, enable **data access authentication mode** on that disk. 

# [Azure portal](#tab/azure-portal)

You can enable this setting when creating the disk, or you can enable it on the **Disk Export** page under **Settings** for existing disks.

:::image type="content" source="./media/disks-upload-download-portal/disks-data-access-auth-mode.png" alt-text="Screenshot of the Disk Export pane with Enable data access authentication mode selected and Save highlighted." lightbox="./media/disks-upload-download-portal/disks-data-access-auth-mode.png":::

# [Azure PowerShell](#tab/azure-powershell)

Set `-DataAccessAuthMode` to `AzureActiveDirectory` to require Microsoft Entra ID authentication when exporting or uploading to the disk. Use the following command to update an existing disk. Replace the values for `-ResourceGroupName` and `-DiskName` before running the command:

```azurepowershell
New-AzDiskUpdateConfig -DataAccessAuthMode "AzureActiveDirectory" | Update-AzDisk -ResourceGroupName 'yourResourceGroupName' -DiskName 'yourDiskName'
```

# [Azure CLI](#tab/azure-cli)

Set `--data-access-auth-mode` to `AzureActiveDirectory` to require Microsoft Entra ID authentication when exporting or uploading to the disk. Use the following command to update an existing disk. Replace the values for `--resource-group` and `--name` before running the command:

```azurecli
az disk update --name yourDiskName --resource-group yourResourceGroup --data-access-auth-mode AzureActiveDirectory
```

---

## Create an Azure Policy assignment

You can also create an Azure Policy assignment with a remediation task. A policy assignment with a remediation task continuously audits your resources and notifies you when any of them don't comply. Use the built-in policy definition **Protect your data with authentication requirements when exporting or uploading to a disk or snapshot**. To learn how to create an Azure Policy assignment, see the [Azure portal](/azure/governance/policy/assign-policy-portal), [Azure CLI](/azure/governance/policy/assign-policy-azurecli), or [Azure PowerShell](/azure/governance/policy/assign-policy-powershell) articles.

## Next steps

- [Upload a VHD to Azure or copy a managed disk to another region - Azure CLI](linux/disks-upload-vhd-to-managed-disk-cli.md)
- [Upload a VHD to Azure or copy a managed disk to another region - Azure PowerShell](windows/disks-upload-vhd-to-managed-disk-powershell.md)
- [Download a Linux VHD from Azure](linux/download-vhd.md)
- [Download a Windows VHD from Azure](windows/download-vhd.md)
