---
title: Share Resources in Azure Compute Gallery
description: Learn how to share resources explicitly or to all Azure users by using role-based access control.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 02/14/2023
ms.author: saraic
ms.reviewer: cynthn
ms.custom: template-how-to , devx-track-azurecli
ms.devlang: azurecli
# Customer intent: As an IT administrator, I want to share resources in Azure Compute Gallery by using role-based access control, so that I can manage user access effectively across subscriptions and tenants.
---

# Share gallery resources across subscriptions and tenants by using RBAC

Because a gallery, definition, and version are all resources in Azure Compute Gallery, you can share them by using the built-in Azure role-based access control (RBAC) roles. By using Azure RBAC roles, you can share these resources with other users, service principals, and groups. You can even share access with individuals outside the tenant where they were created.

After users have access, they can use the gallery resources to deploy a virtual machine (VM) or a virtual machine scale set. Here's a sharing matrix that can help you understand what the users get access to:

| Shared with users | Gallery | Image definition | Image version |
|----------------------|----------------------|--------------|----------------------|
| Gallery | Yes                  | Yes          | Yes                  |
| Image definition     | No                   | Yes          | Yes                  |

We recommend sharing at the gallery level for the best experience. We don't recommend sharing individual image versions. For more information about Azure RBAC, see [Assign Azure roles](/azure/role-based-access-control/role-assignments-portal).

There are three main ways to share images in Compute Gallery, depending on which users you want to share with:

| Sharing with: | People | Groups | Service principal | All users in a specific subscription or tenant | Publicly with all users in Azure |
|---|---|---|---|---|---|
| RBAC sharing | Yes | Yes | Yes | No | No |
| RBAC + [direct shared gallery](./share-gallery-direct.md)  | Yes | Yes | Yes | Yes | No |
| RBAC + [community gallery](./share-gallery-community.md) | Yes | Yes | Yes | No | Yes |

You can also create an [app registration](./share-using-app-registration.md) to share images between tenants.

> [!NOTE]
> You can use images with read permissions on them to deploy virtual machines and disks.
>
> When you use a direct shared gallery, images are distributed widely to all users in a subscription or tenant. A community gallery distributes images publicly. When you share images that contain intellectual property, use caution to prevent widespread distribution.

## Share by using RBAC

When you share a gallery by using RBAC, you need to provide the `imageID` value to anyone who creates a VM or scale set from the image. The person who's deploying the VM or scale set can't list the images that were shared with them via RBAC.

If you share gallery resources with someone outside your Azure tenant, they need your `tenantID` value to sign in and have Azure verify that they have access to the resource before they can use it within their own tenant. You need to provide the `tenantID` value. There is no way for someone outside your organization to query for this value.

For instructions on consuming an image shared via RBAC and creating a VM or a scale set, see:

- [RBAC - Shared within your organization](vm-generalized-image-version.md#rbac---shared-within-your-organization)
- [RBAC - Shared from another tenant](vm-generalized-image-version.md#rbac---shared-from-another-tenant)

### [Portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com).

1. On the page for your gallery, on the left menu, select **Access control (IAM)**.

1. Under **Add**, select **Add role assignment**. The **Add role assignment** pane opens.

1. Under **Role**, select **Reader**.

1. On the **Members** tab, ensure that the user is selected. For **Assign access to**, keep the default of **User, group, or service principal**.

1. Choose **Select members**. On the pane that opens, choose a user account.

1. If the user is outside your organization, the following message appears: **This user will be sent an email that enables them to collaborate with Microsoft**. Select the user with the email address, and then select **Save**.

### [CLI](#tab/cli)

To get the object ID of your gallery, use [az sig show](/cli/azure/sig#az-sig-show):

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Use the object ID as a scope, along with an email address and [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create), to give a user access to the gallery. Replace `<email address>` and `<gallery ID>` with your own information.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

For more information about how to share resources by using RBAC, see [Assign Azure roles by using the Azure CLI](/azure/role-based-access-control/role-assignments-cli).

### [PowerShell](#tab/powershell)

Use an email address and the [`Get-AzADUser`](/powershell/module/az.resources/get-azaduser) cmdlet to get the object ID for the user. Then use [`New-AzRoleAssignment`](/powershell/module/Az.Resources/New-AzRoleAssignment) to give the user access to the gallery. In the following example, replace the example email address (`alinne_montes@contoso.com`) with your own information.

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName

```

---

## Related content

- Create an [image definition and an image version](image-version.md).
- Create a VM from a [generalized](vm-generalized-image-version.md) or [specialized](vm-specialized-image-version.md) image in a gallery.
