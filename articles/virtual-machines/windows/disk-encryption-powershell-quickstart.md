---
title: Create and encrypt a Windows VM by using Azure PowerShell
description: In this quickstart, you learn how to use Azure PowerShell to create and encrypt a Windows virtual machine.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: windows
ms.topic: quickstart
ms.date: 07/14/2026
ai-usage: ai-assisted
ms.custom: devx-track-azurepowershell, mode-api
# Customer intent: As a cloud administrator, I want to create and encrypt a Windows virtual machine by using PowerShell, so that I can ensure the security of sensitive data stored on the VM.
---

# Quickstart: Create and encrypt a Windows virtual machine in Azure by using PowerShell

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

**Applies to:** :heavy_check_mark: Windows VMs

Use the Azure PowerShell module to create and manage Azure resources from the PowerShell command line or in scripts. This quickstart shows how to use the Azure PowerShell module to create a Windows virtual machine (VM), create a Key Vault for the storage of encryption keys, and encrypt the VM.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Create a resource group

Create an Azure resource group with [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). A resource group is a logical container into which Azure resources are deployed and managed:

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## Create a virtual machine

Create an Azure virtual machine with [New-AzVM](/powershell/module/az.compute/new-azvm). Supply credentials to the cmdlet.

```powershell
$cred = Get-Credential

New-AzVM -Name MyVm -Credential $cred -ResourceGroupName MyResourceGroup -Image win2016datacenter -Size Standard_D2S_V3
```

It takes a few minutes to deploy your VM.

## Create a key vault configured for encryption keys

Azure Disk Encryption stores its encryption key in an Azure Key Vault. Create a key vault with [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). To enable the key vault to store encryption keys, use the `-EnabledForDiskEncryption` parameter.

> [!IMPORTANT]
> Each key vault must have a unique name. The following example creates a key vault named *myKV*, but you must name yours something different.

```powershell
New-AzKeyVault -name MyKV -ResourceGroupName myResourceGroup -Location EastUS -EnabledForDiskEncryption
```

## Encrypt the virtual machine

Encrypt your VM by using [Set-AzVmDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension).  

`Set-AzVmDiskEncryptionExtension` needs some values from your key vault object. Get these values by passing the unique name of your key vault to [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault).  

```powershell
$KeyVault = Get-AzKeyVault -VaultName MyKV -ResourceGroupName MyResourceGroup

Set-AzVMDiskEncryptionExtension -ResourceGroupName MyResourceGroup -VMName MyVM -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId
```

After a few minutes, the process returns the following output:

```output
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```

You can verify the encryption process by running [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/Get-AzVMDiskEncryptionStatus).

```powershell
Get-AzVmDiskEncryptionStatus -VMName MyVM -ResourceGroupName MyResourceGroup
```

When encryption is enabled, you see the following fields in the returned output:

```output
OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NoDiskFound
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded
```

## Clean up resources

When you no longer need these resources, use the [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) cmdlet to remove the resource group, VM, and all related resources:

```powershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## Next steps

In this quickstart, you created a virtual machine, created a key vault that was enabled for encryption keys, and encrypted the VM. To learn more about Azure Disk Encryption prerequisites for IaaS VMs, see the next article.

> [!div class="nextstepaction"]
> [Azure Disk Encryption overview](disk-encryption-overview.md)
