---
title: Download a Windows VHD from Azure
description: Download a Windows VHD using the Azure portal.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 02/19/2026
# Customer intent: "As a cloud administrator, I want to download a Windows VHD from Azure, so that I can create new virtual machines or back up the current VM configuration."
---

# Download a Windows VHD from Azure

**Applies to:** :heavy_check_mark: Windows VMs 

This article explains how to download a Windows virtual hard disk (VHD) file from Azure. To download a VHD, the disk can't be attached to a running VM, which means the VM experiences downtime. However, some configurations can safely avoid downtime by [snapshotting the disk](#alternative-snapshot-the-vm-disk) and downloading the VHD from the snapshot.

If you're using [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can use it to restrict uploading of Azure managed disks. For more information, see [Secure downloads and uploads of Azure managed disks](../disks-secure-upload-download.md).

## Optional: Generalize the VM

If you want to use the VHD as an [image](tutorial-custom-images.md) to create other VMs, use [Sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) to generalize the operating system. Otherwise, you need to make a copy of the disk for each VM you want to create.

To use the VHD as an image to create other VMs, generalize the VM.

1. If you haven't already done so, sign in to the [Azure portal](https://portal.azure.com/).
1. [Connect to the VM](connect-logon.md). 
1. On the VM, open the Command Prompt window as an administrator.
1. Change the directory to *%windir%\system32\sysprep* and run sysprep.exe.
1. In the System Preparation Tool dialog box, select **Enter System Out-of-Box Experience (OOBE)**, and make sure that **Generalize** is selected.
1. In Shutdown Options, select **Shutdown**, and then select **OK**. 

If you don't want to generalize your current VM, you can still create a generalized image by first [making a snapshot of the OS disk](#alternative-snapshot-the-vm-disk), creating a new VM from the snapshot, and then generalizing the copy.

## Stop the VM

You can't download a VHD from Azure if it's attached to a running VM. If you want to keep the VM running, you can [create a snapshot and then download the snapshot](#alternative-snapshot-the-vm-disk).

1. On the Hub menu in the Azure portal, select **Virtual Machines**.
1. Select the VM from the list.
1. On the blade for the VM, select **Stop**.

### Alternative: Snapshot the VM disk

> [!NOTE]
> If feasible, stop a VM before taking a snapshot of it, otherwise the snapshot isn't clean. Snapshots of running VMs are in the same state as if their VMs were power cycled or crashed when you take a snapshot. Usually this state is safe but, it could cause problems if the running applications aren't crash resistant.
>  
> Generally, you should only use snapshots of running VMs if the only disk associated with them is a single OS disk. If a VM has one or more data disks, stop the VM before creating a snapshot of the OS or data disks.

Take a snapshot of the disk to download.

1. Select the VM in the [portal](https://portal.azure.com).
1. Select **Disks** in the left menu and then select the disk you want to snapshot. The details of the disk are displayed.  
1. Select **Create Snapshot** from the menu at the top of the page. The **Create snapshot** page opens.
1. In **Name**, type a name for the snapshot. 
1. For **Snapshot type**, select **Full** or **Incremental**.
1. When you're done, select **Review + create**.

Your snapshot is created shortly, and you can then use it to download or create another VM.

## Generate download URL

To download the VHD file, you need to generate a [shared access signature (SAS)](/azure/storage/common/storage-sas-overview?toc=/azure/virtual-machines/windows/toc.json) URL. When you generate the URL, you assign an expiration time to it.

[!INCLUDE [disks-sas-change](../includes/disks-sas-change.md)]

# [Portal](#tab/azure-portal)

1. On the page for the VM, select **Disks** in the left menu.
1. Select the operating system disk for the VM.
1. On the page for the disk, select **Disk Export** from the left menu.
1. The default expiration time of the URL is *3,600 seconds (one hour). You might need to increase this value for Windows OS disks or large data disks. In these situations, **36000** seconds (10 hours) is usually sufficient.
1. Select **Generate URL**.

# [PowerShell](#tab/azure-powershell)

Replace `yourRGName` and `yourDiskName` with your values, and then run the following command to get your SAS.

```azurepowershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName "yourRGName" -DiskName "yourDiskName" -DurationInSecond 86400 -Access 'Read'
```

# [Azure CLI](#tab/azure-cli)

Replace `yourRGName` and `yourDiskName` with your values, and then run the following command to get your SAS.

```azurecli
az disk grant-access --duration-in-seconds 86400 --access-level Read --name yourDiskName --resource-group yourRGName
```

---


> [!NOTE]
> When downloading a Windows OS disk, you might need a longer expiration time to download a large VHD file. Large VHDs can take up to several hours to download depending on your connection and the size of the VM.
>
> While the SAS URL is active, attempting to start the VM results in the error **There is an active shared access signature outstanding for disk** *diskname*. You can revoke the SAS URL by selecting **Cancel export** on the **Disk Export** page.  

## Download VHD

> [!NOTE]
> If you're using Microsoft Entra ID to secure managed disk downloads, the user downloading the VHD needs the appropriate [RBAC permissions](../disks-secure-upload-download.md#assign-rbac-role).

# [Portal](#tab/azure-portal)

1. Under the generated URL, select **Download the VHD file**.
1. You might need to select **Save** in your browser to start the download. The default name for the VHD file is *abcd*.

# [PowerShell](#tab/azure-powershell)

Use the following script to download your VHD:

```azurepowershell
Connect-AzAccount
#Set localFolder to your desired download location
$localFolder = "c:\tempfiles"
$blob = Get-AzStorageBlobContent -Uri $diskSas.AccessSAS -Destination $localFolder -Force 
```

When the download finishes, revoke access to your disk by using `Revoke-AzDiskAccess -ResourceGroupName "yourRGName" -DiskName "yourDiskName"`.

# [Azure CLI](#tab/azure-cli)

Replace `yourPathhere` and `sas-URI` with your values. Then use the following script to download your VHD:

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

- Learn how to [upload a VHD file to Azure](upload-generalized-managed.md). 
- [Create managed disks from unmanaged disks in a storage account](attach-disk-ps.md).
- [Manage Azure disks with PowerShell](tutorial-manage-data-disk.md).
