---
title: Overview of Azure Compute Gallery
description: Learn about Azure Compute Gallery and how to share Azure resources.
author: cynthn
ms.author: cynthn
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: overview
ms.date: 09/20/2023
ms.reviewer: cynthn, mattmcinnes
# Customer intent: As a cloud architect, I want to use Azure Compute Gallery to manage and share image resources efficiently across multiple regions, so that I can enhance deployment scalability, maintain high availability, and optimize storage for my organization's applications and virtual machines.
---

# Store and share resources in Azure Compute Gallery

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure Compute Gallery helps you build structure and organization around your Azure resources, like images and [applications](vm-applications.md). Compute Gallery provides:

- Global replication.<sup>1</sup>
- Versioning and grouping of resources for easier management.
- Highly available resources with zone-redundant storage (ZRS) accounts, in regions that support availability zones. ZRS offers resilience against zonal failures.
- Premium storage support with locally redundant storage (LRS).
- Sharing to the community, across subscriptions, and between Active Directory tenants.
- Scaling your deployments with resource replicas in each region.

With a gallery, you can share your resources with everyone. Or you can limit sharing to particular users, service principals, or Active Directory groups within your organization. You can replicate resources to multiple regions, for quicker scaling of your deployments.

<sup>1</sup> The Azure Compute Gallery service isn't a global resource. For disaster recovery scenarios, the best practice is to have at least two galleries in separate regions.

## Images

For more information about storing images in Azure Compute Gallery, see [Store and share images in Azure Compute Gallery](shared-image-galleries.md).

## VM apps

Although you can create an image of a virtual machine (VM) with apps preinstalled, you would need to update your image each time an application changes. Separating your application installation from your VM images means there's no need to publish a new image for every line of code change.

For more information about storing applications in Azure Compute Gallery, see [VM applications](vm-applications.md).

## Regional support

All public regions can be target regions, but certain regions require that customers go through a request process to gain access. To request the addition of a subscription to the allowlist for a region such as Australia Central or Australia Central 2, submit an [access request](/troubleshoot/azure/general/region-access-request-process).

## Limits

The following limits apply when you deploy resources by using Azure Compute Gallery:

- You can have a maximum of 100 galleries per subscription, per region.
- You can have a maximum of 1,000 image definitions per subscription, per region.
- You can have a maximum of 10,000 image versions per subscription, per region.
- You can have a maximum 100 replicas per image version. However, 50 replicas should be sufficient for most use cases.
- The image size should be less than 2 TB, but you can use shallow replication to support larger image sizes (up to 32 TB).
- Resource movement isn't supported for Azure Compute Gallery resources.

For more information and for examples of how to check your current usage, see [Check resource usage against limits](/azure/networking/check-usage-against-limits).

## Scaling

Azure Compute Gallery allows you to specify the number of replicas that you want to keep. In multiple-VM deployment scenarios, you can spread the VM deployments to other replicas. This action reduces the chance that overloading of a single replica throttles the process of instance creation.

With Compute Gallery, you can deploy up to 1,000 VM instances in a scale set. You can set a different replica count in each target region, based on the scale needs for the region. Because each replica is a copy of your resource, this approach helps scale your deployments linearly with each extra replica.

Although we understand that no two resources or regions are the same, here are general guidelines on how to use replicas in a region:

- For every 50 VMs that you create concurrently, we recommend that you keep one replica. For example, if you're creating 500 VMs concurrently by using the same image in a region, we suggest you keep at least 10 replicas of your image.
- For each scale set that you create concurrently, we recommend that you keep one replica.

We always recommend that you overprovision the number of replicas due to factors like resource size, content, and OS type.

![Diagram that shows how you can scale images.](./media/shared-image-galleries/scaling.png)

## High availability

[Azure ZRS](https://azure.microsoft.com/blog/azure-zone-redundant-storage-in-public-preview/) provides resilience against the failure of an availability zone in a region. With the general availability of Azure Compute Gallery, you can choose to store your images in ZRS accounts in regions that have availability zones.

You can also choose the account type for each target region. The default storage account type is Standard LRS, but you can choose Standard ZRS for regions that have availability zones. For more information on the regional availability of ZRS, see [Azure Storage redundancy](/azure/storage/common/storage-redundancy).

![Diagram that shows zone-redundant storage.](./media/shared-image-galleries/zrs.png)

## Replication

Azure Compute Gallery also allows you to replicate your resources to other Azure regions automatically. You can replicate each image version to different regions, depending on what makes sense for your organization. One example is to always replicate the latest image in multiple regions while all older image versions are available in only one region. This approach can help you save storage costs.

After creation time, you can update the regions that a resource is replicated to. The time that replication to other regions takes depends on the amount of data that you're copying and the number of regions that the version is replicated to. This process can take a few hours in some cases.

While the replication is happening, you can view its status per region. After the image replication is complete in a region, you can deploy a VM or scale set by using that resource in the region.

![Diagram that shows the replication of an image across regions.](./media/shared-image-galleries/replication.png)

<a name=community></a>

## Trusted Launch validation for Azure Compute Gallery images (preview)

Trusted Launch validation for Azure Compute Gallery images is currently in preview. This preview is intended for testing, evaluation, and feedback purposes only. We don't recommend it for production workloads.

When you register for the preview, you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change with general availability.

### What's changing

Starting with API 2025-03-03, all new Compute Gallery image definitions default to:

- `hyperVGeneration`: `V2`
- `SecurityType`: `TrustedLaunchSupported`

### How Trusted Launch validation works

When you specify `TrustedLaunchSupported` or `TrustedLaunchandConfidentialVMSupported` in the Compute Gallery image definition, the platform automatically validates that the image is Trusted Launch capable. The platform adds the validation result to the image version property. This action ensures that the VM and scale set deployments that use these images can default to Trusted Launch if the validation is successful.

### How to enable the preview

To try out Trusted Launch validation for Compute Gallery images, complete the following steps:

1. Register for [Trusted Launch as a default feature](/azure/virtual-machines/trusted-launch#preview-trusted-launch-as-default).
2. Register for the [Trusted Launch Validation preview](https://aka.ms/ACGTLValidationPreview).

After you enable the two features, all new VM and scale set deployments that use Compute Gallery image versions and that are validated successfully for Trusted Launch default to the Trusted Launch security type.

### VM deployments from Azure Compute Gallery images

The following sections compare the current and new behaviors of VM deployments from Compute Gallery images, depending on Trusted Launch validation.

#### Current behavior without Trusted Launch validation

To create a Trusted Launch-supported, Gen2 OS image definition for Compute Gallery, you need to add the following `features` element in your deployment:

```json
"features": [
    {
      "name": "SecurityType",
      "value": "TrustedLaunchSupported"
    }
],
"hyperVGeneration": "V2"
```

#### New behavior with Trusted Launch validation

`TrustedLaunchSupported` is enabled by default on new Compute Gallery image definitions if the deployment meets any of the following criteria:

- The use of API version 2025-03-03 or later for the `Microsoft.Compute/galleries` resource
- The absence of the `SecurityType` feature
- A `null` value for the `SecurityType` feature

Additionally, the Azure platform triggers validation for the OS image to ensure that it supports Trusted launch capabilities. The validation takes a minimum of one hour. Results are available as the image version property:

```json
"validationsProfile": {
  "executedValidations": [
      {
        "type": "TrustedLaunch",
        "status": "Succeeded",
        "version": "0.0.2",
        "executionTime": "2025-07-10T21:27:33.0113984+00:00"
      }
    ],
}
```

You can choose to explicitly bypass the default for new Compute Gallery image definitions by setting `Standard` as the value for `SecurityType` under `features`:

```json
"features": [
    {
      "name": "SecurityType",
      "value": "Standard"
    }
],
"hyperVGeneration": "V2"
```

## Sharing

There are three main ways to share images in Azure Compute Gallery, depending on which users you want to share with:

| Sharing with: | People | Groups | Service principal | All users in a specific subscription or tenant | Publicly with all users in Azure |
|---|---|---|---|---|---|
| [Role-based access control (RBAC) sharing](#rbac) | Yes | Yes | Yes | No | No |
| RBAC + [direct shared gallery](#directly-sharing-with-a-tenant-or-subscription)  | Yes | Yes | Yes | Yes | No |
| RBAC + [community gallery](#community-gallery) | Yes | Yes | Yes | No | Yes |

> [!NOTE]
> You can use images with read permissions on them to deploy virtual machines and disks.
>
> When you use a direct shared gallery, images are distributed widely to all users in a subscription or tenant. A community gallery distributes images publicly. When you share images that contain intellectual property, use caution to prevent widespread distribution.

### RBAC

Because the gallery, definition, and version are all resources, you can share them by using the built-in Azure RBAC roles. By using Azure RBAC roles, you can share these resources with other users, service principals, and groups. You can even share access with individuals outside the tenant where you created the resources.

After users have access to the resource version, they can use it to deploy a VM or a virtual machine scale set. Here's a sharing matrix that can help you understand what the users get access to:

| Shared with users | Compute gallery | Image definition | Image version |
|----------------------|----------------------|--------------|----------------------|
| Compute gallery | Yes                  | Yes          | Yes                  |
| Image definition     | No                   | Yes          | Yes                  |

We recommend sharing at the gallery level for the best experience. We don't recommend sharing individual image versions. For more information about Azure RBAC, see [Assign Azure roles](/azure/role-based-access-control/role-assignments-portal) and [Share by using RBAC](./share-gallery.md).

### Directly sharing with a tenant or subscription

You can give specific subscriptions or tenants access to a direct shared gallery. Sharing a gallery with tenants and subscriptions gives them read-only access to your gallery. For more information, see [Share a gallery with all users in a subscription or tenant](./share-gallery-direct.md).

#### Important considerations

- The feature of direct shared galleries is currently in preview and is subject to the [preview terms for Azure Compute Gallery](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

- To publish images to a direct shared gallery during the preview, you need to [register for the preview](https://aka.ms/directsharedgallery-preview). Creating VMs from a direct shared gallery is open to all Azure users.

- During the preview, you need to create a new gallery with the `sharingProfile.permissions` property set to `Groups`. When you use the Azure CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, and the property can't currently be updated.

- You can't currently create a flexible virtual machine scale set from an image that another tenant shares with you.

#### Limitations

During the preview:

- You can share only with subscriptions that are also in the preview.

- You can share up to 30 subscriptions and 5 tenants.

- A direct shared gallery can't contain encrypted image versions.

- Only the owner of a subscription, or a user or service principal assigned to the Compute Gallery Sharing Admin role at the subscription or gallery level, can enable group-based sharing.

### Community gallery

To share a gallery with all Azure users, you can create a community gallery. Anyone who has an Azure subscription can use community galleries. Someone who creates a VM can browse through images shared with the community by using the Azure portal, the REST API, or the Azure CLI. Sharing images to the community is a new capability in Compute Gallery.

You can make your image galleries public and share them with Azure customers. When a gallery is marked as a community gallery, all images in it become available to all Azure customers as a new resource type under `Microsoft.Compute/communityGalleries`. Azure customers can see the galleries and use them to create VMs. Your original resources of the type `Microsoft.Compute/galleries` are still under your subscription and private.

For more information, see [Share images by using a community gallery](./share-gallery-community.md).

## Activity log

The [activity log](/azure/azure-monitor/essentials/activity-log) displays recent activity on the gallery, image, or version. This information includes any configuration changes and when the item was created and deleted.

You can view the activity log in the Azure portal or create a [diagnostic setting to send the log to a Log Analytics workspace](/azure/azure-monitor/essentials/activity-log#send-to-log-analytics-workspace). In the workspace, you can view events over time or analyze them with other collected data.

The following table lists a few example operations that relate to gallery operations in the activity log. For a complete list of possible log entries, see [the Microsoft.Compute Resource Provider options](/azure/role-based-access-control/resource-provider-operations#compute).

| Operation | Description |
|:---|:---|
| `Microsoft.Compute/galleries/write` | Creates a new gallery or updates an existing one |
| `Microsoft.Compute/galleries/delete`  | Deletes the gallery |
| `Microsoft.Compute/galleries/share/action` | Shares a gallery with different scopes |
| `Microsoft.Compute/galleries/images/read`  | Gets the properties of a gallery image |
| `Microsoft.Compute/galleries/images/write`  | Creates a new gallery image or updates an existing one |
| `Microsoft.Compute/galleries/images/versions/read`  | Gets the properties of a gallery image version |

## Billing

There's no extra charge for using the Azure Compute Gallery service. However, you're charged for the following resources:

- Storage costs for each replica. For images, the storage cost is charged as a snapshot. It's based on the occupied size of the image version, the number of replicas of the image version, and the number of regions that the version is replicated to.
- Network egress charges for replication of the first resource version from the source region to the replicated regions. Subsequent replicas are handled within the region, so there are no additional charges.

For example, let's say:

- You have an image of a 127-GB OS disk that occupies only 10 GB of storage, and you have one empty 32-GB data disk. The occupied size of each image would be only 10 GB.
- The image is replicated to three regions, and each region has two replicas. There are six total snapshots, each using 10 GB.

In this example, you're charged for the storage cost of each snapshot, based on the occupied size of 10 GB. You pay network egress charges for the first replica to be copied to the additional two regions.

For more information on the pricing of snapshots in each region, see [Managed disks pricing](https://azure.microsoft.com/pricing/details/managed-disks/). For more information on network egress, see [Bandwidth pricing](https://azure.microsoft.com/pricing/details/bandwidth/).

## Best practices

- To prevent accidental deletion of images, use resource locks at the gallery level. For more information, see [Lock your Azure resources to protect your infrastructure](/azure/azure-resource-manager/management/lock-resources).

- Use ZRS wherever it's available, for high availability. You can configure ZRS on the replication tab when you create a version of the image or VM application. For information about which regions support ZRS, see [Azure regions list](/azure/reliability/availability-zones-region-support).

- Keep a minimum of three replicas for production images. For every 20 VMs that you create concurrently, we recommend that you keep one replica.

  For example, if you create 1,000 VMs concurrently, you should keep 50 replicas. (You can have a maximum of 50 replicas per region.) To update the replica count, go to the gallery and select **Image Definition** > **Image Version** > **Update replication**.

- Maintain separate galleries for production and test images. Don't put both image types in a single gallery.

- For disaster recovery scenarios, have at least two galleries in separate regions. You can still use image versions in other regions, but if the region for one gallery goes down, you can't create new gallery resources or update existing ones.

- When you create an image definition, keep the **Publisher**, **Offer**, and **SKU** values consistent with Azure Marketplace images to easily identify OS versions.

  For example, if you're customizing a Windows Server 2019 image from Azure Marketplace and you store it as a Compute Gallery image, use the same **Publisher**, **Offer**, and **SKU** values that you used in the Azure Marketplace image.

- If you want to exclude a specific image version during creation of a VM or scale set, use `excludeFromLatest` when you publish images. See [Gallery image versions - create or update](/rest/api/compute/gallery-image-versions/create-or-update#galleryimageversionpublishingprofile).

- If you want to exclude a version in a specific region, use `regionalExcludeFromLatest` instead of the global `excludeFromLatest`. You can set both the global and regional `excludeFromLatest` flag, but the regional flag takes precedence.

   ```
    "publishingProfile": {
      "targetRegions": [
        {
          "name": "brazilsouth",
          "regionalReplicaCount": 1,
          "regionalExcludeFromLatest": false,
          "storageAccountType": "Standard_LRS"
        },
        {
          "name": "canadacentral",
          "regionalReplicaCount": 1,
          "regionalExcludeFromLatest": true,
          "storageAccountType": "Standard_LRS"
        }
      ],
      "replicaCount": 1,
      "excludeFromLatest": true,
      "storageAccountType": "Standard_LRS"
    }
   ```

- To prevent accidental deletion of replicated regions and prevent outages, set `safetyProfile.allowDeletionOfReplicatedLocations` to `false` on image versions. You can also set this property by using [`allow-replicated-location-deletion`](/cli/azure/sig/image-version#az-sig-image-version-create) in the Azure CLI.

   ```
   {
   "properties": { 
    "publishingProfile": { 
      "targetRegions": [ 
        { 
          "name": "West US", 
          "regionalReplicaCount": 1, 
          "storageAccountType": "Standard_LRS", 
          // encryption info         
        }
      ], 
       "replicaCount": 1, 
       "publishedDate": "2018-01-01T00:00:00Z", 
       "storageAccountType": "Standard_LRS" 
    }, 
    "storageProfile": { 
       "source": { 
         "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/images/{imageName}" 
      }, 
    }, 
    "safetyProfile": { 
       "allowDeletionOfReplicatedLocations" : false 
     }, 
   }, 
   "location": "West US", 
   "name": "1.0.0" 
   } 
   ```

- To block deletion of the image before its end-of-support date, set `BlockDeletionBeforeEndOfLife`. This feature helps protect against accidental deletion. You set it by using [ `blockdeletionbeforeendoflife` in the REST API](/rest/api/compute/gallery-image-versions/create-or-update?view=rest-compute&preserve-view=true&tabs=HTTP#galleryimageversionsafetyprofile).

## SDK support

The following SDKs support creating galleries:

- [.NET](/dotnet/api/overview/azure/virtualmachines#management-apis)
- [Java](/java/azure/)
- [Node.js](/javascript/api/overview/azure/arm-compute-readme)
- [Python](/python/api/overview/azure/virtualmachines)
- [Go](/azure/go/)

## Templates

You can create Azure Compute Gallery resources by using quickstart templates:

- [Create a gallery](https://azure.microsoft.com/resources/templates/sig-create/)
- [Create an image definition in a gallery](https://azure.microsoft.com/resources/templates/sig-image-definition-create/)
- [Create an image version in a gallery](https://azure.microsoft.com/resources/templates/sig-image-version-create/)

## Related content

- Learn how to deploy [images](shared-image-galleries.md) and [VM apps](vm-applications.md) by using Azure Compute Gallery.
