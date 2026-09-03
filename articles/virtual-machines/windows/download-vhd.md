---
title: Download a Windows VHD from Azure
description: Download a Windows VHD using the Azure portal, Azure PowerShell, or the Azure CLI.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/02/2026
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to download a Windows VHD from Azure, so that I can create new virtual machines or back up the current VM configuration."
---

# Download a Windows VHD from Azure

**Applies to:** :heavy_check_mark: Windows VMs 

This article explains how to download a Windows virtual hard disk (VHD) file from an Azure managed disk. To download a VHD file, the managed disk can't be attached to a running VM, which means the VM experiences downtime. However, some configurations can safely avoid downtime by [snapshotting the managed disk](#alternative-snapshot-the-azure-vm-disk) and downloading the VHD file from the snapshot.

If you're using [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to control resource access, you can use it to restrict downloads of Azure managed disks. Azure validates the identity and permissions of users who download a secured managed disk. For more information, see [Secure downloads and uploads of Azure managed disks](../disks-secure-upload-download.md).

## Optional: Generalize the Windows VM

If you want to use the VHD file as an [image](tutorial-custom-images.md) to create other VMs, use [Sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) to generalize the operating system. Otherwise, you need to make a copy of the managed disk for each VM you want to create.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. [Connect to the VM](connect-logon.md). 
1. On the VM, open the Command Prompt window as an administrator.
1. Change the directory to *%windir%\system32\sysprep* and run sysprep.exe.
1. In the System Preparation Tool dialog box, select **Enter System Out-of-Box Experience (OOBE)**, and make sure that **Generalize** is selected.
1. In Shutdown Options, select **Shutdown**, and then select **OK**. 

Verify that the VM shuts down after Sysprep finishes.

If you don't want to generalize your current VM, you can still create a generalized image by first [making a snapshot of the OS disk](#alternative-snapshot-the-azure-vm-disk), creating a new VM from the snapshot, and then generalizing the copy.

## Stop the Azure VM before downloading its disk

You can't download a VHD file from an Azure managed disk if the disk is attached to a running VM. If you want to keep the VM running, you can [create a snapshot of the managed disk and then download the snapshot](#alternative-snapshot-the-azure-vm-disk).

1. In the Azure portal, search for and select **Virtual machines**.
1. Select the VM from the list.
1. On the blade for the VM, select **Stop**.
1. Wait until **Status** changes to **Stopped (deallocated)** before continuing.

### Alternative: Snapshot the Azure VM disk

> [!NOTE]
> If feasible, stop a VM before taking a snapshot of it, otherwise the snapshot isn't clean. Snapshots of running VMs are in the same state as if their VMs were power cycled or crashed when you take a snapshot. Usually this state is safe but, it could cause problems if the running applications aren't crash resistant.
>  
> Generally, you should only use snapshots of running VMs if the only disk associated with them is a single OS disk. If a VM has one or more data disks, stop the VM before creating a snapshot of the OS or data disks.

Take a snapshot of the Azure managed disk that you want to download as a VHD file.

1. Select the VM in the [portal](https://portal.azure.com).
1. Select **Disks** in the left menu, and then select the OS disk or data disk that you want to snapshot. The managed disk details appear.
1. Select **Create Snapshot** from the menu at the top of the page. The **Create snapshot** page opens.
1. In **Name**, type a name for the snapshot. 
1. For **Snapshot type**, select **Full** or **Incremental**.
1. When you're done, select **Review + create**.
1. After validation passes, select **Create**.
1. When the deployment finishes, open the snapshot and verify that **Provisioning state** is **Succeeded**.

You can now use the snapshot to download a VHD file or create another VM.

## Generate a SAS URL for the managed disk

To download the VHD file from an Azure managed disk, generate a [shared access signature (SAS)](/azure/storage/common/storage-sas-overview?toc=/azure/virtual-machines/windows/toc.json) URL. When you generate the SAS URL, you assign an expiration time to it.

[!INCLUDE [disks-sas-change](../includes/disks-sas-change.md)]

# [Portal](#tab/azure-portal)

1. On the page for the VM, select **Disks** in the left menu.
1. Select the OS disk or data disk that you want to download.
1. On the page for the disk, select **Disk Export** from the left menu.
1. The default expiration time of the SAS URL is **3600** seconds (one hour). You might need to increase this value for Windows OS disks or large data disks. In these situations, **36000** seconds (10 hours) is usually sufficient.
1. Select **Generate URL**.
1. Verify that the SAS URL and **Download the VHD file** option appear.

# [PowerShell](#tab/azure-powershell)

Replace `<resource-group-name>` and `<disk-name>` with your values. Then, use the [Grant-AzDiskAccess](/powershell/module/az.compute/grant-azdiskaccess) cmdlet to generate a read-only SAS URL for the managed disk:

```azurepowershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName "<resource-group-name>" -DiskName "<disk-name>" -DurationInSecond 86400 -Access 'Read'
```

Verify that `$diskSas.AccessSAS` contains a SAS URL before continuing.

# [Azure CLI](#tab/azure-cli)

Replace `<resource-group-name>` and `<disk-name>` with your values. Then, use the [az disk grant-access](/cli/azure/disk#az-disk-grant-access) command to generate a read-only SAS URL for the managed disk and store the returned `accessSas` value in `$sasUrl`:

```azurecli
sasUrl=$(az disk grant-access --duration-in-seconds 86400 --access-level Read --name "<disk-name>" --resource-group "<resource-group-name>" --query accessSas --output tsv)
```

Verify that `$sasUrl` contains a SAS URL before continuing.

---


> [!NOTE]
> When downloading a Windows OS disk, you might need a longer expiration time to download a large VHD file. Large VHDs can take up to several hours to download depending on your connection and the size of the VM.
>
> While the SAS URL is active, attempting to start the VM results in the error **There is an active shared access signature outstanding for disk** *diskname*. You can revoke the SAS URL by selecting **Cancel export** on the **Disk Export** page.  

## Download the Windows VHD file

> [!NOTE]
> If you're using Microsoft Entra ID to secure managed disk downloads, the user downloading the VHD needs the appropriate [RBAC permissions](../disks-secure-upload-download.md#assign-rbac-role).

# [Portal](#tab/azure-portal)

1. On the **Disk Export** page, under the SAS URL that you [generated for the managed disk](#generate-a-sas-url-for-the-managed-disk), select **Download the VHD file**.
1. You might need to select **Save** in your browser to start the download. The default name for the VHD file is *abcd*.

# [PowerShell](#tab/azure-powershell)

The following script uses [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) to sign in and the [Get-AzStorageBlobContent](/powershell/module/az.storage/get-azstorageblobcontent) cmdlet to download the VHD file. The `$diskSas.AccessSAS` property contains the SAS URL that you [generated with `Grant-AzDiskAccess`](#generate-a-sas-url-for-the-managed-disk). Replace `<download-folder-path>` with your local destination folder:

```azurepowershell
Connect-AzAccount
# Set the local destination folder.
$downloadFolder = "<download-folder-path>"
$download = Get-AzStorageBlobContent -Uri $diskSas.AccessSAS -Destination $downloadFolder -Force
```

When the download finishes, use the [Revoke-AzDiskAccess](/powershell/module/az.compute/revoke-azdiskaccess) cmdlet to revoke access to the managed disk: `Revoke-AzDiskAccess -ResourceGroupName "<resource-group-name>" -DiskName "<disk-name>"`.

# [Azure CLI](#tab/azure-cli)

The `$sasUrl` variable contains the `accessSas` value that you [generated with `az disk grant-access`](#generate-a-sas-url-for-the-managed-disk). Replace `<download-file-path>` with your local destination file path. The following script uses the [az storage blob download](/cli/azure/storage/blob#az-storage-blob-download) command to download your VHD file:

> [!NOTE]
> If you're using Microsoft Entra ID to [secure your managed disk](../disks-secure-upload-download.md) uploads and downloads, add `--auth-mode login` to `az storage blob download`.

```azurecli

# Set the local destination file.
downloadFile="<download-file-path>"
# If you're using Microsoft Entra ID to secure your managed disk uploads and downloads, add --auth-mode login to the following command.
az storage blob download --file "$downloadFile" --blob-url "$sasUrl"
```

When the download finishes, use the [az disk revoke-access](/cli/azure/disk#az-disk-revoke-access) command to revoke access to the managed disk: `az disk revoke-access --name "<disk-name>" --resource-group "<resource-group-name>"`.

---

## Next steps

- Learn how to [upload a VHD file to Azure](upload-generalized-managed.md). 
- [Create managed disks from unmanaged disks in a storage account](attach-disk-ps.md).
- [Manage Azure disks with PowerShell](tutorial-manage-data-disk.md).
