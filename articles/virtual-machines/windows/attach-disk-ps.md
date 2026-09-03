---
title: Attach a data disk to a Windows VM in Azure by using PowerShell
description: How to attach a new or existing data disk to a Windows VM using PowerShell with the Resource Manager deployment model.
author: roygara
ms.service: azure-disk-storage
ms.collection: windows
ms.topic: how-to
ms.date: 09/02/2026
ms.author: rogarana
ms.custom: devx-track-azurepowershell
ai-usage: ai-assisted

# Customer intent: As a cloud administrator, I want to attach new and existing data disks to Windows VMs using PowerShell, so that I can efficiently manage storage solutions for my workloads and optimize performance within the Azure environment.
---
# Attach a data disk to a Windows VM with PowerShell

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

This article shows you how to attach both new and existing disks to a Windows virtual machine by using PowerShell. 

First, review these tips:

* The size of the virtual machine controls how many data disks you can attach. For more information, see [Sizes for virtual machines](../sizes.md).
* To use Premium SSDs, you'll need a [premium storage-enabled VM type](../sizes-memory.md), like the DS-series or GS-series virtual machine.

This article uses PowerShell within the [Azure Cloud Shell](/azure/cloud-shell/overview), which is constantly updated to the latest version. To open the Cloud Shell, select **Try it** from the top of any code block.

## Reduce data disk attach latency

In select regions, the disk attach latency is reduced, so you see an improvement of up to 15%. This improvement is useful if you have planned or unplanned failovers between VMs, you're scaling your workload, or you're running a high-scale stateful workload such as Azure Kubernetes Service. However, this improvement is limited to the explicit disk attach command, [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk). You don't see the performance improvement if you call a command that might implicitly perform an attach, like [Update-AzVM](/powershell/module/az.compute/update-azvm). You don't need to take any action other than calling the explicit attach command to see this improvement.

[!INCLUDE [virtual-machines-disks-fast-attach-detach-regions](../includes/virtual-machines-disks-fast-attach-detach-regions.md)]

## Add an empty data disk to a virtual machine

This example shows how to add an empty data disk to an existing virtual machine.

### Create and attach a managed disk

Set values for the resource group, VM, location, storage type, and data disk name. The [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) cmdlet creates a configuration for an empty 128-GiB managed disk, and [New-AzDisk](/powershell/module/az.compute/new-azdisk) creates the disk. Then, [Get-AzVM](/powershell/module/az.compute/get-azvm) gets the VM configuration, `Add-AzVMDataDisk` adds the disk to it, and `Update-AzVM` applies the updated configuration to the VM.

```azurepowershell-interactive
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US'
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### Create and attach a managed disk in an availability zone

Set values for the resource group, VM, location, storage type, and data disk name. To create a managed disk in an availability zone, use `New-AzDiskConfig` with the `-Zone` parameter. The following example creates an empty 128-GiB managed disk in zone 1, adds it to the VM configuration, and applies the updated configuration to the VM.

```powershell
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US 2'
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128 -Zone 1
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### Initialize the disk

After you add an empty disk, you need to initialize it. To initialize the disk, sign in to a VM and use disk management. If you enable [WinRM](/windows/desktop/winrm/portal) and a certificate on the VM when you create it, you can use remote PowerShell to initialize the disk.

You can also use the [Set-AzVMCustomScriptExtension](/powershell/module/az.compute/set-azvmcustomscriptextension) cmdlet to add a Custom Script Extension to the VM. The following example uses the resource group and VM names from the attachment procedure, along with values for the VM location, extension name, initialization script file, storage account name, storage account key, and container name. Store the script in the specified Azure Storage container before you run the command:

```azurepowershell-interactive
    $location = "location-name"
    $scriptName = "script-name"
    $fileName = "script-file-name"
    Set-AzVMCustomScriptExtension -ResourceGroupName $rgName -Location $location -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"
```

The script file can contain code to initialize the disks, for example:

> [!NOTE]
> The example script uses MBR partition style. If your disk is two tebibytes (TiB) or larger, you must use GPT partitioning. If it's under two TiB, you can use either MBR or GPT.

```azurepowershell-interactive
    $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

    $letters = 70..89 | ForEach-Object { [char]$_ }
    $count = 0
    $labels = "data1","data2"

    foreach ($disk in $disks) {
        $driveLetter = $letters[$count].ToString()
        $disk |
        Initialize-Disk -PartitionStyle MBR -PassThru |
        New-Partition -UseMaximumSize -DriveLetter $driveLetter |
        Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
	$count++
    }
```

## Attach an existing data disk to a VM

Set values for the resource group, VM, and existing managed disk name. Use [Get-AzDisk](/powershell/module/az.compute/get-azdisk) to get the managed disk, and `Get-AzVM` to get the VM configuration. Then, use `Add-AzVMDataDisk` to add the existing disk to the VM configuration, and `Update-AzVM` to apply the updated configuration to the VM.

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$dataDiskName = "myDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## Next steps

You can also deploy managed disks using templates. For more information, see [Using managed disks in Azure Resource Manager Templates](../using-managed-disks-template-deployments.md) or the [quickstart template](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-multiple-data-disk) for deploying multiple data disks.
