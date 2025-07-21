---
title: What's new for Azure Compute Gallery
description: Learn about what's new for Azure Compute Gallery in Azure.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: whats-new
ms.date: 01/25/2023
ms.author: mattmcinnes
ms.reviewer: cynthn
# Customer intent: "As a cloud administrator, I want to review the latest updates to Azure Compute Gallery features, so that I can leverage the new capabilities and best practices to optimize image management and deployment in my organization's infrastructure."
---

# What's new for Azure Compute Gallery
This article is a list of updates to Compute Gallery features in Azure.

## July 2025 updates:
- [Trusted Launch Validation in Preview](./azure-compute-gallery.md#trusted-launch-validation-for-azure-compute-gallery-acg-images-preview) launched.
- [ExcludefromLatest](https://learn.microsoft.com/cli/azure/sig/image-version?view=azure-cli-latest#az-sig-image-version-create) property to exclude an image version from being used for creating resources for testing images and safely rolling out the image to Prod. This can be set at the region level as well to exclude specific regions.
- [Allow Deletion of Replicated Locations](https://learn.microsoft.com/rest/api/compute/gallery-image-versions/create-or-update?view=rest-compute-2025-02-01&tabs=HTTP#galleryimageversionsafetyprofile) property to indicate whether or not removing the gallery image version from replicated regions is allowed and prevents accidental deletion of regions.
- [BlockDeletionBeforeEndOfLife](https://learn.microsoft.com/rest/api/compute/gallery-image-versions/create-or-update?view=rest-compute-2025-02-01&tabs=HTTP#galleryimageversionsafetyprofile) property enables customers to prevent deletion of image before EOL date. Customers can opt-out by explicitly setting "BlockDeletionBeforeEndOfLife" to False
- [Platform metadata](https://learn.microsoft.com/rest/api/compute/gallery-image-versions/create-or-update?view=rest-compute-2024-11-04&tabs=HTTP#platformattribute) such as Publisher/Offer/SKU information from a VM created from Marketplace image is carried over to ACG Image


## January 2023 updates:
- [Launched public preview of Direct shared gallery on 07/25](./share-gallery-direct.md?tabs=portaldirect)
- [Best Practices document now available](./azure-compute-gallery.md#best-practices)
- [Replica count for 'Image Versions' increased from 50 to 100](./azure-compute-gallery.md#limits)

### Supported features:
- ['ARM64' image support](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- ['TrustedLaunch' support](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- ['ConfidentialVM' support](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- ['IsAcceleratedNetworkSupported' support](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- [Replication Mode: Full and Shallow](/cli/azure/sig/image-version?view=azure-cli-latest#commands&preserve-view=true)
  - Shallow replication support image size up to 32 TB
  - Shallow replication is only for test images

## Next steps
For updates and announcements about Azure, see the [Microsoft Azure Blog](https://azure.microsoft.com/blog/).
