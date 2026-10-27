---
title: "Restrict import and export access for managed disks using Azure Private Link"
description: Enable Azure Private Link for managed disks in the Azure portal to securely import and export disks within your virtual network.
author: roygara
ms.author: rogarana
ms.date: 09/16/2026
ms.service: azure-disk-storage
ms.topic: how-to
ms.custom:
  - sfi-image-nochange
  - ge-structured-content-pilot
---

# Restrict import and export access for managed disks by using Azure Private Link

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

In this article, you create a disk access resource and use [private endpoints](/azure/private-link/private-endpoint-overview) to restrict managed disk import and export over [Azure Private Link](/azure/private-link/private-link-overview) from clients on your Azure virtual network. This configuration ensures that import and export operations for configured disks occur within your Azure virtual network.

Following the steps in this article only affects the import and export of your disks, it doesn't affect the ability of your VMs to access disks directly attached to them.

## Limitations

[!INCLUDE [virtual-machines-disks-private-links-limitations](./includes/virtual-machines-disks-private-links-limitations.md)]

## Create a disk access resource

To use Private Link to import and export managed disks, create a disk access resource and link it to a virtual network in the same subscription by creating a private endpoint. Then, associate a disk or a snapshot with the disk access resource.

1. Sign in to the [Azure portal](https://portal.azure.com) and navigate to **Disk Accesses**.

1. Select **+ Create** to create a new disk access resource.

1. On the **Create a disk accesses** pane, select your subscription and a resource group. Under **Instance details**, enter a name and select a region.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-create-basics.png" alt-text="Screenshot of the Create a disk access pane with subscription, resource group, name, and region selected.":::

1. Select **Review + create**.

1. When your resource has been created, navigate directly to it.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/screenshot-resource-button.png" alt-text="Screenshot of the Go to resource button in the Azure portal.":::

## Create a private endpoint

Next, you'll need to create a private endpoint and configure it for disk access.

1. From your disk access resource, under **Settings**, select **Private endpoint connections**.

1. Select **+ Private endpoint**.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-main-private-blade.png" alt-text="Screenshot of a disk access resource with Private endpoint connections highlighted under Settings.":::

1. In the **Create a private endpoint** pane, select a resource group.

1. Provide a name and select the same region in which your disk access resource was created.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-first-blade.png" alt-text="Screenshot of the private endpoint Basics pane with resource group, endpoint name, and region selected.":::

1. Select **Next: Resource**.

1. On the **Resource** pane, select **Connect to an Azure resource in my directory**.

1. For **Resource type**, select **Microsoft.Compute/diskAccesses**.

1. For **Resource**, select the disk access resource you created earlier.

1. Leave the **Target sub-resource** as **disks**.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-second-blade.png" alt-text="Screenshot of the private endpoint Resource pane with a disk access resource and the disks target subresource selected.":::

1. Select **Next : Configuration**.

1. Select the virtual network to which you will limit disk import and export. This prevents the import and export of your disk to other virtual networks.
   
   > [!NOTE]
   > If you have a network security group enabled for the selected subnet, it will be disabled for private endpoints on this subnet only. Other resources on this subnet will retain network security group enforcement.

1. Select the appropriate subnet.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-third-blade.png" alt-text="Screenshot of the private endpoint Configuration pane with a virtual network, subnet, and private DNS integration selected.":::

1. Select **Review + create**.

## Configure a managed disk to use Private Link

Follow these steps:

1. Navigate to the disk you'd like to configure.

1. Under **Settings**, select **Networking**.

1. Select **Private endpoint (through disk access)** and select the disk access you created earlier.
   
   :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-managed-disk-networking-blade.png" alt-text="Screenshot of a managed disk Networking pane with Private endpoint through disk access and a disk access resource selected.":::

1. Select **Save**.
   
   You configured Private Link to import and export your managed disk. You can import by using the [Azure CLI](linux/disks-upload-vhd-to-managed-disk-cli.md) or the [Azure PowerShell module](windows/disks-upload-vhd-to-managed-disk-powershell.md). You can export either [Windows](windows/download-vhd.md) or [Linux](linux/download-vhd.md) VHDs.

## Related content

- [FAQ for Private Link and managed disks](/azure/virtual-machines/faq-for-disks#private-links-for-managed-disks)
- [Export/Copy managed snapshots as VHD to a storage account in different region with PowerShell](/previous-versions/azure/virtual-machines/scripts/virtual-machines-powershell-sample-copy-snapshot-to-storage-account)
- [Upload a VHD to Azure or copy a managed disk to another region - [Azure CLI]](linux/disks-upload-vhd-to-managed-disk-cli.md)
