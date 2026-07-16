---
title: Deploy a nonzonal Premium SSD v2 managed disk
description: Learn how to deploy a nonzonal Premium SSD v2 and review nonzonal limitations.
author: roygara
ms.author: rogarana
ms.date: 06/17/2026
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
# Customer intent: As a cloud administrator, I want to deploy a nonzonal Premium SSD v2 disk, so that I can support workloads that don't require zonal alignment.
---

# Deploy a nonzonal Premium SSD v2

Azure Premium SSD v2 is designed for I/O-intense enterprise workloads that require sub-millisecond disk latencies and high IOPS and throughput at a low cost. For conceptual information on Premium SSD v2, see [Premium SSD v2](/azure/virtual-machines/disks-types#premium-ssd-v2).

You can deploy Premium SSD v2 [nonzonal](/azure/reliability/availability-zones-zonal-resource-resiliency#resource-deployment-types) disks in [select regions](#regional-availability), including regions with and without availability zones.

Premium SSD v2 disks support a 4k physical sector size by default, but you can configure them to use a 512E sector size. While most applications are compatible with 4k sector sizes, some require 512 byte sector sizes. Oracle Database, for example, requires release 12.2 or later to support 4k native disks.

## Limitations

[!INCLUDE [disks-prem-v2-limitations](./includes/disks-prem-v2-limitations.md)]

### Limitations for nonzonal Premium SSD v2 in regions with availability zones

When you attach a [nonzonal](/azure/reliability/availability-zones-zonal-resource-resiliency#resource-deployment-types) Premium SSD v2 to a nonzonal VM in an AZ region, Azure runs a background copy (can take up to 24 hours) to align the disk with the VM's availability zone and optimize latency.

The following additional limitations apply to nonzonal Premium SSD v2 disks:

- You can't attach a nonzonal disk created from a snapshot, including an [instant access snapshot](/azure/virtual-machines/disks-instant-access-snapshots), to a nonzonal VM until the background copy finishes. To check the background copy status, see [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot#performance-impact---background-copy-process).
- You can't resize the nonzonal disk or change customer-managed key during a background copy.

Only one background copy can run on a nonzonal disk at a time. While a background copy is in progress, attaching the nonzonal disk to a running nonzonal VM might fail. Restarting a stopped or deallocated nonzonal VM with the nonzonal disk attached might also fail, because the restart can trigger a second background copy.

### Regional availability

[!INCLUDE [disks-premv2-regions-nonzonal](./includes/disks-premv2-regions-nonzonal.md)]

## Prerequisites

- Install either the latest [Azure CLI](/cli/azure/install-azure-cli) or the latest [Azure PowerShell module](/powershell/azure/install-azure-powershell).

## Determine region availability programmatically

Since not every region and zone supports Premium SSD v2 disks, use the Azure CLI or PowerShell to check region and zone support.

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

To programmatically determine the regions and zones you can deploy to, use either the Azure CLI or Azure PowerShell module.

---

Now that you know the region and zone to deploy to, follow the deployment steps in this article to create a nonzonal Premium SSD v2 and attach it to a VM.

To deploy a zonal Premium SSD v2 instead, see [Deploy a Premium SSD v2 managed disk](disks-deploy-premium-v2.md).

## Use a nonzonal Premium SSD v2

# [Azure CLI](#tab/azure-cli)

Create a nonzonal Premium SSD v2 by using the [az disk create](/cli/azure/disk#az-disk-create) command. Then create a nonzonal VM in the same region that supports Premium Storage and attach the disk to it by using the [az vm create](/cli/azure/vm#az-vm-create) command.

The following script creates a Premium SSD v2 with a 4k sector size. To create a disk with a 512 sector size, update the `$logicalSectorSize` parameter. Replace the values of all the variables with your own, and then run the following script:

```azurecli-interactive
## Initialize variables
diskName="yourDiskName"
resourceGroupName="yourResourceGroupName"
region="yourRegionName"
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
--sku PremiumV2_LRS \
--logical-sector-size $logicalSectorSize

## Create the VM
az vm create -n $vmName -g $resourceGroupName \
--image $vmImage \
--authentication-type password --admin-password $adminPassword --admin-username $adminUserName \
--size $vmSize \
--location $region \
--attach-data-disks $diskName
```

# [PowerShell](#tab/azure-powershell)

Create a nonzonal Premium SSD v2 by using [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) and [New-AzDisk](/powershell/module/az.compute/new-azdisk). Then create a nonzonal VM in the same region, add the disk to the VM configuration with [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk), and apply the updated configuration in Azure with [Update-AzVM](/powershell/module/az.compute/update-azvm).

The following script creates a Premium SSD v2 with a 4k sector size. To create a disk with a 512 sector size, update the `$logicalSectorSize` parameter. Replace the values of all the variables with your own, and then run the following script:

```powershell
# Initialize variables
$resourceGroupName = "yourResourceGroupName"
$region = "useast"
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
1. Go to **Disks** and create a new disk.
1. Select a [supported region](#regional-availability).
1. Select **Change size** and change the disk type to **Premium SSD v2**.
1. If you want, change the size of the disk and the performance, and then select **OK**.
1. Set **Availability zone** to **No infrastructure redundancy required**.
1. Proceed through the rest of the deployment, making any choices that you want.
1. On the **Advanced** tab, select whether you want to deploy a 4k or 512 logical sector size, and then deploy the disk.

 When the disk is successfully deployed, attach it to a new or existing VM.

---

## Next steps

Add a data disk by using either the [Azure portal](/azure/virtual-machines/linux/attach-disk-portal), [Azure CLI](linux/add-disk.md), or [PowerShell](windows/attach-disk-ps.md).

Use [Premium SSD v2 with VMs in availability set](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set).

Provide feedback on [Premium SSD v2](https://aka.ms/premium-ssd-v2-survey).
