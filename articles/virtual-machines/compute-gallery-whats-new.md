---
title: What's New for Azure Compute Gallery
description: Learn about what's new for Azure Compute Gallery in Azure.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: whats-new
ms.date: 01/25/2023
ms.author: mattmcinnes
ms.reviewer: cynthn
# Customer intent: "As a cloud administrator, I want to review the latest updates to Azure Compute Gallery features, so that I can use the new capabilities and best practices to optimize image management and deployment in my organization's infrastructure."
---

# What's new for Azure Compute Gallery

This article lists updates to Azure Compute Gallery features.

## July 2025 updates

- Starting with API version 2025-03-03, new Compute Gallery image definitions default to `hyperVGeneration` set to `V2` and `SecurityType` set to `TrustedLaunchSupported`.
- Trusted Launch validation for Azure Compute Gallery images is [in preview](./azure-compute-gallery.md#trusted-launch-validation-for-azure-compute-gallery-images-preview).
- The [`ExcludefromLatest`](/cli/azure/sig/image-version#az-sig-image-version-create) property excludes the use of an image version for creating resources to test images and safely rolling out the image to production. You can also set this property at the region level to exclude specific regions.
- The [`allowDeletionOfReplicatedLocations`](/rest/api/compute/gallery-image-versions/create-or-update?&tabs=HTTP#galleryimageversionsafetyprofile) property indicates whether or not removing the gallery image version from replicated regions is allowed and prevents accidental deletion of regions.
- The [`BlockDeletionBeforeEndOfLife`](/rest/api/compute/gallery-image-versions/create-or-update?&tabs=HTTP#galleryimageversionsafetyprofile) property enables customers to prevent deletion of images before the end-of-support date. You can opt out by explicitly setting `BlockDeletionBeforeEndOfLife` to `False`.
- [Platform metadata](/rest/api/compute/gallery-image-versions/create-or-update?&tabs=HTTP#platformattribute), such as publisher, offer, or product information from a virtual machine (VM) created from an Azure Marketplace image, is carried over to the Compute Gallery image.

## January 2023 updates

- [Direct shared gallery](./share-gallery-direct.md?tabs=portaldirect) is in preview.
- [Best practices](./azure-compute-gallery.md#best-practices) information is available.
- [Replica count for image versions](./azure-compute-gallery.md#limits) increased from 50 to 100.

### Supported features

- [ARM64 images](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- [Trusted Launch](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- [Confidential VMs](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- [Accelerated networking](/cli/azure/sig/image-definition?view=azure-cli-latest#az-sig-image-definition-create&preserve-view=true)
- [Replication mode: full and shallow](/cli/azure/sig/image-version?view=azure-cli-latest#commands&preserve-view=true)
  - Image size for shallow replication support is up to 32 TB.
  - Shallow replication is only for test images.

## Related content

- For updates and announcements about Azure, see the [Microsoft Azure Blog](https://azure.microsoft.com/blog/).
