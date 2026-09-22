---
title: Expand Virtual Hard Disks Attached to a Windows VM in Azure
description: Learn how to expand Azure managed OS and data disks by using the Azure portal or Azure PowerShell, and then expand the volume in Windows.
author: roygara
ms.service: azure-disk-storage
ms.collection: windows
ms.topic: how-to
ms.date: 09/21/2026
ms.author: rogarana
ms.custom: devx-track-azurepowershell, references_regions
# Customer intent: As a cloud administrator, I want to expand the virtual hard disks attached to a Windows VM using PowerShell, so that I can increase storage capacity for applications and data without significant downtime.
---
# Expand virtual hard disks attached to a Windows virtual machine

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

When you create a new virtual machine (VM) in a resource group by deploying an image from [Azure Marketplace](https://azure.microsoft.com/marketplace/), the default operating system (OS) disk is usually 127 GiB. (Some images have smaller OS disk sizes by default.) You can add data disks to your VM. The number of data disks you can attach depends on the VM size that you selected.

You should install applications and CPU-intensive workloads on data disks. You might need to expand the OS disk if you're supporting a legacy application that installs components on the OS disk or if you're migrating a physical PC or VM from on-premises that has a larger OS disk. To complete an expansion, first increase the Azure managed OS disk or data disk size by using the Azure portal or Azure PowerShell, and then expand the volume in Windows to use the additional space.

An OS disk has a maximum capacity of 4,095 GiB. However, many operating systems are partitioned with [master boot record (MBR)](https://wikipedia.org/wiki/Master_boot_record) by default. MBR limits the usable size to 2 TiB. If you need more than 2 TiB, create and attach data disks and use them for data storage. If you need to store data on the OS disk and require the extra space, [convert it to a GUID Partition Table](/windows-server/storage/disk-management/change-an-mbr-disk-into-a-gpt-disk) (GPT). To learn about the differences between MBR and GPT on Windows deployments, see [Windows and GPT FAQ](/windows-hardware/manufacture/desktop/windows-and-gpt-faq).

Unless you use [Expand without downtime](#expand-without-downtime), expanding a data disk requires the VM to be deallocated. Shrinking an existing disk isn't supported and might result in data loss.

After you expand the disks, [expand the volume in the OS](#expand-the-volume-in-the-operating-system) to take advantage of the larger disk. You can't expand the size of striped volumes.

## Expand without downtime

You can expand data disks without deallocating your VM. The host cache setting of your disk doesn't change whether or not you can expand a data disk without deallocating your VM.

This feature has the following limitations.

[!INCLUDE [virtual-machines-disks-expand-without-downtime-restrictions](../includes/virtual-machines-disks-expand-without-downtime-restrictions.md)]

## Resize a managed disk in the Azure portal

Only eligible data disks can be expanded without deallocating the VM. OS disks and data disks that don't meet the [requirements for expansion without downtime](#expand-without-downtime) require deallocation.

1. In the [Azure portal](https://portal.azure.com/), go to the VM in which you want to expand the disk. If the disk requires deallocation, select **Stop**.
1. On the left menu, under **Settings**, select **Disks**.

    :::image type="content" source="./media/expand-os-disk/select-disks.png" alt-text="Screenshot that shows the Disks option selected in the Settings section of the menu.":::

1. Under **Disk name**, select the disk that you want to expand.

    :::image type="content" source="./media/expand-os-disk/disk-name.png" alt-text="Screenshot that shows the Disks pane with a disk name selected.":::

1. On the left menu, under **Settings**, select **Size + performance**.

    :::image type="content" source="./media/expand-os-disk/configuration.png" alt-text="Screenshot that shows the Size and performance option selected in the Settings section of the menu.":::

1. On **Size + performance**, select the disk size that you want.

    The new size should be greater than the existing disk size. The maximum allowed is 4,095 GiB for OS disks. You can expand the virtual hard disk (VHD) blob beyond that size, but the OS works only with the first 4,095 GiB of space.

    :::image type="content" source="./media/expand-os-disk/size.png" alt-text="Screenshot that shows the Size and performance pane with the disk size selected.":::

1. Select **Resize** at the bottom of the page.

    :::image type="content" source="./media/expand-os-disk/save.png" alt-text="Screenshot that shows the Size and performance pane with the Resize button selected.":::

1. If you deallocated the VM, return to the VM **Overview** pane and select **Start**.

## Resize a managed disk by using PowerShell

Open a PowerShell window in administrative mode and follow these steps. Only eligible data disks can be expanded without deallocating the VM. OS disks and data disks that don't meet the [requirements for expansion without downtime](#expand-without-downtime) require deallocation.

1. Sign in to your Azure account by using [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount), and set the active subscription by using [Set-AzContext](/powershell/module/az.accounts/set-azcontext):

    ```powershell
    Connect-AzAccount
    Set-AzContext -Subscription 'my-subscription-name'
    ```

1. Set your resource group name and VM name:

    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    $diskName = 'my-disk-name'
    ```

1. Obtain a reference to your VM:

    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

1. If the disk requires deallocation, stop the VM before you resize the disk:

    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

1. Obtain a reference to the managed disk. Set the managed disk size to the value that you want and update the disk:

    ```powershell
    $disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $diskName
    $disk.DiskSizeGB = 1023
    Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
    ```

    The new size should be greater than the existing disk size. The maximum allowed is 4,095 GiB for OS disks. You can expand the VHD blob beyond that size, but the OS works only with the first 4,095 GiB of space.

1. Updating the disk might take a few seconds. If you deallocated the VM, restart it after the command finishes running:

    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

Remote into the VM, open **Computer Management** (or **Disk Management**), and expand the drive by using the newly allocated space.

## Expand the volume in the operating system

After you expand the disk for the VM, go into the OS and expand the volume to encompass the new space. There are several methods for expanding a partition. This section covers connecting the VM by using a Remote Desktop Protocol (RDP) connection to expand the partition by using [DiskPart](#use-diskpart) or [Disk Manager](#use-disk-manager).

### Use DiskPart

1. Open an RDP connection to your VM.

1. Open a command prompt and enter `diskpart`.

1. At the `DISKPART` prompt, enter `list volume`. Make a note of the volume that you want to extend.

1. At the `DISKPART` prompt, enter `select volume <volumenumber>`. This action selects the volume `volumenumber` that you want to extend into contiguous, empty space on the same disk.

1. At the `DISKPART` prompt, enter `extend [size=<size>]`. This action extends the selected volume by size in megabytes (MBs).

### Use Disk Manager

1. Start a remote desktop session with the VM.
1. Open **Disk Management**.

    :::image type="content" source="media/expand-os-disk/disk-mgr-1.png" alt-text="Screenshot that shows Disk Management.":::

1. Right-click an existing **C:** drive partition and select **Extend Volume**.

    :::image type="content" source="media/expand-os-disk/disk-mgr-2.png" alt-text="Screenshot that shows how to extend the volume.":::

1. Follow the steps in the wizard to see the disk with updated capacity.

    :::image type="content" source="media/expand-os-disk/disk-mgr-3.png" alt-text="Screenshot that shows the larger C: volume in Disk Manager.":::

## Expand without downtime classic VM SKU support


If you're using a classic VM SKU, it might not support expanding disks without downtime.

Use the following PowerShell script to determine which classic VM SKUs support expansion without downtime:

```azurepowershell
Connect-AzAccount
$subscriptionId="yourSubID"
$location="desiredRegion"
Set-AzContext -Subscription $subscriptionId
$vmSizes=Get-AzComputeResourceSku -Location $location | where{$_.ResourceType -eq 'virtualMachines'}

foreach($vmSize in $vmSizes){
    foreach($capability in $vmSize.Capabilities)
    {
       if(($capability.Name -eq "EphemeralOSDiskSupported" -and $capability.Value -eq "True") -or ($capability.Name -eq "PremiumIO" -and $capability.Value -eq "True") -or ($capability.Name -eq "HyperVGenerations" -and $capability.Value -match "V2"))
        {
            $vmSize.Name
       }
   }
}
```

## Related content

You can also attach disks by using the [Azure portal](/azure/virtual-machines/windows/attach-managed-disk-portal).

