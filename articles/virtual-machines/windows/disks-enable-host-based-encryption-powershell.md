---
title: Enable end-to-end encryption using encryption at host with Azure PowerShell
description: How to enable end-to-end encryption for your Azure VMs using encryption at host.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/21/2026
ms.author: rogarana
ms.custom:
  - devx-track-azurepowershell
  - ignite-2023
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to enable end-to-end encryption for my Virtual Machines, so that I can ensure data security and compliance while managing sensitive information.
---

# Enable end-to-end encryption by using encryption at host with Azure PowerShell

**Applies to:** :heavy_check_mark: Windows VMs 

When you enable encryption at host, data stored on the VM host is encrypted at rest and flows encrypted to the Storage service. For conceptual information on encryption at host, and other managed disk encryption types, see [Encryption at host - End-to-end encryption for your VM data](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## Restrictions

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### Supported VM sizes

The complete list of supported VM sizes can be pulled programmatically. To learn how to retrieve them programmatically, refer to the [Finding supported VM sizes](#finding-supported-vm-sizes) section.
Upgrading the VM size triggers validation that checks whether the new VM size supports the `EncryptionAtHost` feature.

## Prerequisites

You must enable the feature for your subscription before you use the `EncryptionAtHost` property for your VM or virtual machine scale set. Use the following steps to enable the feature for your subscription:

1.	Execute the following command to register the feature for your subscription

    ```powershell
     Register-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute" 
    ```

2.	Check that the registration state is Registered (takes a few minutes) using the following command before trying out the feature.

    ```powershell
     Get-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace "Microsoft.Compute"  
    ```


### Create an Azure Key Vault and disk encryption set

> [!NOTE]
> This section only applies to configurations with customer-managed keys. If you're using platform-managed keys, you can skip to [Create and update VMs and virtual machine scale sets with encryption at host](#create-and-update-vms-and-virtual-machine-scale-sets-with-encryption-at-host).

When you enable the feature, set up an Azure Key Vault and a disk encryption set, if you don't already have them.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]

## Configure encryption at host for VMs and virtual machine scale sets

You can enable encryption at host by setting the `EncryptionAtHost` property under `securityProfile` for VMs or virtual machine scale sets by using API version **2020-06-01** or later.

`"securityProfile": { "encryptionAtHost": "true" }`

## Create and update VMs and virtual machine scale sets with encryption at host

### Create a VM with encryption at host enabled with customer-managed keys

Before you run the script, provide an existing resource group and disk encryption set, along with values for the VM administrator credentials, region, VM size, and virtual network. The script creates the virtual network and network interface, uses `New-AzVMConfig` with `-EncryptionAtHost`, and deploys the Windows VM with `New-AzVM`. The disk encryption set encrypts the cache of the OS and data disks with customer-managed keys. The temporary disk is encrypted with platform-managed keys.

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

# Enable encryption at host by specifying EncryptionAtHost parameter

$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize -EncryptionAtHost
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption with a customer managed key for OS disk by setting DiskEncryptionSetId property 

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage

# Add a data disk encrypted with a customer managed key by setting DiskEncryptionSetId property 

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0 -DiskEncryptionSetId $diskEncryptionSet.Id 
    
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

### Create a VM with encryption at host enabled with platform-managed keys

Before you run the script, provide an existing resource group and values for the VM administrator credentials, region, VM size, and virtual network. The script creates the virtual network and network interface, uses `New-AzVMConfig` with `-EncryptionAtHost`, and deploys the Windows VM with `New-AzVM`. Encryption at host encrypts the cache of the OS and data disks, and the temporary disk, with platform-managed keys.

```powershell
$VMLocalAdminUser = "yourVMLocalAdminUserName"
$VMLocalAdminSecurePassword = ConvertTo-SecureString <password> -AsPlainText -Force
$LocationName = "yourRegion"
$ResourceGroupName = "yourResourceGroupName"
$ComputerName = "yourComputerName"
$VMName = "yourVMName"
$VMSize = "yourVMSize"
    
$NetworkName = "yourNetworkName"
$NICName = "yourNICName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0].Id
    
$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

# Enable encryption at host by specifying EncryptionAtHost parameter

$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize -EncryptionAtHost

$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -CreateOption FromImage

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0
    
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine
```

### Update a VM to enable encryption at host

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMName = "yourVMName"

$VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName

Stop-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName -Force

Update-AzVM -VM $VM -ResourceGroupName $ResourceGroupName -EncryptionAtHost $true
```

### Check the status of encryption at host for a VM

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMName = "yourVMName"

$VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName

$VM.SecurityProfile.EncryptionAtHost
```

### Disable encryption at host

You must deallocate your VM before you can disable encryption at host.

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMName = "yourVMName"

$VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName

Stop-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName -Force

Update-AzVM -VM $VM -ResourceGroupName $ResourceGroupName -EncryptionAtHost $false
```

### Create a virtual machine scale set with encryption at host enabled with customer-managed keys

> [!IMPORTANT]
>Starting November 2023, virtual machine scale sets created using PowerShell and Azure CLI default to Flexible Orchestration Mode if you don't specify an orchestration mode. For more information about this change and what actions you should take, see [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](
https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

Before you run the script, provide an existing resource group and disk encryption set, along with values for the administrator credentials, region, VM size, and virtual network. The script creates the virtual network, configures a two-instance Flexible virtual machine scale set with `New-AzVmssConfig` and `-EncryptionAtHost`, and deploys it with `New-AzVmss`. The disk encryption set encrypts the cache of the OS and data disks with customer-managed keys. The temporary disks are encrypted with platform-managed keys.

```powershell
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

# Enable encryption at host by specifying EncryptionAtHost parameter

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -OrchestrationMode "Flexible" -EncryptionAtHost 

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption with a customer managed key for the OS disk by setting DiskEncryptionSetId property 

$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -DiskEncryptionSetId $diskEncryptionSet.Id -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

# Add a data disk encrypted with a customer managed key by setting DiskEncryptionSetId property 

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS -DiskEncryptionSetId $diskEncryptionSet.Id

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

### Create a virtual machine scale set with encryption at host enabled with platform-managed keys

> [!IMPORTANT]
>Starting November 2023, virtual machine scale sets created using PowerShell and Azure CLI default to Flexible Orchestration Mode if you don't specify an orchestration mode. For more information about this change and what actions you should take, see [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](
https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

Before you run the script, provide an existing resource group and values for the administrator credentials, region, VM size, and virtual network. The script creates the virtual network, configures a two-instance Flexible virtual machine scale set with `New-AzVmssConfig` and `-EncryptionAtHost`, and deploys it with `New-AzVmss`. Encryption at host encrypts the cache of the OS and data disks, and the temporary disks, with platform-managed keys.

```powershell
$VMLocalAdminUser = "yourLocalAdminUser"
$VMLocalAdminSecurePassword = ConvertTo-SecureString Password@123 -AsPlainText -Force
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$ComputerNamePrefix = "yourComputerNamePrefix"
$VMScaleSetName = "yourVMSSName"
$VMSize = "Standard_DS3_v2"
    
$NetworkName = "yourVNETName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix

$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" -SubnetId $Vnet.Subnets[0].Id 

# Enable encryption at host by specifying EncryptionAtHost parameter

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -OrchestrationMode "Flexible" -EncryptionAtHost

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig
 
$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS 

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

### Update a virtual machine scale set to enable encryption at host

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMScaleSetName = "yourVMSSName"

$VMSS = Get-AzVmss -ResourceGroupName $ResourceGroupName -Name $VMScaleSetName

Update-AzVmss -VirtualMachineScaleSet $VMSS -Name $VMScaleSetName -ResourceGroupName $ResourceGroupName -EncryptionAtHost $true
```

### Check the status of encryption at host for a virtual machine scale set

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMScaleSetName = "yourVMSSName"

$VMSS = Get-AzVmss -ResourceGroupName $ResourceGroupName -Name $VMScaleSetName

$VMSS.VirtualMachineProfile.SecurityProfile.EncryptionAtHost
```

### Update a virtual machine scale set to disable encryption at host

You can disable encryption at host on your virtual machine scale set, but this change affects only VMs created after you disable encryption at host. For existing VMs, you must deallocate the VM, [disable encryption at host on that individual VM](#disable-encryption-at-host), then reallocate the VM.

```powershell
$ResourceGroupName = "yourResourceGroupName"
$VMScaleSetName = "yourVMSSName"

$VMSS = Get-AzVmss -ResourceGroupName $ResourceGroupName -Name $VMScaleSetName

Update-AzVmss -VirtualMachineScaleSet $VMSS -Name $VMScaleSetName -ResourceGroupName $ResourceGroupName -EncryptionAtHost $false
```

## Finding supported VM sizes

Legacy VM Sizes aren't supported. You can find the list of supported VM sizes by either:

Calling the [Resource Skus API](/rest/api/compute/resourceskus/list) and checking that the `EncryptionAtHostSupported` capability is set to **True**.

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "CentralUS"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

Or, calling the [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku) PowerShell cmdlet.

```powershell
Get-AzComputeResourceSku -Location "centralus" |
    Where-Object { $_.ResourceType -eq 'virtualMachines' -and $_.capabilities.where({ $_.Name -eq 'EncryptionAtHostSupported' }, 'First').Value -eq 'True' } |
    Select-Object -ExpandProperty Name
```

## Next steps

Now that you've created and configured these resources, you can use them to secure your managed disks. The following link contains example scripts, each with a respective scenario, that you can use to secure your managed disks.

[Azure Resource Manager template samples](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
