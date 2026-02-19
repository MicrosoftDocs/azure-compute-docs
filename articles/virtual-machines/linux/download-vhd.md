---
title: Download a Linux VHD from Azure 
description: Download a Linux VHD using the Azure CLI and the Azure portal.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 02/19/2026
# Customer intent: As a system administrator, I want to download a Linux VHD from Azure, so that I can create backups or migrate virtual machines to another environment.
---

# Download a Linux VHD from Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article describes how to download a Linux virtual hard disk (VHD) file from Azure. To download a VHD, the disk can't be attached to a running VM, which means the VM experiences downtime. Some configurations can safely avoid downtime by [snapshotting the disk](#alternative-snapshot-the-vm-disk) and downloading the VHD from the snapshot.

If you're using [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can use it to restrict uploading of Azure managed disks. For more information, see [Secure downloads and uploads of Azure managed disks](../disks-secure-upload-download.md).

## Stop the VM

You can't download a VHD from Azure if the VHD is attached to a running VM. If you want to keep the VM running, you can [create a snapshot and then download the snapshot](#alternative-snapshot-the-vm-disk).

To stop the VM:

1.	Sign in to the [Azure portal](https://portal.azure.com/).
1.	On the left menu, select **Virtual Machines**.
1.	Select the VM from the list.
1.	On the page for the VM, select **Stop**.

    :::image type="content" source="./media/download-vhd/export-stop.PNG" alt-text="Shows the menu button to stop the VM.":::

Once the VM is stopped, 

### Alternative: Snapshot the VM disk

> [!NOTE]
> If feasible, stop a VM before taking a snapshot of it. Otherwise, the snapshot isn't clean. Snapshots of running VMs are in the same state as if their VMs were power cycled or crashed when you take a snapshot. Usually, this state is safe but it could cause problems if the running applications at the time aren't crash resistant.
>  
> Generally, you should only use snapshots of running VMs if the only disk associated with them is a single OS disk. If a VM has one or more data disks, stop the VM before creating a snapshot of the OS or data disks.

Take a snapshot of the disk to download.

1. Select the VM in the [portal](https://portal.azure.com).
1. Select **Disks** in the left menu and then select the disk you want to snapshot. The details of the disk are displayed.  
1. Select **Create Snapshot** from the menu at the top of the page. The **Create snapshot** page opens.
1. In **Name**, type a name for the snapshot. 
1. For **Snapshot type**, select **Full** or **Incremental**.
1. When you're done, select **Review + create**.

After you create your snapshot, you can use it to download a VHD or create another VM.

## Generate SAS URL

To download the VHD file, you need to generate a [shared access signature (SAS)](/azure/storage/common/storage-sas-overview?toc=/azure/virtual-machines/windows/toc.json) URL. When the URL is generated, an expiration time is assigned to the URL.

[!INCLUDE [disks-sas-change](../includes/disks-sas-change.md)]

# [Portal](#tab/azure-portal)

1. On the menu of the page for the VM, select **Disks**.
1. Select the operating system disk for the VM, and then select **Disk Export**.
1. If necessary, update the value of **URL expires in (seconds)** to give you enough time to complete the download. The default is 3,600 seconds (one hour).
1. Select **Generate URL**.

# [PowerShell](#tab/azure-powershell)

```azurepowershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName "yourRGName" -DiskName "yourDiskName" -DurationInSecond 86400 -Access 'Read'
```

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk grant-access --duration-in-seconds 86400 --access-level Read --name yourDiskName --resource-group yourRGName
```

---
      
## Download VHD

> [!NOTE]
> If you're using Microsoft Entra ID to secure managed disk downloads, the user downloading the VHD needs the appropriate [RBAC permissions](../disks-secure-upload-download.md#assign-rbac-role).

# [Portal](#tab/azure-portal)

1.	Under the generated URL, select **Download the VHD file**.

    :::image type="content" source="./media/download-vhd/export-download.PNG" alt-text="Shows the button to download the VHD.":::

1.	You might need to select **Save** in the browser to start the download. The default name for the VHD file is *abcd*.

# [PowerShell](#tab/azure-powershell)

Use the following script to download your VHD:

```azurepowershell
Connect-AzAccount
#Set localFolder to your desired download location
$localFolder = "yourPathHere"
$blob = Get-AzStorageBlobContent -Uri $diskSas.AccessSAS -Destination $localFolder -Force 
```

When the download finishes, revoke access to your disk by using `Revoke-AzDiskAccess -ResourceGroupName "yourRGName" -DiskName "yourDiskName"`.

# [Azure CLI](#tab/azure-cli)

Replace `yourPathhere` and `sas-URI` with your values. Then, use the following script to download your VHD:

> [!NOTE]
> If you're using Microsoft Entra ID to [secure your managed disk](../disks-secure-upload-download.md) uploads and downloads, add `--auth-mode login` to `az storage blob download`.

```azurecli

#set localFolder to your desired download location
localFolder=yourPathHere
#If you're using Azure AD to secure your managed disk uploads and downloads, add --auth-mode login to the following command.
az storage blob download -f $localFolder --blob-url "sas-URI"
```

When the download finishes, revoke access to your disk by using `az disk revoke-access --name diskName --resource-group yourRGName`.

---

## Next steps

- Learn how to [upload and create a Linux VM from custom disk with the Azure CLI](upload-vhd.md). 
- [Manage Azure disks the Azure CLI](tutorial-manage-disks.md).
