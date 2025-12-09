---
title: Soft Delete in Azure Compute Gallery (Preview)
description: Learn how to enable the Soft Delete feature in Azure Compute Gallery and recover images from accidental deletes.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/29/2025
ms.author: saraic
ms.reviewer: jushiman
ms.custom: template-how-to
ms.devlang: azurecli
# Customer intent: As a cloud service provider or cloud administrator, I want to enable Soft Delete on an Azure Compute Gallery, so that I can recover images that were accidentally deleted.
---

# Soft Delete in Azure Compute Gallery (Preview) 

> [!IMPORTANT]
> The Soft Delete feature is currently in PREVIEW. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The Soft Delete feature in Azure Compute Gallery (ACG) lets you recover accidentally deleted resources within a 7 day retention period. After this timeframe, the platform permanently deletes the resources. Currently, this feature is only available for ACG images.

Soft Delete is available for the following gallery types:
-	Private
-	Direct Shared Gallery
-	Community Gallery

When a resource, such as an image, is deleted using Soft Delete, it isn't immediately removed from the system. Instead, it enters a *soft-deleted* state, during which it remains recoverable for up to seven days. This grace period gives you time to review and restore any resources that were mistakenly deleted, preventing permanent loss.

After the retention window expires, the deleted items are automatically purged and can't be recovered. Soft Delete is useful in environments where accidental deletions can disrupt workflows or cause data loss.

## Prerequisites
- Register the [Azure Compute Gallery Soft Delete feature](/azure/azure-resource-manager/management/preview-features) in the Azure portal.
- Alternatively, register Azure Compute Gallery Soft Delete feature by running the following `az feature register` command from CLI:
   ```
   az feature register --name SIGSoftDelete --namespace Microsoft.Compute 
   ```
- The minimum API version required to use Soft Delete is `2024-03-03`.

## Limitations
-	Soft Delete is only supported in the Azure portal or Rest API with some limitations.
-	You can't currently update the retention policy.
-	Virtual Machine Apps Recovery isn't supported. 
-	Deleting a gallery or image definition with Soft Deleted resources isn't supported.
-	Only the following users are allowed to enable Soft Delete: the owner of a subscription, user or service principal assigned to the Compute Gallery Sharing Admin role at the subscription or gallery level
-	Not supported in National Clouds.
-	You can't enable Soft Delete if the gallery or image definition and image version are in different locations.
-	Virtual Machine Scale Sets in Flexible orchestration aren't compatible with the Soft Delete feature.
-	When an image in a gallery with Soft Delete enabled has a failed provisioning state, it can't be deleted. The recommended workaround is to temporarily disable Soft Delete on the gallery, delete the image, and then re-enable Soft Delete.

## Billing
Soft Delete is provided at no cost. After an image is soft deleted, only one replica per region is retained. You are charged for that single replica until the image is permanently deleted.

## Enable Soft Delete on a gallery
You can enable Soft Delete on a new or existing gallery.

### [REST](#tab/rest-1)
To enable Soft Delete on an existing gallery, send the following `PUT` request. As part of the request, include and set the `Softdeletepolicy` and `isSoftDeleteEnabled` values to `true`.

```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}?api-version=2024-03-03 
{
"properties": {
"softDeletePolicy": {
"isSoftDeleteEnabled": true
}
},
"location": "{location}
}
```

### [Azure portal](#tab/portal-1)
To enable Soft Delete on a new gallery, find the feature on the Basics tab during the gallery creation process in the Azure portal. The following steps instruct you on how to access this feature during that process.

1. Log in to the [Azure portal](https://portal.azure.com). 

1. In the search bar, search for and select **Azure compute galleries**. 

1. Select **Create** on the **Azure compute galleries** page. 

1. In the **Basics** tab, turn on the **Enables soft delete for image versions** option under the **Soft delete** section. 

1. When you're done, select **Review + Create**.

---

## Soft delete an image
Once Soft Delete is enabled on a gallery, all images in the gallery are soft deleted.

### [REST](#tab/rest-2)
To soft delete an image, send the following `DELETE` request on the resource you intend to delete.

```rest
DELETE 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
```

### [Azure portal](#tab/portal-2)
Soft delete an image in the Azure portal.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. At the bottom of the page, select the **Image Version** to be deleted and click on **Delete**.

---

## List all soft deleted images
View a list of soft deleted images in a gallery or image definition.

### [REST](#tab/rest-3)
Send the following `GET` request on the resource of the image definition to list all the soft deleted images.

```rest
GET 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-defination-name}/versions?api-version=2024-03-03
```

### [Azure portal](#tab/portal-3)
Go to the image definition in the Azure portal to list all the soft deleted images.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. Go to the **Image Definitions** blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions.

---

## Recover a soft deleted image
In this example, we demonstrate how to recover a soft deleted image.

### [REST](#tab/rest-4)
To recover soft deleted images in a gallery, send the following `PUT` request on the image version to be recovered along with the home region of the image.

```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
{
    "location": "eastus2euap"
}
```

### [Azure portal](#tab/portal-4)
Go to the image definition in the Azure portal to recover a soft deleted image.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. Go to the **Image Definitions** blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions.

1. Select the image version to recover and click on **Undelete**.

---

## Hard Delete an image
Hard Delete permanently deletes the image without the possibility of a recovery.

### [REST](#tab/rest-5)
To hard delete an image in a gallery, send the following `DELETE` request on the image version.

```rest
DELETE  
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
```

### [Azure portal](#tab/portal-5)
Go to the image definition in the Azure portal to hard delete an image.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. Go to the **Image Definitions** blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions.

1. Select the image version to hard delete and click on **Delete**. 

---

## Frequently Asked Questions

### Does Soft Delete become effective immediately upon being enabled in the gallery?
Yes, as long as the soft delete operation completes successfully. 

### Can I enable Soft Delete on an existing gallery and instantly delete and recover images, or is there a delay?
Yes, as long as the soft delete operation completes successfully.

### Can I update the retention period beyond seven days?
No, currently you can't update the retention period.

### Can I delete a gallery or image definition that has soft deleted images?
No, you can either delete all the soft deleted images or disable soft delete on the gallery to delete the image definition.

