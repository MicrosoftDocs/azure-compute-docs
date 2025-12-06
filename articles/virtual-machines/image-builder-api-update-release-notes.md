---
title: What's New in Azure VM Image Builder
description: This article offers the latest release notes, known issues, bug fixes, deprecated functionality, and upcoming changes for Azure VM Image Builder.
author: kof-f
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 02/13/2024
ms.reviewer: mattmcinnes
ms.subservice: image-builder
ms.custom: references_regions
# Customer intent: As a software developer utilizing VM Image Builder, I want to stay informed about the latest API changes and feature updates, so that I can effectively manage and optimize my image creation processes without encountering compatibility issues.
---

# What's new in Azure VM Image Builder

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article contains all major API changes and feature updates for the Azure VM Image Builder service.

## Updates

### September 2024

Automatic image creation via triggers is deactivated if the image template build fails multiple times consecutively. This deactivation avoids unnecessary build failures.

You can still manually build the image template. After a manual build succeeds, the automatic triggers are reactivated.

This behavior is the same regardless of which API version you use for the image template resource.

### May 2024

#### Breaking change: Case sensitivity

As of May 21, 2024, VM Image Builder API version 2024-02-01 and later enforces case sensitivity for all fields. The capitalization of letters in your API requests must match exactly with the expected format.

> [!IMPORTANT]
> If you're an existing user of VM Image Builder, this change does *not* affect your existing resources. The enforcement of case sensitivity applies only to newly created resources that use API version 2024-02-01 and later. Your existing resources continue to function as expected without any changes.
>
> If you encounter any problems related to case sensitivity, refer to the updated VM Image Builder API documentation for guidance.

Previously, the VM Image Builder API was more forgiving in terms of case. Moving forward, precision is crucial. When you make API calls, ensure that you use the correct capitalization for field names, parameters, and values. For example, if a field is named `vmBoot`, you must use `vmBoot` (not `VMBoot` or `vmboot`).

If you send an API request to VM Image Builder API version 2024-02-01 or later with incorrect case or unrecognized fields, the service rejects it. You receive an error response indicating that the request is invalid. The error looks something like this example:

`Unmarshalling entity encountered error: unmarshalling type *v2024_02_01.ImageTemplate: struct field Properties: unmarshalling type *v2024_02_01.ImageTemplateProperties: struct field Optimize: unmarshalling type *v2024_02_01.ImageTemplatePropertiesOptimize: unmarshalling type *v2024_02_01.ImageTemplatePropertiesOptimize, unknown field \"vmboot\". There is an issue with the syntax with the JSON template you are submitting. Please check the JSON template for syntax and grammar. For more information on the syntax and grammar of the JSON template, visit http://aka.ms/azvmimagebuildertmplref.`

The error message mentions an "unknown field" and directs you to the official documentation: [Create an Azure VM Image Builder Bicep or Azure Resource Manager JSON template](./linux/image-builder-json.md).

> [!NOTE]
> When you're making API calls to the VM Image Builder service, always reference the [Swagger documentation](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/imagebuilder/resource-manager/Microsoft.VirtualMachineImages). This documentation serves as the definitive source of truth for VM Image Builder API specifications. Although the public documentation was updated to include the proper capitalization and field names ahead of the API release, the Swagger definition contains precise details about each VM Image Builder API. These details help ensure that you're making calls to the service correctly.

The following documentation changes were made to match the field names in API version 2024-02-01.

In the [Create an Azure VM Image Builder Bicep or Azure Resource Manager JSON template](./linux/image-builder-json.md) documentation:

- Fields updated:
  - Replaced several mentions of `vmboot` with `vmBoot`.
  - Replaced one mention of `imageVersionID` with `imageVersionId`.

- Field removed:
  - `apiVersion`: We recommend avoiding the inclusion of this field in your requests because it's not explicitly specified in the API. Including it in your JSON template might lead to errors in your image build.

In the [Azure VM Image Builder networking options](./linux/image-builder-networking.md) documentation:

- Field updated:
  - Replaced one mention of `VirtualNetworkConfig` with `vnetConfig`.

- Fields removed:
  - `subnetName` in the `vnetConfig` property: This field is deprecated. The new field is `subnetId`.
  - `resourceGroupName` in the `vnetConfig` property: This field is deprecated. The new field is `subnetId`.

#### Pinning to an older VM Image Builder API version

If you want to avoid making changes to the properties in your image templates due to the new case-sensitivity rules, you have the option to pin your Azure VM Image Builder API calls to a previous API version. This pinning allows you to continue using the familiar behavior without any modifications.

> [!IMPORTANT]
> Pinning to an older VM Image Builder API version can provide compatibility with your existing templates, but we don't recommend it due to the following factors:
>
> - Older API versions might eventually be deprecated.
> - By pinning to an older API version, you miss the latest features and improvements introduced in newer versions. These enhancements often improve performance, security, and functionality.

To ensure compatibility with your existing templates when you're creating or updating an image template, specify the desired API version by including the `api-version` parameter in your call to the service. For example:

# [HTTP](#tab/http)

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.VirtualMachineImages/imageTemplates/{imageTemplateName}?api-version=2022-07-01
```

# [Azure CLI](#tab/-interactive)

```azurecli-interactive
az resource create \
    --api-version=2022-07-01 \
    --resource-group <resourceGroupName> \
    --properties <jsonResource> \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n <imageTemplateName>
```

# [Azure PowerShell](#tab/azurepowershell-interactive)

```azurepowershell-interactive
New-AzResourceGroupDeployment -ResourceGroupName <resourceGroupName> -TemplateFile <templateFilePath> -TemplateParameterObject @{"api-version" = "2022-07-01"; "imageTemplateName" = <imageTemplateName>; "svclocation" = <location>}
```

---

#### Testing your code

After you pin to the older API version, test your code to verify that it behaves as expected. Ensure that your existing templates continue to function correctly.

### November 2023

VM Image Builder is enabling Isolated Image Builds via Azure Container Instances in a phased manner. The rollout is expected finish by early 2024. Your existing image templates continue to work, and there's no change in the way you create or build new image templates.

You might observe a different set of transient Azure resources appearing temporarily in the staging resource group. It doesn't affect your actual builds or the way you interact with VM Image Builder. For more information, see [Isolated Image Builds](./security-isolated-image-builds-image-builder.md).

To use Isolated Image Builds, make sure that:

- Your subscription is registered for the `Microsoft.ContainerInstance` provider.
- There are no policies blocking deployment of Azure Container Instances resources.
- Quota is available for Azure Container Instances resources.

### April 2023

New portal functionality was added for VM Image Builder. Search for **Image Templates** in the Azure portal, and then select **Create**. You can also use [this template configuration](https://ms.portal.azure.com/#create/Microsoft.ImageTemplate) to get started with building and validating custom images inside the portal.

## API releases

### Version 2024-02-01

#### Improvements

- You can use the new `autoRun` property to run the image build on template creation or update. For more information, see [Properties: `autoRun`](../virtual-machines/linux/image-builder-json.md#properties-autorun).

- You can use the new `managedResourceTags` property to apply tags to the resources that the VM Image Builder service creates in the staging resource group during the image build. For more information, see [Properties: `managedResourceTags`](../virtual-machines/linux/image-builder-json.md#properties-managedresourcetags).

- You can use the new `containerInstanceSubnetId` property to specify a subnet on which Azure Container Instances will be deployed for Isolated Image Builds. You can specify this field only if you specify `subnetId`. This field must be on the same virtual network as the subnet specified in `subnetId`. For more information, see [Bring your own build VM subnet and bring your own Container Instances subnet](./security-isolated-image-builds-image-builder.md#bring-your-own-build-vm-subnet-and-bring-your-own-aci-subnet).

- This version adds support for updating the `vmProfile` property, including the following fields:
  - `vmSize`
  - `osDiskSizeGB`
  - `userAssignedIdentities`
  - `vnetConfig`
    - `subnetId`
    - `containerInstanceSubnetId`

  For more information on the `vmProfile` property, see [vmProfile](../virtual-machines/linux/image-builder-json.md#properties-vmprofile).

#### Changes

API version 2024-02-01 introduces a breaking change that enforces case sensitivity for all fields. The capitalization of letters in your API requests must match exactly with the expected format. If you send an API request to VM Image Builder API version 2024-02-01 or later with an incorrect case or unrecognized fields, the service rejects it. You receive an error response indicating that the request is invalid. For more information, see [Breaking change: Case sensitivity](#may-2024) in this article.

### Version 2023-07-01

#### Changes

The new `errorHandling` property gives you more control over how errors are handled during the image building process. For more information, see [errorHandling](../virtual-machines/linux/image-builder-json.md#properties-errorhandling).

### Version 2022-07-01

#### Improvements

- This version adds support to use the latest image version stored in Azure Compute Gallery as the source for the image template.
- This version adds `versioning` to support generating version numbers for image distributions. For more information, see [Properties: `versioning`](../virtual-machines/linux/image-builder-json.md#versioning).
- This version adds support for per-region configuration when you're distributing to Azure Compute Gallery. For more information, see [Distribute: targetRegions](../virtual-machines/linux/image-builder-json.md#distribute-targetregions).
- This version adds a new `File` validation type. For more information, see [Properties: `validate`](../virtual-machines/linux/image-builder-json.md#properties-validate).
- You can now distribute virtual hard disks (VHDs) to a custom blob or container in a custom storage account. For more information, see [Distribute: VHD](../virtual-machines/linux/image-builder-json.md#distribute-vhd).
- This version adds support for using a [direct shared gallery](/azure/virtual-machines/shared-image-galleries?tabs=azure-cli#sharing) image as the source for the image template.

#### Changes

- `replicationRegions` is now deprecated for gallery distributions. Instead, use [`gallery-replication-regions`](/cli/azure/image/builder/output?view=azure-cli-latest#az-image-builder-output-add-examples&preserve-view=true).
- You can now distribute VHDs to a custom blob or container in a custom storage account.
- This version adds the `targetRegions` array, which applies only to the `SharedImage` type of distribution. For more information on `targetRegions`, see [Store and share resources in Azure Compute Gallery](../../articles/virtual-machines/azure-compute-gallery.md).
- This version adds support for using a [direct shared gallery](/azure/virtual-machines/shared-image-galleries?tabs=azure-cli#sharing) image as the source for the image template. Direct shared galleries are currently in preview.
- Triggers are now available in preview to set up automatic image builds. For more information, see [How to enable automatic image creation with Azure VM Image Builder triggers](./image-builder-triggers-how-to.md).

### Version 2022-02-14

#### Improvements

- [Validation support](./linux/image-builder-json.md#properties-validate)
  - Shell (Linux): Script or inline
  - PowerShell (Windows): Script or inline, run elevated, run as system
  - Source-validation-only mode
- [Customized support for staging resource groups](./linux/image-builder-json.md#properties-stagingresourcegroup)

### Version 2021-10-01

#### Breaking change

API version 2021-10-01 introduces a change to the error schema that will be part of every future API release. If you have any Azure VM Image Builder automations, be aware of the [new error output](#error-output-for-version-2021-10-01-and-later) when you switch to API version 2021-10-01 or later.

We recommend, after you switch to the latest API version, that you don't revert to an earlier version. If you revert, you'll have to change your automation again to produce the earlier error schema. We don't expect the error schema to change again in future releases.

##### Error output for version 2020-02-14 and earlier

```
{ 
  "code": "ValidationFailed",
  "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template." 
} 
```

##### Error output for version 2021-10-01 and later

```
{ 
  "error": {
    "code": "ValidationFailed", 
    "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template." 
  }
}
```

#### Improvements

- Added support for [managed identities for the build VM](linux/image-builder-json.md#user-assigned-identity-for-the-image-builder-build-vm).
- Added support for customization of proxy VM size.

### Version 2020-02-14

#### Improvements

- Added support for creating images from the following sources:
  - Managed image
  - Azure Compute Gallery
  - Platform Image Repository (including Platform Image Purchase Plan)
- Added support for the following customizations:
  - Shell (Linux): Script or inline
  - PowerShell (Windows): Script or inline, run elevated, run as system
  - File (Linux and Windows)
  - Windows restart (Windows)
  - Windows Update (Windows): Search criteria, filters, and update limit
- Added support for the following distribution types:
  - VHD
  - Managed image
  - Azure Compute Gallery
- Added support for customers to use their own virtual network.
- Added support for customers to customize the build VM (VM size, operating system disk size).
- Added support for user-assigned managed identities (for customize/distribute steps).
- Added support for [Generation 2 images](image-builder-overview.md#hyper-v-generation).

### Preview API

The following API is deprecated but still supported:

- Version 2019-05-01-preview

## Related content

- [Azure VM Image Builder overview](image-builder-overview.md)
