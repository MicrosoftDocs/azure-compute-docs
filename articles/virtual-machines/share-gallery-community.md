---
title: Share Azure Compute Gallery Resources with a Community Gallery
description: Learn how to use a community gallery to share VM images stored in Azure Compute Gallery.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/20/2023
ms.author: saraic
ms.reviewer: cynthn, mattmcinnes
ms.custom: template-how-to
ms.devlang: azurecli
# Customer intent: As a cloud service provider, I want to share VM images through a community gallery, so that I can make my resources available to a broader audience of Azure users without compromising my private gallery.
---

# Share images by using a community gallery

To share a gallery with all Azure users, you can create a [community gallery](azure-compute-gallery.md#community-gallery). Anyone who has an Azure subscription can use community galleries. Someone who creates a virtual machine (VM) can browse through images shared with the community by using the Azure portal, the REST API, or the Azure CLI. Sharing images with the community is a new capability in Azure Compute Gallery.

You can make your image galleries public and share them with Azure customers. When a gallery is marked as a community gallery, all images in it become available to all Azure customers as a new resource type under `Microsoft.Compute/communityGalleries`. Azure customers can see the galleries and use them to create VMs. Your original resources of the type `Microsoft.Compute/galleries` are still under your subscription and private.

> [!IMPORTANT]
> Microsoft does not provide support for community images. For any image-related issues, contact the image publishers. However, Microsoft does provide commercially reasonable support to isolate any platform issue that involves a community image. You can find the publisher contact by querying the image.

There are three main ways to share images in Compute Gallery, depending on which users you want to share with:

| Sharing with: | People | Groups | Service principal | All users in a specific subscription or tenant | Publicly with all users in Azure |
|---|---|---|---|---|---|
| [Role-based access control (RBAC) sharing](share-gallery.md) | Yes | Yes | Yes | No | No |
| RBAC + [direct shared gallery](./share-gallery-direct.md)  | Yes | Yes | Yes | Yes | No |
| RBAC + [community gallery](./share-gallery-community.md) | Yes | Yes | Yes | No | Yes |

> [!NOTE]
> You can use images with read permissions on them to deploy virtual machines and disks.
>
> When you use a direct shared gallery, images are distributed widely to all users in a subscription or tenant. A community gallery distributes images publicly. When you share images that contain intellectual property, use caution to prevent widespread distribution.

## Disclaimer

[!INCLUDE [community-gallery-artifacts](./includes/community-gallery-artifacts.md)]

## Limitations for sharing images with the community

- You can't convert an existing private gallery (RBAC-enabled gallery) to a community gallery.
- You can't use a partner image from Azure Marketplace and publish it to the community.
- Encrypted images are not supported.
- The feature isn't available in government clouds.
- Image resources need to be created in the same region as the gallery. For example, if you create a gallery in West US, the image definitions and image versions should be created in West US if you want to make them available.
- You currently can't share [VM applications](vm-applications.md) with the community.

## How sharing with the community works

You [create a gallery resource](create-gallery.md#create-a-community-gallery) under `Microsoft.Compute/Galleries` and choose `community` as a sharing option.

When you're ready, you flag your gallery as ready to be shared publicly. Only the owner of a subscription, or a user or service principal with the Compute Gallery Sharing Admin role at the subscription or gallery level, can enable a gallery to go public to the community. At this point, the Azure infrastructure creates proxy read-only regional resources under `Microsoft.Compute/CommunityGalleries`. These proxy resources are public.

Users can interact only with the proxy resources. They never interact with your private resources. As the publisher of a private resource, you should consider the private resource as your handle to the public proxy resources. The `prefix` value that you provide when you create the gallery is used, along with a GUID, to create the public-facing name for your gallery.

Azure users can see the latest image versions shared with the community in the portal, or query for them by using the Azure CLI. Only the latest version of an image is listed in the community gallery.

When you create a community gallery, you need to provide contact information for your images. This information helps facilitate communication between the consumer of the image and the publisher if the consumer needs assistance. Microsoft doesn't offer support for these images. This information is *publicly available*, so be careful when you provide it:

- Community gallery prefix.
- Publisher support email.
- Publisher URL.
- Legal agreement URL. Do not put secrets, passwords, share access signature (SAS) URIs, or any other sensitive information in this URL.

Information from your image definitions is also publicly available, including what you provide for **Publisher**, **Offer**, and **SKU**.

> [!WARNING]
> If you want to stop sharing a gallery publicly, you can update the gallery to stop sharing. However, making the gallery private will prevent existing users of virtual machine scale sets from scaling their resources.

## How an image publisher chooses between publishing to Azure Marketplace and a community gallery

When to publish to Azure Marketplace:

- You signed Azure Marketplace terms.
- You want to publish commercial images.
- You want to publish a stable version or major release.

When to publish to a community gallery:

- You can't sign Azure Marketplace terms and still want to share images publicly on Azure.
- You want to publish non-commercial images.
- You want to publish daily or nightly builds.

## How an image consumer chooses between Azure Marketplace and community images

A consumer might choose Azure Marketplace images when that consumer needs:

- Microsoft-certified images.
- Images for production workloads.
- Microsoft and partner images.
- Paid images with additional software offerings.
- Images that Microsoft supports.

A consumer might choose community images when that consumer needs:

- A community version of an image published by an open-source community.
- Images from a trusted publisher with known contact information.
- Images for testing.
- Free images.
- Images supported by the image owners, not Microsoft.

## Reporting issues with a community image

Using community-submitted VM images has several risks. Images could contain malware, security vulnerabilities, or intellectual property. To help create a secure and reliable experience for the community, you can report images when you see these issues.

The easiest way to report issues with a community gallery is to use the portal, which pre-fills information for the report:

- For issues with links or other information in the fields of an image definition, select **Report community image**.
- For malicious code or other issues in a specific image version, select **Report** under the **Report version** column in the table of image versions.

You can also use the following links to report issues, but the forms aren't pre-filled:

- For malicious images, [report abuse](https://msrc.microsoft.com/report/abuse).
- For violations of intellectual property, [submit an infringement report](https://msrc.microsoft.com/report/infringement).

## Best practices

- Images published to the community gallery should be [generalized](generalize.yml) and have no sensitive or machine-specific details. To learn more about preparing an image, see the OS-specific information for [Linux](./linux/create-upload-generic.md) or [Windows](./windows/prepare-for-upload-vhd-image.md).

- If you want to block sharing images with the community at the organization level, create an Azure policy with the following policy rule:

  ```
    "policyRule": {
        "if": {
          "allOf": [
            {
              "field": "type",
              "equals": "Microsoft.Compute/galleries"
            },
            {
              "field": "Microsoft.Compute/galleries/sharingProfile.permissions",
              "equals": "Community"
            }
          ]
        },
        "then": {
          "effect": "[parameters('effect')]"
        }
      }
  ```

## FAQ

### What are the charges for using a gallery that's shared with the community?

There are no charges for using the service itself. However, content publishers do receive:

- Storage charges for application versions and replicas in each of the regions (source and target). These charges are based on the chosen type of storage account.
- Network egress charges for replication across regions.

Consumers of the image might have to pay additional software costs if the base image is using an Azure Marketplace image with software charges.

### Is it safe to use images shared with the community?

Users should exercise caution while using images from unverified sources. These images aren't subject to certification, and they're not scanned for malware or vulnerabilities.

### If an image that's shared with the community doesn't work, who provides support?

Azure isn't responsible for any issues that users might encounter with community-shared images. The image publisher provides the support. If you need support, use the image publisher's contact information.  

### Is community gallery sharing part of Azure Marketplace?

No, community gallery sharing isn't part of Azure Marketplace. It's a feature of Azure Compute Gallery. Anyone who has an Azure subscription can use a community gallery to make their images public.

### How do I request that an image shared with the community is replicated to a specific region?

Only the content publishers have control over the regions where their images are available. If you don't find an image in a specific region, contact the publisher directly.

## Start sharing publicly

For a gallery to be shared publicly, it needs to be created as a community gallery. For more information, see [Create a community gallery](create-gallery.md#create-a-community-gallery).

### [CLI](#tab/cli)

When you're ready to make the gallery available to the community, use the [`az sig share enable-community`](/cli/azure/sig/share#az-sig-share-enable-community) command. Only a user in the Owner role definition can enable a gallery for community sharing.

```azurecli-interactive
az sig share enable-community \
   --gallery-name $galleryName \
   --resource-group $resourceGroup 
```

To go back to only RBAC-based sharing, use the [`az sig share reset`](/cli/azure/sig/share#az-sig-share-reset) command.

To delete a gallery that's shared with the community, you must first run `az sig share reset` to stop sharing. Then you can delete the gallery.

### [REST](#tab/rest)

To go live with community sharing, send the following `POST` request. As part of the request, include the property `operationType` with the value `EnableCommunity`.  

```rest
POST 
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compu 
te/galleries/{galleryName}/share?api-version=2021-07-01 
{ 
  "operationType" : "EnableCommunity"
} 
```

### [Portal](#tab/portal)

When you're ready to make the gallery public:

1. On the page for the gallery, select **Sharing** from the left menu.

1. On the top of the pane, select **Share**.

   :::image type="content" source="media/create-gallery/share.png" alt-text="Screenshot that shows the Share button for sharing a gallery with the community.":::

1. When you finish, select **Save**.

---

> [!IMPORTANT]
> If you're listed as the owner of your subscription, but you're having trouble sharing the gallery publicly, you might need to explicitly [add yourself as owner again](/azure/role-based-access-control/role-assignments-portal-subscription-admin).

To go back to only RBAC-based sharing, use the [`az sig share reset`](/cli/azure/sig/share#az-sig-share-reset) command.

To delete a gallery that's shared with the community, you must first run `az sig share reset` to stop sharing. Then you can delete the gallery.

## Related content

- Create an [image definition and an image version](image-version.md).
- Create a VM from a [generalized](vm-generalized-image-version.md#community-gallery) or [specialized](vm-specialized-image-version.md#community-gallery) image in a community gallery.
