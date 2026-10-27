---
title: Enable customer-managed keys for Azure managed disks with PowerShell
description: Enable server-side encryption using customer-managed keys on your managed disks with Azure PowerShell.
author: roygara
ms.date: 09/21/2026
ms.topic: how-to
ms.author: rogarana
ms.service: azure-disk-storage
ms.custom: devx-track-azurepowershell
ai-usage: ai-assisted
# Customer intent: As an IT admin, I want to enable server-side encryption with customer-managed keys for managed disks using PowerShell, so that I can ensure data security and compliance with my organization's encryption policies.
---

# Enable customer-managed keys for Azure managed disks with PowerShell

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure Disk Storage supports server-side encryption (SSE) with customer-managed keys for managed disks. For conceptual information about customer-managed keys and other managed disk encryption types, see [Customer-managed keys](../disk-encryption.md#customer-managed-keys).

## Restrictions

For now, customer-managed keys have the following restrictions:

[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

## Set up Azure Key Vault and a disk encryption set with automatic key rotation

To use customer-managed keys with server-side encryption, set up an Azure Key Vault and a disk encryption set.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]


## Manage customer-managed keys for Azure managed disks with PowerShell

After you create and configure the required resources, use the following example Azure PowerShell scripts to create encrypted VMs and disks, encrypt existing managed disks and scale sets, rotate a disk encryption set key, and check a disk's server-side encryption status.

### Create a VM using a Marketplace image, encrypting the OS and data disks with customer-managed keys

Before you run the script, provide an existing resource group and disk encryption set, along with values for the VM administrator credentials, region, VM size, and virtual network. The script creates the virtual network, network interface, and VM from a Windows Server Marketplace image. It uses [`Get-AzDiskEncryptionSet`](/powershell/module/az.compute/get-azdiskencryptionset) to retrieve the disk encryption set and [`New-AzVM`](/powershell/module/az.compute/new-azvm) to create the VM with an encrypted OS disk and a 128-GiB encrypted data disk. Replace the example values with your own parameters, and then run the script.

```powershell
$VMLocalAdminUser = "yourVMLocalAdminUserName"
$VMLocalAdminSecurePassword = ConvertTo-SecureString <password> -AsPlainText -Force
$LocationName = "yourRegion"
$ResourceGroupName = "yourResourceGroupName"
$ComputerName = "yourComputerName"
$VMName = "yourVMName"
$VMSize = "yourVMSize"
$diskEncryptionSetName="yourdiskEncryptionSetName"
    
$NetworkName = "yourNetworkName"
$NICName = "yourNICName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0].Id
    
$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
    
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0 -DiskEncryptionSetId $diskEncryptionSet.Id 
    
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

### Create an empty disk encrypted using server-side encryption with customer-managed keys and attach it to a VM

The script uses [`Add-AzVMDataDisk`](/powershell/module/az.compute/add-azvmdatadisk) to add the encrypted data disk to the VM configuration and [`Update-AzVM`](/powershell/module/az.compute/update-azvm) to apply the configuration. Replace the example values with your own parameters, and then run the script.

```PowerShell
$vmName = "yourVMName"
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskSKU = "Premium_LRS"
$diskSizeinGiB = 30
$diskLUN = 1
$diskEncryptionSetName="yourDiskEncryptionSetName"


$vm = Get-AzVM -Name $vmName -ResourceGroupName $ResourceGroupName 

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -DiskSizeInGB $diskSizeinGiB -StorageAccountType $diskSKU -Lun $diskLUN -DiskEncryptionSetId $diskEncryptionSet.Id 

Update-AzVM -ResourceGroupName $ResourceGroupName -VM $vm

```

### Encrypt existing managed disks 

Your existing disks must not be attached to a running VM. The script uses [`New-AzDiskUpdateConfig`](/powershell/module/az.compute/new-azdiskupdateconfig) and [`Update-AzDisk`](/powershell/module/az.compute/update-azdisk) to configure customer-managed keys for the disk.

```PowerShell
$rgName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskEncryptionSetName = "yourDiskEncryptionSetName"
 
$diskEncryptionSet = Get-AzDiskEncryptionSet -ResourceGroupName $rgName -Name $diskEncryptionSetName
 
New-AzDiskUpdateConfig -EncryptionType "EncryptionAtRestWithCustomerKey" -DiskEncryptionSetId $diskEncryptionSet.Id | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
```

### Encrypt an existing virtual machine scale set (uniform orchestration mode) by using server-side encryption and customer-managed keys

This script will work for scale sets in uniform orchestration mode only. For scale sets in flexible orchestration mode, follow the Encrypt existing managed disks for each VM.

The script uses [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss) to retrieve the scale set and [`Update-AzVmss`](/powershell/module/az.compute/update-azvmss) to apply the disk encryption set. Replace the example values with your own parameters, and then run the script.

```powershell
#set variables 
$vmssname = "name of the vmss that is already created"
$diskencryptionsetname = "name of the diskencryptionset already created"
$vmssrgname = "vmss resourcegroup name"
$diskencryptionsetrgname = "diskencryptionset resourcegroup name"

#get vmss object and create diskencryptionset object attach to vmss os disk
$ssevmss = get-azvmss -ResourceGroupName $vmssrgname -VMScaleSetName $vmssname
$ssevmss.VirtualMachineProfile.StorageProfile.OsDisk.ManagedDisk.DiskEncryptionSet = New-Object -TypeName Microsoft.Azure.Management.Compute.Models.DiskEncryptionSetParameters

#get diskencryption object and retrieve the resource id
$des = Get-AzDiskEncryptionSet -ResourceGroupName $diskencryptionsetrgname -Name $diskencryptionsetname
write-host "the diskencryptionset resource id is:" $des.Id

#associate DES resource id to os disk and update vmss 
$ssevmss.VirtualMachineProfile.StorageProfile.OsDisk.ManagedDisk.DiskEncryptionSet.id = $des.Id
$ssevmss | update-azvmss
```

### Create a virtual machine scale set using a Marketplace image, encrypting the OS and data disks with customer-managed keys

Before you run the script, provide an existing resource group and disk encryption set, along with values for the administrator credentials, region, VM size, and virtual network. The script creates the virtual network and uses [`New-AzVmss`](/powershell/module/az.compute/new-azvmss) to create a two-instance uniform virtual machine scale set from a Windows Server Marketplace image. The scale set's OS disks and 128-GiB data disks use the disk encryption set. Replace the example values with your own parameters, and then run the script.

> [!IMPORTANT]
>Starting November 2023, VM scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

```PowerShell
$VMLocalAdminUser = "yourLocalAdminUser"
$VMLocalAdminSecurePassword = ConvertTo-SecureString Password@123 -AsPlainText -Force
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$ComputerNamePrefix = "yourComputerNamePrefix"
$VMScaleSetName = "yourVMSSName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourDiskEncryptionSetName"
    
$NetworkName = "yourVNETName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix

$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" -SubnetId $Vnet.Subnets[0].Id 

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -UpgradePolicyMode 'Automatic' -OrchestrationMode 'Uniform'

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption at rest with customer managed keys for OS disk by setting DiskEncryptionSetId property 

$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -DiskEncryptionSetId $diskEncryptionSet.Id -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

# Add a data disk encrypted at rest with customer managed keys by setting DiskEncryptionSetId property 

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS -DiskEncryptionSetId $diskEncryptionSet.Id

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

### Change the key of a disk encryption set to rotate the key for referenced resources

The script uses [`Update-AzDiskEncryptionSet`](/powershell/module/az.compute/update-azdiskencryptionset) to change the key referenced by the disk encryption set. Replace the example values with your own parameters, and then run the script.

```PowerShell
$ResourceGroupName="yourResourceGroupName"
$keyVaultName="yourKeyVaultName"
$keyName="yourKeyName"
$diskEncryptionSetName="yourDiskEncryptionSetName"

$keyVault = Get-AzKeyVault -VaultName $keyVaultName -ResourceGroupName $ResourceGroupName

$keyVaultKey = Get-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName

Update-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName -SourceVaultId $keyVault.ResourceId -KeyUrl $keyVaultKey.Id
```

### Find the status of server-side encryption of a disk

The script uses [`Get-AzDisk`](/powershell/module/az.compute/get-azdisk) to retrieve the disk's server-side encryption type.

[!INCLUDE [virtual-machines-disks-encryption-status-powershell](../includes/virtual-machines-disks-encryption-status-powershell.md)]

For disks configured by the preceding examples, `EncryptionAtRestWithCustomerKey` indicates server-side encryption with a customer-managed key. `EncryptionAtRestWithPlatformAndCustomerKeys` indicates double encryption with both customer-managed and platform-managed keys.

> [!IMPORTANT]
> Customer-managed keys rely on managed identities for Azure resources, a feature of Microsoft Entra ID. When you configure customer-managed keys, a managed identity is automatically assigned to your resources under the covers. If you subsequently move the subscription, resource group, or managed disk from one Microsoft Entra directory to another, the managed identity associated with the managed disks is not transferred to the new tenant, so customer-managed keys may no longer work. For more information, see [Transferring a subscription between Microsoft Entra directories](/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories).

## Next steps

- [Explore the Azure Resource Manager templates for creating encrypted disks with customer-managed keys](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Replicate machines with customer-managed keys enabled disks](/azure/site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks)
- [Set up disaster recovery of VMware VMs to Azure with PowerShell](/azure/site-recovery/vmware-azure-disaster-recovery-powershell#replicate-vmware-vms)
- [Set up disaster recovery to Azure for Hyper-V VMs using PowerShell and Azure Resource Manager](/azure/site-recovery/hyper-v-azure-powershell-resource-manager#step-7-enable-vm-protection)
