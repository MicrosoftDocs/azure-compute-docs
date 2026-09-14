---
title: Migrate Azure Compute Gallery resources from China North and China East
description: Learn how to migrate Azure Compute Gallery resources from the retired China North and China East regions in Microsoft Azure operated by 21Vianet.
author: sandeepraichura
ms.author: saraic
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/09/2026
ms.reviewer: wwilliams
ms.custom: references_regions
# Customer intent: As an Azure Compute Gallery administrator, I want to migrate resources from the retired China North and China East regions so that my images and dependent workloads remain available.
---

# Migrate Azure Compute Gallery resources from China North and China East

China North (`chinanorth`) and China East (`chinaeast`) in Microsoft Azure operated by 21Vianet retire on July 1, 2026. If you have Azure Compute Gallery resources in either region, use this article to understand which resources Microsoft migrates and which resources you must replace.

> [!IMPORTANT]
> On **September 30, 2026**, gallery image versions whose home region is China North or China East stop working and are permanently deleted. This deletion includes replicas in regions that didn't retire. You can't restore the versions after they're deleted.

Microsoft is migrating affected resource groups, galleries, and gallery image definitions to China North 3 (`chinanorth3`). Microsoft doesn't migrate gallery image versions. You must publish replacement versions in a surviving region for any versions that you still need.

## Understand what is migrated

The following table summarizes the migration responsibility for each resource type.

| Resource type | Migration responsibility | Required action |
| --- | --- | --- |
| Resource group | Microsoft | No action is required for the resource group itself. Handle resources in the group separately. |
| Gallery | Microsoft | No action is required. The gallery retains its name, resource ID, and properties when its region changes to China North 3. Review automation that refers to the gallery by region. |
| Gallery image definition | Microsoft | No action is required for the image definition itself. The definition moves with its gallery. |
| Gallery image version | Customer | Create a replacement image version in a surviving region if you still need the version. Update all dependent deployments to use the replacement. |

An image definition contains metadata such as the publisher, offer, SKU, and operating system type. An image version contains the deployable image data. Migrating a gallery and its image definitions doesn't migrate the versions within those definitions.

## Identify affected image versions and dependencies

Inventory every gallery image version whose home region is China North or China East. For each version, determine whether any of the following resources or processes use it:

- Virtual machines (VMs) and Virtual Machine Scale Sets in any region.
- Resources in other subscriptions or Microsoft Entra tenants that consume a shared gallery.
- Deployment templates, automation scripts, image build pipelines, and disaster recovery runbooks.
- Rebuild, rollback, or audit processes that might use the version infrequently.

Don't limit your review to workloads running in China North and China East. An affected image version might have replicas in China North 2, China North 3, or another surviving region. A replica can't outlive the image version in its home region and is permanently deleted with the version.

The absence of a currently deployed VM or scale set isn't proof that a version is unused. Check deployment definitions and processes that might not have run recently.

### Find affected resources with Azure Resource Graph

Use the following Azure Resource Graph queries to inventory affected resources. In the [Azure portal operated by 21Vianet](https://portal.azure.cn), open **Resource Graph Explorer**, replace `{SUBSCRIPTION_ID}` with your subscription ID, and then run each query.

#### Galleries and gallery image definitions

The following query lists galleries and gallery image definitions in China North and China East. Microsoft migrates these resources and you don't need to take any migration action. The results include the resource group that contains each resource.

```kusto
resources
| where subscriptionId =~ '{SUBSCRIPTION_ID}'
| where type in~ (
	'microsoft.compute/galleries',
	'microsoft.compute/galleries/images')
| where location in~ ('chinanorth', 'chinaeast')
| extend resourceType = case(
	type =~ 'microsoft.compute/galleries', 'Gallery',
	'Gallery image definition')
| project resourceType, name, resourceGroup, location, id
| order by resourceGroup asc, id asc
```

#### Gallery image versions

The following query lists gallery image versions whose home region is China North or China East. Take action for every version that you need to retain.

```kusto
resources
| where subscriptionId =~ '{SUBSCRIPTION_ID}'
| where type =~ 'microsoft.compute/galleries/images/versions'
| where location in~ ('chinanorth', 'chinaeast')
| mv-expand targetRegion = properties.publishingProfile.targetRegions
| summarize
    replicationRegions = make_list(tostring(targetRegion.name))
    by id, homeRegion = location
| project id, homeRegion, replicationRegions
| order by id asc
```

#### VMs and scale sets that use an affected gallery image

The following query lists VMs and Virtual Machine Scale Sets that reference versions of a specified gallery image definition. In the image reference filter, replace `<gallery>` and `<image-definition>` with the affected gallery and image definition names. Select all relevant subscriptions in the Resource Graph Explorer scope before you run the query. The workloads might be in regions other than China North and China East.

```kusto
resources
| where type in~ (
	"microsoft.compute/virtualmachines",
	"microsoft.compute/virtualmachinescalesets"
)
| extend imageReferenceId = case(
	type =~ "microsoft.compute/virtualmachines",
		tostring(properties.storageProfile.imageReference.id),
	tostring(properties.virtualMachineProfile.storageProfile.imageReference.id)
)
| where imageReferenceId contains "/providers/Microsoft.Compute/galleries/<gallery>/images/<image-definition>"
| project
	subscriptionId,
	resourceGroup,
	name,
	type,
	location,
	imageReferenceId
```

Run the queries for each subscription that might contain affected resources. The results don't identify deployment templates, pipelines, or other undeployed references, so complete the dependency review described in this section.

## Migrate an image version that you still need

Complete the following steps for each affected image version that you need to retain:

1. Identify every workload and deployment process that references the affected version. Include consumers in other subscriptions and tenants.
1. Locate the source used to create the version, such as a managed image, virtual hard disk (VHD), snapshot, VM, or image build pipeline.
1. Move or rebuild the source in a surviving region. A source located in China North or China East is also subject to the region retirement.
1. Create a gallery in a surviving region, or use an existing gallery there. You don't need to wait for Microsoft's migration of the affected gallery to finish.
1. Create a compatible image definition in the target gallery if one doesn't already exist.
1. [Create a replacement image version](image-version.md) from the source. Use a surviving region such as China North 3 as the home region, and replicate the version to every region where you deploy it.
1. [Deploy a test VM](vm-generalized-image-version.md) from the replacement version and validate the operating system, applications, configuration, and startup behavior. For a specialized image, see [Create a VM from a specialized image version](vm-specialized-image-version.md).
1. Update deployment templates, pipelines, and automation to reference the replacement version.
1. Update each affected scale set model to use the replacement version, and roll the model change out to existing instances according to the scale set's upgrade policy. For more information, see [Modify a Virtual Machine Scale Set](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set).
1. Coordinate with the owners of consumers in other subscriptions or tenants. They must update resources that you don't administer.
1. Confirm that no required deployment or reimage operation still depends on the affected version. You can then delete the old version.

Start by finding external consumers and locating the original source artifacts. Missing sources, sources in a retired region, and consumers that you don't administer can add significant time to the migration.

## Handle an image version with no detected use

For each affected version that has no detected VM or scale set references:

1. Check templates, pipelines, runbooks, rollback plans, and audit requirements for references that aren't represented by currently deployed resources.
1. If you still need the version, follow the migration procedure in this article and publish a replacement in a surviving region.
1. If you no longer need the version, delete it before the retirement cleanup or allow it to be permanently deleted on September 30, 2026.
1. Record a keep-or-delete decision for every affected version.

## Expected impact if you don't migrate

On September 30, 2026, affected gallery image versions and all their replicas are permanently deleted. Existing running VMs aren't stopped solely because their source image version is deleted. However, operations that require the deleted version fail, including:

- Creating new VMs from the version.
- Scaling out a scale set that references the version.
- Reimaging a VM or scale set instance that depends on the version.
- Running templates, pipelines, or automation that reference the version.

After deletion, you must build and publish a replacement from the original source. Soft delete doesn't provide recovery for image versions deleted as part of this region retirement.

## Frequently asked questions

### Which regions are affected, and where should I migrate my resources?

China North (`chinanorth`) and China East (`chinaeast`) were retired on July 1, 2026. The recommended target region is China North 3 (`chinanorth3`).

### My gallery is being migrated automatically. Does that include everything in the gallery?

No. Microsoft migrates the gallery and its image definitions, but not its image versions. You must publish replacement image versions in a surviving region before September 30, 2026 for any versions that you still need.

### Do I need to opt in to the migrations that Microsoft performs?

No. Affected resource groups, galleries, and gallery image definitions are automatically included in the platform migration to China North 3.

### Does my gallery keep its name and resource ID after migration?

Yes. The gallery keeps its name, resource ID, and resource properties. Only its region changes. References that use the resource ID continue to resolve after migration. Review templates, scripts, pipelines, and documentation that refer to the gallery by region and update them to use China North 3.

### Can I publish replacement image versions before my gallery is migrated?

Yes. Start now. You can publish a replacement version to an existing gallery in a surviving region or create a gallery there. Publishing replacement versions doesn't depend on the platform migration of your affected gallery or image definitions.

### Can Microsoft migrate my gallery image versions?

No. There's no platform-serviced migration for gallery image versions. You must create replacement versions in a surviving region from their original sources.

### How can I identify my affected resources?

Inventory gallery image versions whose home region is China North or China East, and follow the dependency review in [Identify affected image versions and dependencies](#identify-affected-image-versions-and-dependencies). Check all subscriptions, tenants, regions, deployment templates, pipelines, and automation that might consume the versions. Don't rely only on currently deployed VMs and scale sets, because infrequently used deployment processes might also reference an affected version.

### Is an affected image version safe if it has a replica in a surviving region?

No. A replica can't outlive the image version in its home region. If the home region is China North or China East, the version and all its replicas are permanently deleted on September 30, 2026, including replicas in China North 2 and China North 3. Replication isn't a substitute for publishing a replacement version whose home region is a surviving region.

### I don't run workloads in China North or China East. Am I affected?

Possibly. VMs and scale sets in surviving regions might use replicas of image versions whose home region is China North or China East. Review dependencies in every region where you operate.

### What if an affected image version is used outside my subscription or tenant?

Notify the owners of those consuming resources as soon as possible. You can publish a replacement version and grant the required access, but only the resource owners can update workloads that you don't administer. Allow enough time for each owner to validate and deploy the replacement.

### What happens to existing VMs and scale sets after September 30, 2026?

Existing running VMs aren't stopped solely because their source image version is deleted. However, new VM deployments, scale-set scale-out operations, reimage operations, and automation that depend on the deleted version fail. Update scale-set models and deployment processes to use a replacement version before the deadline.

### What should I do if I can't complete the migration before the deadline?

Open an Azure support request in the [Azure portal operated by 21Vianet](https://portal.azure.cn) as soon as you identify a blocker. Don't wait until close to September 30, 2026.

## Get help

If you have questions, need more information about detected dependencies, or don't expect to complete the migration before September 30, 2026, open an Azure support request in the [Azure portal operated by 21Vianet](https://portal.azure.cn) as soon as possible.

## Related content

- [Store and share images in Azure Compute Gallery](shared-image-galleries.md)
- [Create an image definition and image version](image-version.md)
- [Create a VM from a generalized image version](vm-generalized-image-version.md)
- [Create a VM from a specialized image version](vm-specialized-image-version.md)
