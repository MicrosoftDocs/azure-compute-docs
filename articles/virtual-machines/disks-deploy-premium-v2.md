---
title: Deploy a Premium SSD v2 managed disk
description: Learn how to deploy zonal and nonzonal Premium SSD v2 managed disks and review their regional availability.
author: roygara
ms.author: rogarana
ms.date: 08/11/2026
ms.topic: how-to
ms.service: azure-disk-storage
ai-usage: ai-assisted
ms.custom:
  - references_regions
  - devx-track-azurecli
  - devx-track-azurepowershell
  - innovation-engine
  - sfi-ropc-nochange\portal
  - portal
# Customer intent: As a cloud administrator, I want to deploy a zonal or nonzonal Premium SSD v2 disk, so that I can support IO-intensive workloads with low latency and high throughput.
---

# Deploy a Premium SSD v2

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://go.microsoft.com/fwlink/?linkid=2303310)

Azure Premium SSD v2 is designed for I/O-intensive enterprise workloads that require submillisecond disk latency, high IOPS, and high throughput at a low cost. Premium SSD v2 is suited for workloads such as SQL Server, Oracle, MariaDB, SAP, Cassandra, MongoDB, big data and analytics, and gaming on virtual machines or stateful containers. For conceptual information on Premium SSD v2, see [Premium SSD v2](/azure/virtual-machines/disks-types#premium-ssd-v2).

You can deploy Premium SSD v2 disks as zonal resources or as [nonzonal resources](/azure/reliability/availability-zones-zonal-resource-resiliency#resource-deployment-types). Use a zonal disk when your workload runs in an availability zone. Use a nonzonal disk for a nonzonal VM or in a region without availability zones.

Premium SSD v2 disks support a 4k physical sector size by default, but you can configure them to use a 512E sector size. While most applications are compatible with 4k sector sizes, some require 512-byte sector sizes. Oracle Database, for example, requires release 12.2 or later to support 4k native disks.

## Premium SSD v2 limitations

[!INCLUDE [disks-prem-v2-limitations](./includes/disks-prem-v2-limitations.md)]

## Regional availability

### Zonal disks

[!INCLUDE [disks-premv2-regions](./includes/disks-premv2-regions.md)]

### Nonzonal disks

[!INCLUDE [disks-premv2-regions-nonzonal](./includes/disks-premv2-regions-nonzonal.md)]

## Prerequisites

- Install either the latest [Azure CLI](/cli/azure/install-azure-cli) or the latest [Azure PowerShell module](/powershell/azure/install-azure-powershell). 

## Determine region availability programmatically

Because not every region and zone supports Premium SSD v2 disks, use the Azure CLI or PowerShell to check region and zone support.

# [Azure CLI](#tab/azure-cli)

To find the regions and zones that support Premium SSD v2 disks, replace `yourSubscriptionId` with your subscription, and then run the [az vm list-skus](/cli/azure/vm#az-vm-list-skus) command:

```azurecli
az login

subscriptionId="<yourSubscriptionId>"

az account set --subscription $subscriptionId

az vm list-skus --resource-type disks --query "[?name=='PremiumV2_LRS'].{Region:locationInfo[0].location, Zones:locationInfo[0].zones}" 
```

# [PowerShell](#tab/azure-powershell)

To find the regions and zones that support Premium SSD v2 disks, replace `yourSubscriptionId` with your subscription, and then run the [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku) command:

```powershell
Connect-AzAccount

$subscriptionId="yourSubscriptionId"

Set-AzContext -Subscription $subscriptionId

Get-AzComputeResourceSku | where {$_.ResourceType -eq 'disks' -and $_.Name -eq 'Premiumv2_LRS'} 
```

# [Azure portal](#tab/portal)

To programmatically determine the regions and zones you can deploy to, use either Azure CLI or Azure PowerShell.

---

After you identify a supported region and zone, choose the deployment type that matches your VM.

## Deploy a zonal Premium SSD v2

Zonal Premium SSD v2 disks are available in [select regions with availability zones](#zonal-disks).

# [Azure CLI](#tab/azure-cli)

Create a Premium SSD v2 in an availability zone by using the [az disk create](/cli/azure/disk#az-disk-create) command. Then create a VM in the same region and availability zone that supports Premium Storage and attach the disk to it by using the [az vm create](/cli/azure/vm#az-vm-create) command. 

The following script creates a Premium SSD v2 with a 4k sector size. To deploy one with a 512 sector size, update the `$logicalSectorSize` parameter. Replace the values of all the variables with your own, then run the following script:

```azurecli-interactive
## Initialize variables
diskName="yourDiskName"
resourceGroupName="yourResourceGroupName"
region="yourRegionName"
zone="yourZoneNumber"
##Replace 4096 with 512 to deploy a disk with 512 sector size
logicalSectorSize=4096
vmName="yourVMName"
vmImage="Win2016Datacenter"
adminPassword="yourAdminPassword"
adminUserName="yourAdminUserName"
vmSize="Standard_D4s_v3"

## Create a Premium SSD v2
az disk create -n $diskName -g $resourceGroupName \
--size-gb 100 \
--disk-iops-read-write 5000 \
--disk-mbps-read-write 150 \
--location $region \
--zone $zone \
--sku PremiumV2_LRS \
--logical-sector-size $logicalSectorSize

## Create the VM
az vm create -n $vmName -g $resourceGroupName \
--image $vmImage \
--zone $zone \
--authentication-type password --admin-password $adminPassword --admin-username $adminUserName \
--size $vmSize \
--location $region \
--attach-data-disks $diskName
```

# [PowerShell](#tab/azure-powershell)

Create a Premium SSD v2 in an availability zone by using [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) and [New-AzDisk](/powershell/module/az.compute/new-azdisk). Then create a VM in the same region and availability zone. Use [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk) to add the disk to the VM configuration, and then use [Update-AzVM](/powershell/module/az.compute/update-azvm) to apply the change in Azure.

The following script creates a Premium SSD v2 with a 4k sector size. To deploy one with a 512 sector size, update the `$logicalSectorSize` parameter. Replace the values of all the variables with your own, then run the following script:

```powershell
# Initialize variables
$resourceGroupName = "yourResourceGroupName"
$region = "useast"
$zone = "yourZoneNumber"
$diskName = "yourDiskName"
$diskSizeInGiB = 100
$diskIOPS = 5000
$diskThroughputInMBPS = 150
#To use a 512 sector size, replace 4096 with 512
$logicalSectorSize=4096
$lun = 1
$vmName = "yourVMName"
$vmImage = "Win2016Datacenter"
$vmSize = "Standard_D4s_v3"
$vmAdminUser = "yourAdminUserName"
$vmAdminPassword = ConvertTo-SecureString "yourAdminUserPassword" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($vmAdminUser, $vmAdminPassword);

# Create a Premium SSD v2
$diskconfig = New-AzDiskConfig `
-Location $region `
-Zone $zone `
-DiskSizeGB $diskSizeInGiB `
-DiskIOPSReadWrite $diskIOPS `
-DiskMBpsReadWrite $diskThroughputInMBPS `
-AccountType PremiumV2_LRS `
-LogicalSectorSize $logicalSectorSize `
-CreateOption Empty

New-AzDisk `
-ResourceGroupName $resourceGroupName `
-DiskName $diskName `
-Disk $diskconfig

# Create the VM
New-AzVm `
    -ResourceGroupName $resourceGroupName `
    -Name $vmName `
    -Location $region `
    -Zone $zone `
    -Image $vmImage `
    -Size $vmSize `
    -Credential $credential

# Attach the disk to the VM
$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

# [Azure portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Navigate to **Virtual machines** and follow the normal VM creation process.
1. On the **Basics** page, select a [supported region](#regional-availability) and set **Availability options** to **Availability zone**.

    For a zonal deployment, create a zonal VM or Virtual Machine Scale Set, and specify the availability zone you want before adding Premium SSD v2 disks to your configuration.

1. Select one or more of the zones.
1. Fill in the rest of the values on the page as you like.

    :::image type="content" source="media/disks-deploy-premium-v2/premv2-portal-deploy.png" alt-text="Screenshot of the basics page, region and availability options and zones highlighted." lightbox="media/disks-deploy-premium-v2/premv2-portal-deploy.png":::

1. Proceed to the **Disks** page.
1. Under **Data disks** select **Create and attach a new disk**.

    :::image type="content" source="media/disks-deploy-premium-v2/premv2-create-data-disk.png" alt-text="Screenshot highlighting create and attach a new disk on the disk page." lightbox="media/disks-deploy-premium-v2/premv2-create-data-disk.png":::

1. Select the **Disk SKU** and select **Premium SSD v2**.

    :::image type="content" source="media/disks-deploy-premium-v2/premv2-select.png" alt-text="Screenshot selecting Premium SSD v2 SKU." lightbox="media/disks-deploy-premium-v2/premv2-select.png":::

1. Select whether you'd like to deploy a 4k or 512 logical sector size.

    :::image type="content" source="media/disks-deploy-premium-v2/premv2-sector-size.png" alt-text="Screenshot of deployment logical sector size deployment options." lightbox="media/disks-deploy-premium-v2/premv2-sector-size.png":::

1. Proceed through the rest of the VM deployment, making any choices that you desire.

You've now deployed a VM with a Premium SSD v2.

---

## Deploy a nonzonal Premium SSD v2

Deploy a nonzonal disk for a nonzonal VM in a [supported region](#nonzonal-disks), including regions with and without availability zones.

<a name="limitations-for-nonzonal-disks-in-regions-with-availability-zones"></a>

## Additional limitations for nonzonal disks in regions with availability zones

When you attach a nonzonal Premium SSD v2 disk to a nonzonal VM in a region with availability zones, Azure runs a background copy to align the disk with the VM's availability zone and optimize latency. The copy can take up to 24 hours.

The following additional limitations apply during the background copy:

- You can't attach a nonzonal disk created from a snapshot, including an [instant access snapshot](/azure/virtual-machines/disks-instant-access-snapshots), to a nonzonal VM until the copy finishes. To check the copy status, see [Performance impact of background copy](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot#performance-impact---background-copy-process).
- You can't resize the disk or change its customer-managed key.

Only one background copy can run on a nonzonal disk at a time. While a background copy is in progress, attaching the disk to a running nonzonal VM might fail. Restarting a stopped or deallocated nonzonal VM with the disk attached might also fail because the restart can trigger a second background copy.

# [Azure CLI](#tab/azure-cli)

Create a nonzonal Premium SSD v2 disk by using the [az disk create](/cli/azure/disk#az-disk-create) command. Then create a nonzonal VM in the same region that supports Premium Storage and attach the disk by using the [az vm create](/cli/azure/vm#az-vm-create) command.

The following script creates a Premium SSD v2 disk with a 4k sector size. To create a disk with a 512E sector size, change the `logicalSectorSize` value to `512`. Replace the variable values with your own before you run the script.

```azurecli-interactive
## Initialize variables
diskName="yourDiskName"
resourceGroupName="yourResourceGroupName"
region="yourRegionName"
logicalSectorSize=4096
vmName="yourVMName"
vmImage="Win2016Datacenter"
adminPassword="yourAdminPassword"
adminUserName="yourAdminUserName"
vmSize="Standard_D4s_v3"

## Create a Premium SSD v2 disk
az disk create -n $diskName -g $resourceGroupName \
--size-gb 100 \
--disk-iops-read-write 5000 \
--disk-mbps-read-write 150 \
--location $region \
--sku PremiumV2_LRS \
--logical-sector-size $logicalSectorSize

## Create the VM and attach the disk
az vm create -n $vmName -g $resourceGroupName \
--image $vmImage \
--authentication-type password --admin-password $adminPassword --admin-username $adminUserName \
--size $vmSize \
--location $region \
--attach-data-disks $diskName
```

# [PowerShell](#tab/azure-powershell)

Create a nonzonal Premium SSD v2 disk by using [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) and [New-AzDisk](/powershell/module/az.compute/new-azdisk). Then create a nonzonal VM in the same region, add the disk to the VM configuration with [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk), and apply the updated configuration with [Update-AzVM](/powershell/module/az.compute/update-azvm).

The following script creates a Premium SSD v2 disk with a 4k sector size. To create a disk with a 512E sector size, change the `$logicalSectorSize` value to `512`. Replace the variable values with your own before you run the script.

```powershell
# Initialize variables
$resourceGroupName = "yourResourceGroupName"
$region = "useast"
$diskName = "yourDiskName"
$diskSizeInGiB = 100
$diskIOPS = 5000
$diskThroughputInMBPS = 150
$logicalSectorSize = 4096
$lun = 1
$vmName = "yourVMName"
$vmImage = "Win2016Datacenter"
$vmSize = "Standard_D4s_v3"
$vmAdminUser = "yourAdminUserName"
$vmAdminPassword = ConvertTo-SecureString "yourAdminUserPassword" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($vmAdminUser, $vmAdminPassword)

# Create a Premium SSD v2 disk
$diskConfig = New-AzDiskConfig `
    -Location $region `
    -DiskSizeGB $diskSizeInGiB `
    -DiskIOPSReadWrite $diskIOPS `
    -DiskMBpsReadWrite $diskThroughputInMBPS `
    -AccountType PremiumV2_LRS `
    -LogicalSectorSize $logicalSectorSize `
    -CreateOption Empty

New-AzDisk `
    -ResourceGroupName $resourceGroupName `
    -DiskName $diskName `
    -Disk $diskConfig

# Create the VM
New-AzVm `
    -ResourceGroupName $resourceGroupName `
    -Name $vmName `
    -Location $region `
    -Image $vmImage `
    -Size $vmSize `
    -Credential $credential

# Attach the disk to the VM
$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

# [Azure portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Go to **Disks**, and create a disk.
1. Select a [supported region](#nonzonal-disks).
1. Select **Change size**, select **Premium SSD v2**, and then select **OK**.
1. Set **Availability zone** to **No infrastructure redundancy required**.
1. On the **Advanced** tab, select a 4k or 512E logical sector size.
1. Complete the remaining settings, and then create the disk.
1. Attach the disk to a new or existing VM.

---

## Adjust disk performance

You can adjust the performance of a Premium SSD v2 four times within a 24 hour period. Creating a disk counts as one of these times, so for the first 24 hours after creating a Premium SSD v2 you can only adjust its performance up to three times.

For conceptual information on adjusting disk performance, see [Premium SSD v2 performance](/azure/virtual-machines/disks-types#premium-ssd-v2-performance).

# [Azure CLI](#tab/azure-cli)

Use the [az disk update](/cli/azure/disk#az-disk-update) command to change the performance configuration of your Premium SSD v2. For example, you can use the `disk-iops-read-write` parameter to adjust the max IOPS limit, and the `disk-mbps-read-write` parameter to adjust the max throughput limit of your Premium SSD v2.  

The following command adjusts the performance of your disk. Update the values in the command, and then run it:

```azurecli
az disk update --subscription $subscription --resource-group $rgname --name $diskName --disk-iops-read-write=5000 --disk-mbps-read-write=200
```

# [PowerShell](#tab/azure-powershell)

Use the [New-AzDiskUpdateConfig](/powershell/module/az.compute/new-azdiskupdateconfig) command to define your new performance configuration values for your Premium SSD v2 disks, and then use the [Update-AzDisk](/powershell/module/az.compute/update-azdisk) command to apply your configuration changes to your disk. For example, you can use the `DiskIOPSReadWrite` parameter to adjust the max IOPS limit, and the `DiskMBpsReadWrite` parameter to adjust the max throughput limit of your Premium SSD v2.  

The following command adjusts the performance of your disk. Update the values in the command, and then run it:

```azurepowershell
$diskupdateconfig = New-AzDiskUpdateConfig -DiskIOPSReadWrite 5000 -DiskMBpsReadWrite 200
Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -DiskUpdate $diskupdateconfig
```

# [Azure portal](#tab/portal)

1. Navigate to the disk you'd like to modify in the [Azure portal](https://portal.azure.com/).
1. Select **Size + Performance**
1. Set the values for **Disk IOPS** or **Disk throughput (MB/s)** or both, to meet your needs, then select **Save**.

---

## Next steps

Add a data disk by using either the [Azure portal](/azure/virtual-machines/linux/attach-disk-portal), [Azure CLI](linux/add-disk.md), or [PowerShell](windows/attach-disk-ps.md).

Use [Premium SSD v2 with VMs in availability set](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set).

Provide feedback on [Premium SSD v2](https://aka.ms/premium-ssd-v2-survey).
