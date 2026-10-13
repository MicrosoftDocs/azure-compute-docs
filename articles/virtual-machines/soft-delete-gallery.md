---
title: Soft Delete in Azure Compute Gallery (Preview)
description: Learn how to enable the Soft Delete feature in Azure Compute Gallery and recover images from accidental deletes.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/02/2026
ms.author: saraic
ms.reviewer: jushiman
ms.custom: template-how-to
ms.devlang: azurecli
# Customer intent: As a cloud service provider or cloud administrator, I want to enable Soft Delete on an Azure Compute Gallery, so that I can recover images that were accidentally deleted.
---

# Soft Delete in Azure Compute Gallery (Preview) 

> [!IMPORTANT]
> The Soft Delete feature is currently in PREVIEW. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The Soft delete feature in Azure Compute Gallery lets you recover accidentally deleted image versions before they're permanently deleted. Soft delete is available starting with API version `2024-03-03`. API version `2026-03-03` adds customizable retention and grace periods and changes when a soft-deleted image can be consumed. This feature is currently available only for Azure Compute Gallery images.

Soft Delete is available for the following gallery types:
-	Private
-	Direct Shared Gallery
-	Community Gallery

When an image version is deleted by using Soft delete, it isn't immediately removed from the system. Instead, it enters a *soft-deleted* state and moves through the following lifecycle:

- **Retention period:** With API version `2026-03-03`, the image remains recoverable and available for consumption. You can use the image to create VMs and virtual machine scale sets, and you can scale out existing virtual machine scale sets. The retention period can be up to 365 days and defaults to seven days.
- **Grace period:** The grace period begins when the retention period expires. The image remains recoverable, but it can no longer be consumed. Attempts to use it to create VMs or virtual machine scale sets or to scale out existing virtual machine scale sets fail. The grace period can be up to 90 days and defaults to seven days.
- **Permanent deletion:** The platform permanently deletes the image when the grace period ends.

With API versions earlier than `2026-03-03`, a soft-deleted image can't be used to create a VM or virtual machine scale set or to scale out an existing scale set during the retention period. API version `2026-03-03` allows these operations during the retention period and blocks them only after the grace period begins.

Soft delete is useful in environments where accidental deletions can disrupt workflows or cause data loss. If you don't need the recovery window, you can bypass Soft delete and permanently delete an image version.

## Prerequisites
- Register the [Azure Compute Gallery Soft Delete feature](/azure/azure-resource-manager/management/preview-features) in the Azure portal.
- Alternatively, register Azure Compute Gallery Soft Delete feature by running the following `az feature register` command from CLI:
   ```
   az feature register --name SIGSoftDelete --namespace Microsoft.Compute 
   ```
- The minimum API version required to use Soft Delete is `2024-03-03`.
- Use API version `2026-03-03` to customize the retention and grace periods, consume images during the retention period, and use the new image lifecycle properties.

## Limitations
-	Soft Delete is only supported in the Azure portal or Rest API with some limitations.
-	Virtual Machine Apps Recovery isn't supported. 
-	Deleting a gallery or image definition with Soft Deleted resources isn't supported.
-	Only the following users are allowed to enable Soft Delete: the owner of a subscription, user or service principal assigned to the Compute Gallery Sharing Admin role at the subscription or gallery level
-	You can't enable Soft Delete if the gallery or image definition and image version are in different locations.
-	Image recovery isn't possible if the gallery or image definition and image version are in different regions.
-	Virtual Machine Scale Sets in Flexible orchestration aren't compatible with the Soft Delete feature.
-	When an image in a gallery with Soft Delete enabled has a failed provisioning state, you can bypass Soft Delete and hard delete the image.

## Billing
Soft Delete is free. The replica count doesn't reduce when an image enters the soft-deleted state. You're charged for all retained replicas until the image is permanently deleted.

## Enable Soft Delete on a gallery
You can enable Soft Delete on a new or existing gallery.

### [REST](#tab/rest-1)
To enable Soft Delete on an existing gallery, send the following `PUT` request. As part of the request, include the `softDeletePolicy` property, set `isSoftDeleteEnabled` to `true`, and specify `retentionPeriodInDays` and `gracePeriodInDays`. Set the retention period from 1 through 365 days and the grace period from 1 through 90 days. Both periods default to seven days. The following example uses a seven-day retention period and a 30-day grace period.

```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}?api-version=2026-03-03 
{
    "properties": {
        "softDeletePolicy": {
            "isSoftDeleteEnabled": true,
            "retentionPeriodInDays": 7,
            "gracePeriodInDays": 30
        }
    },
    "location": "{location}"
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
To soft delete an image, send the following `DELETE` request on the resource you want to delete. Omit the optional `bypassSoftDelete` parameter or set it to `false`.

```rest
DELETE 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-definition-name}/versions/{version-name}?api-version=2026-03-03
```

### [Azure portal](#tab/portal-2)
Soft delete an image in the Azure portal.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. At the bottom of the page, select the **Image Version** to be deleted and click on **Delete**.

---

## List all soft deleted images
View a list of soft-deleted images in a gallery or image definition. Starting with API version `2026-03-03`, the response includes the following lifecycle properties:

| Property | Description |
|---|---|
| `softDeleteTime` | The time when the image entered the soft-deleted state. |
| `consumptionEndTime` | The time when the retention period expires and the grace period begins. After this time, attempts to use the image to create VMs or scale out virtual machine scale sets fail. This value depends on the configured retention period. |
| `hardDeletionTargetTime` | The target time when the grace period expires and the platform permanently deletes the image. The image can be recovered until this time. This value depends on the configured grace period. |
| `imageState` | The current image state. `Active` means that the image is active and isn't soft deleted. `SoftDeleted` means that the image has entered the soft-deleted state. A soft-deleted image remains consumable until `consumptionEndTime` and recoverable until `hardDeletionTargetTime`. During the grace period between these times, the image remains in the `SoftDeleted` state and is recoverable but isn't consumable. |

### [REST](#tab/rest-3)
Send the following `GET` request on the resource of the image definition to list all the soft deleted images.

```rest
GET 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-definition-name}/versions?api-version=2026-03-03
```

### [Azure portal](#tab/portal-3)
Go to the image definition in the Azure portal to list all the soft deleted images.

1. In the [Azure portal](https://portal.azure.com), go to **Azure compute galleries**. 

1. Select your Soft Delete enabled gallery on the **Azure compute galleries** page.

1. Go to the **Image Definitions** blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions.

---

## Recover a soft deleted image
You can recover a soft deleted image during the retention and grace periods, until `hardDeletionTargetTime`. In this example, we demonstrate how to recover a soft deleted image.

### [REST](#tab/rest-4)
To recover soft deleted images in a gallery, send the following `PUT` request on the image version to be recovered along with the home region of the image.

```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-definition-name}/versions/{version-name}?api-version=2026-03-03
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
Hard delete permanently deletes an image without the possibility of recovery. You can hard delete an image that's already in the soft deleted state without waiting for its retention and grace periods to end. Alternatively, you can bypass Soft Delete and permanently delete an active image without placing it in the soft deleted state.

### [REST](#tab/rest-5)
#### Hard delete an image in the soft-deleted state

To permanently delete an image that's already in the soft-deleted state, send the following request. This operation deletes the soft-deleted artifact immediately, without waiting for the retention and grace periods to end.

```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-definition-name}/versions/{version-name}?api-version=2024-03-03
```

#### Bypass soft delete for an active image

The optional `bypassSoftDelete` query parameter is a Boolean value. Set it to `true` to permanently delete an active gallery image version without retaining it in the recycle bin. This operation is irreversible.

| `bypassSoftDelete` | Gallery soft-delete policy | Result |
|---|---|---|
| `true` | Enabled or disabled | Permanently deletes the gallery image version. The version isn't retained in the recycle bin and can't be restored. |
| `false` or omitted | Enabled | Soft deletes the version. The version is retained in the recycle bin and can be restored during the configured recovery period. |
| `false` or omitted | Disabled | Permanently deletes the version because no soft-delete policy applies. |

To bypass Soft Delete and permanently delete an active image version, send the following request:

```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/images/{galleryImageName}/versions/{galleryImageVersionName}?api-version=2026-03-03&bypassSoftDelete=true
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
Yes. By using API version `2026-03-03`, you can configure the retention period for up to 365 days. The default retention period is seven days.

### Can I update the grace period?
Yes. With API version `2026-03-03`, you can configure the grace period for up to 90 days. The default grace period is seven days.

### Can I delete a gallery or image definition that has soft deleted images?
No, you can either delete all the soft deleted images or disable soft delete on the gallery to delete the image definition.

