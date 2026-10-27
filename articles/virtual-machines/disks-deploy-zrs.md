---
title: Deploy a ZRS managed disk
description: Learn how to deploy a managed disk that uses zone-redundant storage (ZRS).
author: roygara
ms.author: rogarana
ms.date: 09/16/2026
ms.topic: how-to
ms.service: azure-disk-storage
ms.devlang: azurecli
ms.custom:
  - references_regions
  - devx-track-azurepowershell
  - devx-track-azurecli
  - sfi-ropc-nochange
  - portal
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to deploy zone-redundant storage (ZRS) managed disks for my virtual machines, so that I can ensure high availability and resilience for my applications across multiple availability zones."
---

# Deploy a managed disk that uses zone-redundant storage

This article covers how to deploy a disk that uses zone-redundant storage (ZRS) as a redundancy option. ZRS replicates your Azure managed disk synchronously across three Azure availability zones in the selected region. Each availability zone is a separate physical location with independent power, cooling, and networking.

For conceptual information on ZRS, see [Zone-redundant storage for managed disks](disks-redundancy.md#zone-redundant-storage-for-managed-disks)

## Limitations

[!INCLUDE [disk-storage-zrs-limitations](./includes/disk-storage-zrs-limitations.md)]

## Regional availability

[!INCLUDE [disk-storage-zrs-regions](./includes/disk-storage-zrs-regions.md)]

## Deploy ZRS managed disks

# [Azure portal](#tab/portal)

### Create a virtual machine with a ZRS OS disk

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Navigate to **Virtual machines** and follow the normal VM creation process.
1. Proceed to the **Disks** pane.
1. Select your disk and select one of the ZRS disks in the drop-down.

    :::image type="content" source="media/disks-deploy-zrs/disks-zrs-portal-select-blade.png" alt-text="Screenshot of the vm creation workflow, disks pane, OS disk dropdown is expanded with the ZRS Premium SSD and Standard SSD options highlighted." lightbox="media/disks-deploy-zrs/disks-zrs-portal-select-blade.png":::

1. Proceed through the rest of the VM deployment, making any choices that you desire.

You've now deployed a VM with a ZRS OS disk.

### Create a ZRS disk

1. In the Azure portal, search for and select **Disks**.
1. Select **+ Add** to create a new disk.
1. Select a supported region and **Availability zone** to **None**.
1. Select **Change size**.

    :::image type="content" source="media/disks-deploy-zrs/create-zrs-disk-pane.png" alt-text="Screenshot of the disk creation workflow, basics pane." lightbox="media/disks-deploy-zrs/create-zrs-disk-pane.png":::

1. Select one of the available ZRS disks and select **OK**.

    :::image type="content" source="media/disks-deploy-zrs/select-zrs-disk-sku.png" alt-text="Screenshot of the disk creation workflow, select a disk size pane, ZRS disks highlighted." lightbox="media/disks-deploy-zrs/select-zrs-disk-sku.png":::

1. Continue through the deployment process.

You have now created a managed disk that uses ZRS.

# [Azure CLI](#tab/azure-cli)

### Create a virtual machine with ZRS disks

The following Azure CLI example uses [az group create](/cli/azure/group#az-group-create) and [az vm create](/cli/azure/vm#az-vm-create) to create an Ubuntu virtual machine (VM) with a Standard SSD ZRS OS disk and a 128-GiB Premium SSD ZRS data disk. Replace the variable values with the names, region, VM size, and image for your deployment.

```azurecli
rgName=yourRGName
vmName=yourVMName
location=westus2
vmSize=Standard_DS2_v2
image=Ubuntu2204
osDiskSku=StandardSSD_ZRS
dataDiskSku=Premium_ZRS

az group create -n $rgName -l $location

az vm create -g $rgName \
-n $vmName \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--data-disk-sizes-gb 128 \
--storage-sku os=$osDiskSku 0=$dataDiskSku
```

### Create virtual machines with a shared ZRS disk in different zones

The following Azure CLI example uses [az disk create](/cli/azure/disk#az-disk-create) to create a 1-TiB shared Premium SSD ZRS disk that supports two attachments. It then uses [az vm create](/cli/azure/vm#az-vm-create) to create two Ubuntu VMs in availability zones 1 and 2 and attach the shared disk to both VMs. Replace the variable values with the names, region, VM size, and image for your deployment.

```azurecli

location=westus2
rgName=yourRGName
vmNamePrefix=yourVMNamePrefix
vmSize=Standard_DS2_v2
image=Ubuntu2204
osDiskSku=StandardSSD_LRS
sharedDiskName=yourSharedDiskName
sharedDataDiskSku=Premium_ZRS

az group create -n $rgName -l $location

az disk create -g $rgName \
-n $sharedDiskName \
-l $location \
--size-gb 1024 \
--sku $sharedDataDiskSku \
--max-shares 2


sharedDiskId=$(az disk show -g $rgName -n $sharedDiskName --query 'id' -o tsv)

az vm create -g $rgName \
-n $vmNamePrefix"01" \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--zone 1 \
--attach-data-disks $sharedDiskId \
--storage-sku os=$osDiskSku \
--vnet-name $vmNamePrefix"_vnet" \
--subnet $vmNamePrefix"_subnet"

az vm create -g $rgName \
-n $vmNamePrefix"02" \
-l $location \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--zone 2 \
--attach-data-disks $sharedDiskId \
--storage-sku os=$osDiskSku \
--vnet-name $vmNamePrefix"_vnet" \
--subnet $vmNamePrefix"_subnet"

```

### Create a virtual machine scale set with ZRS disks

The following Azure CLI example uses [az vmss create](/cli/azure/vmss#az-vmss-create) to create an Ubuntu virtual machine scale set with Standard SSD ZRS OS disks and 128-GiB Premium SSD ZRS data disks. Replace the variable values with the resource group, region, scale set name, VM size, and image for your deployment.

```azurecli
location=westus2
rgName=yourRGName
vmssName=yourVMSSName
vmSize=Standard_DS3_V2
image=Ubuntu2204
osDiskSku=StandardSSD_ZRS
dataDiskSku=Premium_ZRS

az group create -n $rgName -l $location

az vmss create -g $rgName \
-n $vmssName \
-l $location \
--encryption-at-host \
--image $image \
--upgrade-policy automatic \
--generate-ssh-keys \
--data-disk-sizes-gb 128 \
--storage-sku os=$osDiskSku 0=$dataDiskSku
```
# [Azure PowerShell](#tab/azure-powershell)

### Create a virtual machine with ZRS disks

The following Azure PowerShell example uses [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount), [Set-AzContext](/powershell/module/az.accounts/set-azcontext), and [New-AzVM](/powershell/module/az.compute/new-azvm) to create a Windows VM with a Standard SSD ZRS OS disk and a 128-GiB Premium SSD ZRS data disk. Replace the variable values with the subscription, administrator credentials, region, existing resource group, VM name, and VM size for your deployment.

```powershell
$subscriptionId="yourSubscriptionId"
$vmLocalAdminUser = "yourAdminUserName"
$vmLocalAdminSecurePassword = ConvertTo-SecureString "yourVMPassword" -AsPlainText -Force
$location = "westus2"
$rgName = "yourResourceGroupName"
$vmName = "yourVMName"
$vmSize = "Standard_DS2_v2"
$osDiskSku = "StandardSSD_ZRS"
$dataDiskSku = "Premium_ZRS"


Connect-AzAccount

Set-AzContext -Subscription $subscriptionId

$subnet = New-AzVirtualNetworkSubnetConfig -Name $($vmName+"_subnet") `
                                           -AddressPrefix "10.0.0.0/24"

$vnet = New-AzVirtualNetwork -Name $($vmName+"_vnet") `
                             -ResourceGroupName $rgName `
                             -Location $location `
                             -AddressPrefix "10.0.0.0/16" `
                             -Subnet $subnet

$nic = New-AzNetworkInterface -Name $($vmName+"_nic") `
                              -ResourceGroupName $rgName `
                              -Location $location `
                              -SubnetId $vnet.Subnets[0].Id


$vm = New-AzVMConfig -VMName $vmName `
                     -VMSize $vmSize


$credential = New-Object System.Management.Automation.PSCredential ($vmLocalAdminUser, $vmLocalAdminSecurePassword);

$vm = Set-AzVMOperatingSystem -VM $vm `
                              -ComputerName $vmName `
                              -Windows `
                              -Credential $credential

$vm = Add-AzVMNetworkInterface -VM $vm -Id $NIC.Id

$vm = Set-AzVMSourceImage -VM $vm `
                          -PublisherName 'MicrosoftWindowsServer' `
                          -Offer 'WindowsServer' `
                          -Skus '2012-R2-Datacenter' `
                          -Version latest


$vm = Set-AzVMOSDisk -VM $vm `
                     -Name $($vmName +"_OSDisk") `
                     -CreateOption FromImage `
                     -StorageAccountType $osDiskSku

$vm = Add-AzVMDataDisk -VM $vm `
                       -Name $($vmName +"_DataDisk1") `
                       -DiskSizeInGB 128 `
                       -StorageAccountType $dataDiskSku `
                       -CreateOption Empty -Lun 0

New-AzVM -ResourceGroupName $rgName `
         -Location $location `
         -VM $vm -Verbose
```

### Create virtual machines with a shared ZRS disk in different zones

The following Azure PowerShell example uses [New-AzDisk](/powershell/module/az.compute/new-azdisk) to create a 1-TiB shared Premium SSD ZRS disk that supports two attachments. It uses [New-AzVM](/powershell/module/az.compute/new-azvm) to create two VMs in availability zones 1 and 2, [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk) to attach the shared disk to each VM, and [Update-AzVM](/powershell/module/az.compute/update-azvm) to apply each attachment. Replace the variable values with the region, existing resource group, VM names, VM size, shared disk name, and administrator credentials for your deployment.

```powershell
$location = "westus2"
$rgName = "yourResourceGroupName"
$vmNamePrefix = "yourVMPrefix"
$vmSize = "Standard_DS2_v2"
$sharedDiskName = "yourSharedDiskName"
$sharedDataDiskSku = "Premium_ZRS"
$vmLocalAdminUser = "yourVMAdminUserName"
$vmLocalAdminSecurePassword = ConvertTo-SecureString "yourPassword" -AsPlainText -Force


$datadiskconfig = New-AzDiskConfig -Location $location `
                                   -DiskSizeGB 1024 `
                                   -AccountType $sharedDataDiskSku `
                                   -CreateOption Empty `
                                   -MaxSharesCount 2 `

$sharedDisk=New-AzDisk -ResourceGroupName $rgName `
            -DiskName $sharedDiskName `
            -Disk $datadiskconfig

$credential = New-Object System.Management.Automation.PSCredential ($vmLocalAdminUser, $vmLocalAdminSecurePassword);

$vm1 = New-AzVM `
        -ResourceGroupName $rgName `
        -Name $($vmNamePrefix+"01") `
        -Zone 1 `
        -Location $location `
        -Size $vmSize `
        -VirtualNetworkName $($vmNamePrefix+"_vnet") `
        -SubnetName $($vmNamePrefix+"_subnet") `
        -SecurityGroupName $($vmNamePrefix+"01_sg") `
        -PublicIpAddressName $($vmNamePrefix+"01_ip") `
        -Credential $credential


$vm1 = Add-AzVMDataDisk -VM $vm1 -Name $sharedDiskName -CreateOption Attach -ManagedDiskId $sharedDisk.Id -Lun 0

Update-AzVM -VM $vm1 -ResourceGroupName $rgName

$vm2 = New-AzVM `
        -ResourceGroupName $rgName `
        -Name $($vmNamePrefix+"02") `
        -Zone 2 `
        -Location $location `
        -Size $vmSize `
        -VirtualNetworkName $($vmNamePrefix+"_vnet") `
        -SubnetName ($vmNamePrefix+"_subnet") `
        -SecurityGroupName $($vmNamePrefix+"02_sg") `
        -PublicIpAddressName $($vmNamePrefix+"02_ip") `
        -Credential $credential `
        -OpenPorts 80,3389


$vm2 = Add-AzVMDataDisk -VM $vm2 -Name $sharedDiskName -CreateOption Attach -ManagedDiskId $sharedDisk.Id -Lun 0

Update-AzVM -VM $vm2 -ResourceGroupName $rgName
```

### Create a virtual machine scale set with ZRS disks

The following Azure PowerShell example uses [New-AzVmss](/powershell/module/az.compute/new-azvmss) to create a Windows virtual machine scale set in an existing resource group. The scale set uses Standard SSD ZRS OS disks and 128-GiB Premium SSD ZRS data disks. Replace the variable values with the region, existing resource group, scale set name, VM size, and administrator credentials for your deployment.

```powershell
$vmLocalAdminUser = "yourLocalAdminUser"
$vmLocalAdminSecurePassword = ConvertTo-SecureString "yourVMPassword" -AsPlainText -Force
$location = "westus2"
$rgName = "yourResourceGroupName"
$vmScaleSetName = "yourScaleSetName"
$vmSize = "Standard_DS3_v2"
$osDiskSku = "StandardSSD_ZRS"
$dataDiskSku = "Premium_ZRS"


$subnet = New-AzVirtualNetworkSubnetConfig -Name $($vmScaleSetName+"_subnet") `
                                                 -AddressPrefix "10.0.0.0/24"

$vnet = New-AzVirtualNetwork -Name $($vmScaleSetName+"_vnet") `
                             -ResourceGroupName $rgName `
                             -Location $location `
                             -AddressPrefix "10.0.0.0/16" `
                             -Subnet $subnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" `
                               -SubnetId $vnet.Subnets[0].Id


$vmss = New-AzVmssConfig -Location $location `
                         -SkuCapacity 2 `
                         -SkuName $vmSize `
                         -UpgradePolicyMode 'Automatic'

$vmss = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" `
                                                -VirtualMachineScaleSet $vmss `
                                                -Primary $true `
                                                -IpConfiguration $ipConfig

$vmss = Set-AzVmssStorageProfile $vmss -OsDiskCreateOption "FromImage" `
                                       -ImageReferenceOffer 'WindowsServer' `
                                       -ImageReferenceSku '2012-R2-Datacenter' `
                                       -ImageReferenceVersion latest `
                                       -ImageReferencePublisher 'MicrosoftWindowsServer' `
                                       -ManagedDisk $osDiskSku

$vmss = Set-AzVmssOsProfile $vmss -ComputerNamePrefix $vmScaleSetName `
                                  -AdminUsername $vmLocalAdminUser `
                                  -AdminPassword $vmLocalAdminSecurePassword

$vmss = Add-AzVmssDataDisk -VirtualMachineScaleSet $vmss `
                           -CreateOption Empty `
                           -Lun 1 `
                           -DiskSizeGB 128 `
                           -StorageAccountType $dataDiskSku

New-AzVmss -VirtualMachineScaleSet $vmss `
           -ResourceGroupName $rgName `
           -VMScaleSetName $vmScaleSetName
```

# [Resource Manager template](#tab/azure-resource-manager)

The templates in this section use version `2020-12-01` of the Microsoft.Compute API to deploy ZRS managed disks.

### Prerequisites

You must enable the `SsdZrsManagedDisks` feature for your subscription. Use [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) to register the feature, then use [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature) to confirm its registration state.

1. Register the feature for your subscription.

    ```powershell
     Register-AzProviderFeature -FeatureName "SsdZrsManagedDisks" -ProviderNamespace "Microsoft.Compute"
    ```

1. Confirm that the registration state is **Registered** before you deploy a template. Registration might take a few minutes.

    ```powershell
     Get-AzProviderFeature -FeatureName "SsdZrsManagedDisks" -ProviderNamespace "Microsoft.Compute"
    ```

### Create a virtual machine with ZRS disks

The following Azure PowerShell example uses [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) to create a resource group and [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) to deploy a Resource Manager template that creates a VM with a Standard SSD ZRS OS disk and a Premium SSD ZRS data disk. Replace the variable values with the VM name, administrator credentials, region, and resource group for your deployment.

```powershell
$vmName = "yourVMName"
$adminUsername = "yourAdminUsername"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_ZRS"
$dataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "yourResourceGroupName"

New-AzResourceGroup -Name $resourceGroupName -Location $region
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMWithZRSDataDisks.json" `
-resourceName $vmName `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $dataDiskType
```

### Create virtual machines with a shared ZRS disk in different zones

The following Azure PowerShell example uses [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) to deploy a Resource Manager template that creates two VMs in different availability zones and attaches the same shared Premium SSD ZRS disk to both VMs. Replace the variable values with the VM name prefix, administrator credentials, region, and existing resource group for your deployment.

```powershell
$vmNamePrefix = "yourVMNamePrefix"
$adminUsername = "yourAdminUserName"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_LRS"
$sharedDataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "zrstesting1"

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMsWithASharedDisk.json" `
-vmNamePrefix $vmNamePrefix `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $sharedDataDiskType
```

### Create a virtual machine scale set with ZRS disks

The following Azure PowerShell example uses [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) to deploy a Resource Manager template that creates a virtual machine scale set with Standard SSD LRS OS disks and Premium SSD ZRS data disks. Replace the variable values with the scale set name, administrator credentials, region, and existing resource group for your deployment.

```powershell
$vmssName="yourVMSSName"
$adminUsername="yourAdminName"
$adminPassword=ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$region="eastus2euap"
$resourceGroupName="yourResourceGroupName"
$osDiskType="StandardSSD_LRS"
$dataDiskType="Premium_ZRS"

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMSSWithZRSDisks.json" `
-vmssName $vmssName `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $dataDiskType
```
---

## Next steps

- Check out more [Azure Resource Manager templates to create a VM with ZRS disks](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/ZRSDisks).
