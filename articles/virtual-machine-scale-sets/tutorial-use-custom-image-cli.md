---
title: Tutorial - Use a custom VM image in a scale set with Azure CLI
description: Learn how to use the Azure CLI to create a custom VM image that you can use to deploy a Virtual Machine Scale Set
author: ju-shim
ms.service: azure-virtual-machine-scale-sets
ms.subservice: shared-image-gallery
ms.topic: tutorial
ms.date: 10/28/2024
ms.reviewer: mimckitt
ms.author: jushiman
ms.custom: mvc, devx-track-azurecli, innovation-engine
# Customer intent: As a cloud architect, I want to create and utilize a custom VM image for a scale set using the CLI, so that I can efficiently deploy pre-configured VM instances that meet my application's requirements.
---

# Tutorial: Create and use a custom image for Virtual Machine Scale Sets with the Azure CLI

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://go.microsoft.com/fwlink/?linkid=2303113)

When you create a scale set, you specify an image to be used when the VM instances are deployed. To reduce the number of tasks after VM instances are deployed, you can use a custom VM image. This custom VM image includes any required application installs or configurations. Any VM instances created in the scale set use the custom VM image and are ready to serve your application traffic. In this tutorial you learn how to:

> [!div class="checklist"]
> * Create an Azure Compute Gallery
> * Create a specialized image definition
> * Create an image version
> * Create a scale set from a specialized image
> * Share an image gallery

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This article requires version 2.4.0 or later of the Azure CLI. If using Azure Cloud Shell, the latest version is already installed.

## Overview
An [Azure Compute Gallery](../virtual-machines/shared-image-galleries.md) simplifies custom image sharing across your organization. Custom images are like marketplace images, but you create them yourself. Custom images can be used to bootstrap configurations such as preloading applications, application configurations, and other OS configurations.

The Azure Compute Gallery lets you share your custom VM images with others. Choose which images you want to share, which regions you want to make them available in, and who you want to share them with.

## Create and configure a source VM
First, create a resource group with [az group create](/cli/azure/group), then create a VM with [az vm create](/cli/azure/vm#az-vm-create). This VM is then used as the source for the image. 

The following example creates a Linux-based VM named *myVM* in the resource group named *myResourceGroup*. 

```azurecli-interactive
export RANDOM_ID=$(openssl rand -hex 3)
export MY_RESOURCE_GROUP_NAME="myResourceGroup$RANDOM_ID"
export REGION="eastus"
export MY_VM_NAME="myVM"

az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION

az vm create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name $MY_VM_NAME \
  --image debian11 \
  --admin-username azureuser \
  --generate-ssh-keys
```

> [!TIP] 
> The **ID** of your VM is shown in the output of the [az vm create](/cli/azure/vm#az-vm-create) command. Copy and store this in a safe location so you can use it later in this tutorial.

## Create an image gallery
An image gallery is the primary resource used for enabling image sharing.

Allowed characters for gallery names are uppercase or lowercase letters, digits, dots, and periods. The gallery name can't contain dashes.   Gallery names must be unique within your subscription.

Create an image gallery using [az sig create](/cli/azure/sig#az-sig-create). 

In the following example:

* You create a resource group for the gallery named *myGalleryRG* located in *East US*.
* The gallery is named *myGallery*.

```azurecli-interactive
export MY_GALLERY_RG_NAME="myGalleryRG$RANDOM_ID"
export MY_GALLERY_NAME="myGallery$RANDOM_ID"

az group create --name $MY_GALLERY_RG_NAME --location $REGION
az sig create --resource-group $MY_GALLERY_RG_NAME --gallery-name $MY_GALLERY_NAME
```

## Create an image definition
Image definitions create a logical grouping for images. They're used to manage information about the image versions that are created within them.

Image definition names can be made up of uppercase or lowercase letters, digits, dots, dashes, and periods.

Make sure your image definition is the right type: 

* **State** - If you have generalized the VM (using Sysprep for Windows, or waagent -deprovision for Linux), then you should create a generalized image definition using `--os-state generalized`. If you want to use the VM without removing existing user accounts, create a specialized image definition using `--os-state specialized`.
* **Security type** - New Azure VMs are created with Trusted Launch configured by default. This tutorial includes subsequent code samples that reflect the Trusted Launch configuration when creating the image definition and scale set. If you're creating an image with a VM that doesn't have Trusted Launch enabled, make sure to reflect the correct security type when you create both of those resources. For more information about Trusted Launch, see [Trusted Launch for Azure virtual machines](/azure/virtual-machines/trusted-launch).

For more information about the values you can specify for an image definition, see [Image definitions](../virtual-machines/shared-image-galleries.md#image-definitions).

Create an image definition in the gallery using [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

In the following example, the image definition is:
* Named *myImageDefinition*.
* Configured for a [specialized](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) Linux OS image. To create a definition for images using a Windows OS, use `--os-type Windows`.
* Configured for Trusted Launch.

```azurecli-interactive
export MY_IMAGE_DEF_NAME="myImageDefinition$RANDOM_ID"
MY_PUBLISHER_NAME="myPublisher$RANDOM_ID"

az sig image-definition create \
   --resource-group $MY_GALLERY_RG_NAME \
   --gallery-name $MY_GALLERY_NAME \
   --gallery-image-definition $MY_IMAGE_DEF_NAME \
   --publisher $MY_PUBLISHER_NAME \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized \
   --features SecurityType=TrustedLaunch
```

> [!TIP]
> The **ID** of your image definition is shown in the output of the command. Copy and store this in a safe location so you can use it later in this tutorial.

## Create the image version
Create an image version from the VM using [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).

Allowed characters for the image version are numbers and periods. Numbers must be within the range of a 32-bit integer. Format: *MajorVersion*.*MinorVersion*.*Patch*.

In the following example: 

* The version of the image is *1.0.0*.
* We create one replica in the *South Central US* region and one replica in the *East US* region. The replication regions must include the region the source VM is located.
* `--virtual-machine` is the ID of the VM we created previously.

```azurecli-interactive
export MY_VM_ID=$(az vm show --name $MY_VM_NAME --resource-group $MY_RESOURCE_GROUP_NAME --query "id" --output tsv)

az sig image-version create \
   --resource-group $MY_GALLERY_RG_NAME \
   --gallery-name $MY_GALLERY_NAME \
   --gallery-image-definition $MY_IMAGE_DEF_NAME \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1" \
   --virtual-machine $MY_VM_ID
```

> [!NOTE]
> You need to wait for the image version to completely finish being built and replicated before you can use the same image to create another image version.
>
> You can also store your image in Premium storage by a adding `--storage-account-type  premium_lrs`, or [Zone Redundant Storage](/azure/storage/common/storage-redundancy) by adding `--storage-account-type  standard_zrs` when you create the image version.


## Create a scale set from the image

You create a scale set using [`az vmss create`](/cli/azure/vmss#az-vmss-create). If you're using a specialized source VM, add the `--specialized` parameter to indicate it's a specialized image.

When you use the image definition ID for `--image` to create the scale set instances, you create a scale set that uses the latest version of the image that is available. If you want a specific version of the image, make sure you include the image _version_ ID when you define the `--image`.

* **Latest image example**: `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/galleries/myGallery/images/myImage`

* **Specific image example**: `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/galleries/myGallery/images/myImage/versions/1.0.0`

In the following example, the scale set is:
* Named *myScaleSet* 
* Using the latest version of the *myImageDefinition* image.
* Configured for Trusted Launch.

```azurecli
export MY_IMAGE_DEF_ID=$(az sig image-definition show --resource-group $MY_GALLERY_RG_NAME --gallery-name $MY_GALLERY_NAME --gallery-image-definition $MY_IMAGE_DEF_NAME --query "id" --output tsv)
export MY_SCALE_SET_RG_NAME="myResourceGroup$RANDOM_ID"
export MY_SCALE_SET_NAME="myScaleSet$RANDOM_ID"

az group create --name $MY_SCALE_SET_RG_NAME --location eastus

az vmss create \
   --resource-group $MY_SCALE_SET_RG_NAME \
   --name $MY_SCALE_SET_NAME \
   --orchestration-mode flexible \
   --image $MY_IMAGE_DEF_ID \
   --specialized \
   --security-type TrustedLaunch
```

It takes a few minutes to create and configure all the scale set resources and VMs.

## Share the gallery

You can share images across subscriptions using Azure role-based access control (Azure RBAC), and you can share them at the gallery, image definition, or image version levels. Any user with read permission to an image version, even across subscriptions, is able to deploy a VM using the image version.

We recommend that you share with other users at the gallery level. 

The following example:
* Gets the object ID of the gallery using [az sig show](/cli/azure/sig#az-sig-show).
* Provides access to the gallery using [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create).
    * Uses the object ID as the scope of the assignment.
    * Uses the signed-in user's ID as the assignee for demonstration purposes. When you use this code in your test or production code, make sure you update the assignee to reflect who you want to be able to access this image. For more information about how to share resources using Azure RBAC, see [Add or remove Azure role assignments using Azure CLI](/azure/role-based-access-control/role-assignments-cli). , along with an email address, using [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) to give a user access to the shared image gallery. 

```azurecli-interactive
export MY_GALLERY_ID=$(az sig show --resource-group $MY_GALLERY_RG_NAME --gallery-name $MY_GALLERY_NAME --query "id" --output tsv)
export CALLER_ID=$(az ad signed-in-user show --query id -o tsv)

az role assignment create \
   --role "Reader" \
   --assignee $CALLER_ID \
   --scope $MY_GALLERY_ID
```

## Clean up resources
To remove your scale set and additional resources, delete the resource group and all its resources with [az group delete](/cli/azure/group). The `--no-wait` parameter returns control to the prompt without waiting for the operation to complete. The `--yes` parameter confirms that you wish to delete the resources without an additional prompt to do so.

## Next steps
In this tutorial, you learned how to create and use a custom VM image for your scale sets with the Azure CLI:

> [!div class="checklist"]
> * Create an Azure Compute Gallery
> * Create a specialized image definition
> * Create an image version
> * Create a scale set from a specialized image
> * Share an image gallery

Advance to the next tutorial to learn how to deploy applications to your scale set.

> [!div class="nextstepaction"]
> [Deploy applications to your scale sets](tutorial-install-apps-cli.md)
