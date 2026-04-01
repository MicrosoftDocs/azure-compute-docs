---
title: FAQ Ephemeral OS disks
description: Frequently asked questions on ephemeral OS disks for Azure VMs.
author: Aarthi-Vijayaraghavan
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 05/26/2022
ms.author: aarthiv
ms.subservice: disks
# Customer intent: As a cloud architect, I want to understand the limitations and requirements of using ephemeral OS disks for Azure VMs, so that I can effectively plan and deploy virtual machines with optimal storage configurability and resource management.
---

# Frequently asked questions about Ephemeral OS disks

**Q: What is the size of the Ephemeral OS Disks?**

A: We support platform, Shared Image Gallery, and custom images. Size of Ephemeral OS disk depends on the local storage (cache disk, temp disk, or NVMe disk) available on the Virtual Machine (VM). Max size of Ephemeral OS disk can be 2 TiB. All writes to the OS disk are local on the same node as the Virtual Machine. 

**Q: Can the ephemeral OS disk be resized?**

A: No, once the ephemeral OS disk is provisioned, the OS disk can't be resized. 

**Q: Can the ephemeral OS disk placement be modified after creation of VM?**

A: No, once the ephemeral OS disk is provisioned, the OS disk placement can't be changed. But the VM can be recreated via ARM template deployment/PowerShell/CLI by updating the OS disk placement of choosing. This results in the recreation of the VM with Data on the OS disk deleted and OS is reprovisioned.

**Q: Is there any Temp disk created if image size equals to Temp disk size of VM size selected?**

A: No, in that case, Temp disk is not created.

**Q: How is NVMe disk placement different from temp disk placement?**

A: Nvme disk placement utilizes the entire Nvme disk, while temp disk placement utilizes space equal to the OS disk size. For example, if you're using Standard Ubuntu image on Standard_D2ads_v5 with temp disk placement, you get an OS disk size of 30 GiB & temp disk size of 45 GiB. If you're using the same Standard Ubuntu image on Standard_D2ads_v6 with NVMe disk placement, then you get and OS disk size of 30 GiB with no NVMe disk.

**Q: How does NVMe disk placement work for VMs with multiple NVMe disks?**

A: NVMe Disk Placement utilizes the minimum number of disks required for the OS disk. For example, Standard_D16ads_v6 has two disks of 440 GiB each. If you create and Ephemeral OS disk of size less than 440 GiB, then only one disk is used for creating the ephemeral OS disk. The second NVMe disk is returned to you as a raw local disk. 

**Q: Are Ephemeral OS disks supported on low-priority VMs and Spot VMs?**

A: Yes. There's no option of Stop-Deallocate for Ephemeral VMs, rather users need to Delete instead of deallocating them.

**Q: Can I attach a Managed Disks to an Ephemeral VM?**

A: Yes, you can attach a managed data disk to a VM that uses an ephemeral OS disk. 

**Q: Will all VM sizes be supported for ephemeral OS disks?**

A: No, most Premium Storage VM sizes are supported (DS, ES, FS, GS, M, etc.). To know whether a particular VM size supports ephemeral OS disks for an OS image size, you can use the following script. It takes the OS image size and location as inputs and provides a list of VMs and corresponding placement supported. If cache, temp, and NVMe disk placement are marked as not supported then Ephemeral OS disk can't be used for the given OS image size.

```azurepowershell-interactive
[CmdletBinding()]
param([Parameter(Mandatory=$true)]
      [ValidateNotNullOrEmpty()]
      [string]$Location,
      [Parameter(Mandatory=$true)]
      [long]$OSImageSizeInGB
      )
 
Function HasSupportEphemeralOSDisk([object[]] $capability)
{
    return $capability | where { $_.Name -eq "EphemeralOSDiskSupported" -and $_.Value -eq "True"}
}
 
Function Get-MaxTempDiskAndCacheSize([object[]] $capabilities)
{
    $MaxResourceVolumeGB = 0;
    $CachedDiskGB = 0;
    $NvmeDiskGB = 0;
 
    foreach($capability in $capabilities)
    {
        if ($capability.Name -eq "MaxResourceVolumeMB")
        { $MaxResourceVolumeGB = [int]($capability.Value / 1024) }
 
        if ($capability.Name -eq "CachedDiskBytes")
        { $CachedDiskGB = [int]($capability.Value / (1024 * 1024 * 1024)) }

         if ($capability.Name -eq "NvmeDiskSizeInMiB")
        { $NvmeDiskGB = [int]($capability.Value / (1024)) }

        if ($capability.Name -eq "SupportedEphemeralOSDiskPlacements")
        { $NvmeSupported = [bool]($capability.Value -contains "NvmeDisk") }
    
    }
    
    if (!$NvmeSupported)
    { $NvmeDiskGB = 0; }
    return ($MaxResourceVolumeGB, $CachedDiskGB, $NvmeDiskGB)
}
 
Function Get-EphemeralSupportedVMSku
{
    [CmdletBinding()]
    Param
    (
        [Parameter(Mandatory=$true)]
        [long]$OSImageSizeInGB,
        [Parameter(Mandatory=$true)]
        [string]$Location
    )
 
    $VmSkus = Get-AzComputeResourceSku $Location | Where-Object { $_.ResourceType -eq "virtualMachines" -and (HasSupportEphemeralOSDisk $_.Capabilities) -ne $null }
 
    $Response = @()
    foreach ($sku in $VmSkus)
    {
        ($MaxResourceVolumeGB, $CachedDiskGB, $NvmeDiskGB) = Get-MaxTempDiskAndCacheSize $sku.Capabilities
 
        $Response += New-Object PSObject -Property @{
            ResourceSKU = $sku.Size
            NvmeDiskPlacement = @{ $true = "NOT SUPPORTED"; $false = "SUPPORTED"}[$NvmeDiskGB -lt $OSImageSizeInGB]
            TempDiskPlacement = @{ $true = "NOT SUPPORTED"; $false = "SUPPORTED"}[$MaxResourceVolumeGB -lt $OSImageSizeInGB]
            CacheDiskPlacement = @{ $true = "NOT SUPPORTED"; $false = "SUPPORTED"}[$CachedDiskGB -lt $OSImageSizeInGB]
         };
    }
 
    return $Response
}
 
Get-EphemeralSupportedVMSku -OSImageSizeInGB $OSImageSizeInGB -Location $Location | Format-Table
```
 
**Q: Can the ephemeral OS disk be applied to existing VMs and scale sets?**

A: No, ephemeral OS disk can only be used during VM and scale set creation. 

**Q: Can you mix ephemeral and normal OS disks in a scale set?**

A: No, you can't have a mix of ephemeral and persistent OS disk instances within the same scale set. 

**Q: Can the ephemeral OS disk be created using PowerShell or CLI?**

A: Yes, you can create VMs with Ephemeral OS Disk using REST, Templates, PowerShell, and CLI.
