---
title: Configure Ultra Disks for Azure virtual machines
description: Learn how to check Ultra Disk availability, deploy and attach Ultra Disks, configure 512-byte sectors, and adjust performance for Azure virtual machines.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 09/18/2026
ms.author: rogarana
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, devx-track-arm-template, portal
# Customer intent: "As a cloud administrator, I want to deploy Ultra Disks for Azure VMs, so that I can achieve maximum performance for data-intensive workloads and optimize that performance without needing to restart my virtual machines."
---

# Configure Ultra Disks for Azure virtual machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article explains how to check Ultra Disk availability, deploy a VM with an Ultra Disk, configure an Ultra Disk with a 512-byte sector size, attach an Ultra Disk to an existing VM, and adjust Ultra Disk performance. For conceptual information about Ultra Disks, see [What disk types are available in Azure?](disks-types.md#ultra-disks)

Azure Ultra Disks offer high throughput, high IOPS, and consistent low latency disk storage for Azure IaaS virtual machines (VMs). This new offering provides top of the line performance at the same availability levels as our existing disks offerings. One major benefit of Ultra Disks is the ability to dynamically change the performance of the SSD along with your workloads without the need to restart your VMs. Ultra Disks are suited for data-intensive workloads such as SAP HANA, top tier databases, and transaction-heavy workloads.

## GA scope and limitations

[!INCLUDE [managed-disks-ultra-disks-ga-scope-and-limitations](includes/managed-disks-ultra-disks-ga-scope-and-limitations.md)]

## Determine VM size and region availability

### VMs using availability zones

To use Ultra Disks, you need to determine which availability zone you are in. Not every region supports every VM size with Ultra Disks. To determine if your region, zone, and VM size support Ultra Disks, run either of the following commands, make sure to replace the **region**, **vmSize**, and **subscriptionId** values first:

#### Azure CLI

Use [`az vm list-skus`](/cli/azure/vm#az-vm-list-skus) to check Ultra Disk availability for a VM size, region, and subscription.

```azurecli
subscriptionId="<yourSubID>"
# Example value is southeastasia
region="<yourLocation>"
# Example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines --location $region --query "[?name=='$vmSize'].locationInfo[0].zoneDetails[0].Name" --subscription $subscriptionId
```

#### Azure PowerShell

Use [`Get-AzComputeResourceSku`](/powershell/module/az.compute/get-azcomputeresourcesku) to check Ultra Disk availability for a VM size and region.

```powershell
# Example value is southeastasia
$region = "<yourLocation>"
# Example value is Standard_E64s_v3
$vmSize = "<yourVMSize>"
$sku = (Get-AzComputeResourceSku | where {$_.Locations -icontains($region) -and ($_.Name -eq $vmSize) -and $_.LocationInfo[0].ZoneDetails.Count -gt 0})
if($sku){$sku[0].LocationInfo[0].ZoneDetails} Else {Write-host "$vmSize is not supported with Ultra Disk in $region region"}
```

The response will be similar to the form below, where X is the zone to use for deploying in your chosen region. X could be either 1, 2, or 3.

Preserve the **Zones** value, it represents your availability zone and you'll need it in order to deploy an Ultra Disk.

|ResourceType  |Name  |Location  |Zones  |Restriction  |Capability  |Value  |
|---------|---------|---------|---------|---------|---------|---------|
|disks     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

> [!NOTE]
> If there was no response from the command, then the selected VM size is not supported with Ultra Disks in the selected region.

Now that you know which zone to deploy to, follow the deployment steps in this article to either deploy a VM with an Ultra Disk attached or attach an Ultra Disk to an existing VM.

### VMs with no redundancy options

Ultra Disks deployed in select regions must be deployed without any redundancy options for now. However, not every VM size that supports Ultra Disks are necessarily in these regions. To determine which VM sizes support Ultra Disks, use either of the following code snippets. Make sure to replace the `vmSize`, `region`, and `subscriptionId` values first:

#### Azure CLI

```azurecli
subscriptionId="<yourSubID>"
# Example value is westus
region="<yourLocation>"
# Example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines --location $region --query "[?name=='$vmSize'].capabilities" --subscription $subscriptionId
```

#### Azure PowerShell

```powershell
# Example value is westus
$region = "<yourLocation>"
# Example value is Standard_E64s_v3
$vmSize = "<yourVMSize>"
(Get-AzComputeResourceSku | where {$_.Locations -icontains($region) -and ($_.Name -eq $vmSize) })[0].Capabilities
```

The response will be similar to the following form, `UltraSSDAvailable   True` indicates whether the VM size supports Ultra Disks in this region.

```
Name                                         Value
----                                         -----
MaxResourceVolumeMB                          884736
OSVhdSizeMB                                  1047552
vCPUs                                        64
HyperVGenerations                            V1,V2
MemoryGB                                     432
MaxDataDiskCount                             32
LowPriorityCapable                           True
PremiumIO                                    True
VMDeploymentTypes                            IaaS
vCPUsAvailable                               64
ACUs                                         160
vCPUsPerCore                                 2
CombinedTempDiskAndCachedIOPS                128000
CombinedTempDiskAndCachedReadBytesPerSecond  1073741824
CombinedTempDiskAndCachedWriteBytesPerSecond 1073741824
CachedDiskBytes                              1717986918400
UncachedDiskIOPS                             80000
UncachedDiskBytesPerSecond                   1258291200
EphemeralOSDiskSupported                     True
AcceleratedNetworkingEnabled                 True
RdmaEnabled                                  False
MaxNetworkInterfaces                         8
UltraSSDAvailable                            True
```

## Deploy a VM with Ultra Disks by using Azure Resource Manager

First, determine the VM size to deploy. For a list of supported VM sizes, see the [GA scope and limitations](#ga-scope-and-limitations) section.

If you would like to create a VM with multiple Ultra Disks, refer to the sample [Create a VM with multiple Ultra Disks](https://aka.ms/ultradiskArmTemplate).

If you intend to use your own template, make sure that **apiVersion** for `Microsoft.Compute/virtualMachines` and `Microsoft.Compute/Disks` is set as `2018-06-01` (or later).

Set the disk sku to **UltraSSD_LRS**, then set the disk capacity, IOPS, availability zone, and throughput in MBps to create an Ultra Disk.

Once the VM is provisioned, you can partition and format the data disks and configure them for your workloads.


## Deploy a VM with an Ultra Disk

# [Portal](#tab/azure-portal)

This section covers deploying a virtual machine equipped with an Ultra Disk as a data disk. It assumes you have familiarity with deploying a virtual machine, if you don't, see our [Quickstart: Create a Windows virtual machine in the Azure portal](./windows/quick-create-portal.md).

1. Sign in to the [Azure portal](https://portal.azure.com/) and navigate to deploy a virtual machine (VM).
1. Make sure to choose a [supported VM size and region](#ga-scope-and-limitations).
1. Select **Availability zone** in **Availability options**.
1. Fill in the remaining entries with selections of your choice.
1. Select **Disks**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png" alt-text="Screenshot of the Create a virtual machine Basics pane with Availability options set to Availability zone and Availability zone set to Zone 3." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png":::

1. On the Disks blade, select **Yes** for **Enable Ultra Disk compatibility**.
1. Select **Create and attach a new disk** to attach an Ultra Disk now.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-disk-enable.png" alt-text="Screenshot of the virtual machine Disks pane with Enable Ultra Disk compatibility selected and Create and attach a new disk highlighted."  lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-disk-enable.png":::

1. On the **Create a new disk** blade, enter a name, then select **Change size**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-create-disk.png" alt-text="Screenshot of the Create a new disk pane with a 1,024-GiB Premium SSD LRS selected and Change size highlighted." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-create-disk.png":::


1. Change the **Disk SKU** to **Ultra Disk**.
1. Change the values of **Custom disk size (GiB)**, **Disk IOPS**, and **Disk throughput** to ones of your choice.
1. Select **OK** in both blades.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-select-ultra-disk-size.png" alt-text="Screenshot of the Select a disk size pane with the Ultra Disk SKU and custom disk size, IOPS, and throughput settings highlighted." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-select-ultra-disk-size.png":::

1. Continue with the VM deployment, the same as you would deploy any other VM.

# [Azure CLI](#tab/azure-cli)

First, determine the VM size to deploy. See the [GA scope and limitations](#ga-scope-and-limitations) section for a list of supported VM sizes.

You must create a VM that is capable of using Ultra Disks, in order to attach an Ultra Disk.

Set the variables to your own values. Set `zone` to the availability zone that you got from [Determine VM size and region availability](#determine-vm-size-and-region-availability). Then run the following Azure CLI commands to create an Ultra-enabled VM with an attached Ultra Disk:

```azurecli-interactive
subscriptionId="<yourSubscriptionID>"
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
region="<yourLocation>"
zone="<yourAvailabilityZone>"
user="<yourAdminUsername>"
password="<yourAdminPassword>"

az disk create --subscription $subscriptionId -n $diskName -g $rgName --size-gb 1024 --location $region --zone $zone --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400
az vm create --subscription $subscriptionId -n $vmName -g $rgName --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $region --attach-data-disks $diskName
```

Use [`az vm show`](/cli/azure/vm#az-vm-show) to confirm that `ultraSSDEnabled` is `true` and the Ultra Disk appears in the VM's data disks:

```azurecli-interactive
az vm show -g <yourResourceGroupName> -n <yourVMName> --query "{ultraSSDEnabled:additionalCapabilities.ultraSSDEnabled,dataDisks:storageProfile.dataDisks[].name}"
```

# [Azure PowerShell](#tab/azure-powershell)

First, determine the VM size to deploy. See the [GA scope and limitations](#ga-scope-and-limitations) section for a list of supported VM sizes.

To use Ultra Disks, you must create a VM that can use Ultra Disks. Set the variables to your own values. Set `$zone` to the availability zone that you got from [Determine VM size and region availability](#determine-vm-size-and-region-availability). Then run the following [New-AzVM](/powershell/module/az.compute/new-azvm) command to create an Ultra-enabled VM:

```powershell
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$region = "<yourLocation>"
$zone = "<yourAvailabilityZone>"

New-AzVM `
    -ResourceGroupName $rgName `
    -Name $vmName `
    -Location $region `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -Size "Standard_D4s_v3" `
    -Zone $zone
```

### Create and attach the disk

Once your VM has been deployed, you can create and attach an Ultra Disk to it, use the following script:

```powershell
# Set parameters and select subscription
$subscriptionId = "<yourSubscriptionID>"
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$region = "<yourLocation>"
$zone = "<yourAvailabilityZone>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscriptionId

# Create the disk
$diskConfig = New-AzDiskConfig `
    -Location $region `
    -DiskSizeGB 8 `
    -DiskIOPSReadWrite 1000 `
    -DiskMBpsReadWrite 100 `
    -AccountType UltraSSD_LRS `
    -CreateOption Empty `
    -Zone $zone

New-AzDisk `
    -ResourceGroupName $rgName `
    -DiskName $diskName `
    -Disk $diskConfig

# Add disk to VM
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $rgName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $rgName
```

Use [`Get-AzVM`](/powershell/module/az.compute/get-azvm) to confirm that Ultra Disk compatibility is enabled and the Ultra Disk appears in the VM's data disks:

```powershell
$vm = Get-AzVM -ResourceGroupName '<yourResourceGroup>' -Name '<yourVMName>'
$vm.AdditionalCapabilities.UltraSSDEnabled
$vm.StorageProfile.DataDisks | Select-Object Name, Lun
```

---

## Deploy an Ultra Disk with a 512-byte sector size

# [Portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com/), then search for and select **Disks**.
1. Select **+ New** to create a new disk.
1. Select a region that supports Ultra Disks and select an availability zone, fill in the rest of the values as you desire.
1. Select **Change size**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/create-managed-disk-basics-workflow.png" alt-text="Screenshot of the Create a managed disk Basics pane with Region set to West US 2, Availability zone set to 1, and Change size highlighted." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/create-managed-disk-basics-workflow.png":::

1. For **Disk SKU** select **Ultra Disk**, then fill in the values for the desired performance and select **OK**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-disk-size-ultra.png" alt-text="Screenshot of the Select a disk size pane with Ultra Disk selected, custom disk size set to 1,024 GiB, disk IOPS set to 2,048, and disk throughput set to 8 MB/s." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/select-disk-size-ultra.png":::

1. On the **Basics** blade, select the **Advanced** tab.
1. Select **512** for **Logical sector size**, then select **Review + Create**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-different-sector-size-ultra.png" alt-text="Screenshot of the Create a managed disk Advanced pane with Logical sector size set to 512 bytes." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/select-different-sector-size-ultra.png":::

# [Azure CLI](#tab/azure-cli)

First, determine the VM size to deploy. See the [GA scope and limitations](#ga-scope-and-limitations) section for a list of supported VM sizes.

You must create a VM that is capable of using Ultra Disks in order to attach an Ultra Disk.

Set the variables to your own values. Set `zone` to the availability zone that you got from [Determine VM size and region availability](#determine-vm-size-and-region-availability). Then run the following Azure CLI commands to create a VM with an Ultra Disk that has a 512-byte sector size.

```azurecli
subscriptionId="<yourSubscriptionID>"
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
region="<yourLocation>"
zone="<yourAvailabilityZone>"
user="<yourAdminUsername>"
password="<yourAdminPassword>"

# Create an Ultra Disk with 512-byte sector size
az disk create --subscription $subscriptionId -n $diskName -g $rgName --size-gb 1024 --location $region --zone $zone --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400 --logical-sector-size 512
az vm create --subscription $subscriptionId -n $vmName -g $rgName --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $region --attach-data-disks $diskName
```

Use [`az disk show`](/cli/azure/disk#az-disk-show) to confirm that `provisioningState` is `Succeeded` and `logicalSectorSize` is `512`:

```azurecli
az disk show -g <yourResourceGroupName> -n <yourDiskName> --query "{provisioningState:provisioningState,logicalSectorSize:logicalSectorSize}"
```

# [Azure PowerShell](#tab/azure-powershell)

First, determine the VM size to deploy. See the [GA scope and limitations](#ga-scope-and-limitations) section for a list of supported VM sizes.

To use Ultra Disks, you must create a VM that can use Ultra Disks. Set the variables to your own values. Set `$zone` to the availability zone that you got from [Determine VM size and region availability](#determine-vm-size-and-region-availability). Then run the following [New-AzVM](/powershell/module/az.compute/new-azvm) command to create an Ultra-enabled VM:

```powershell
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$region = "<yourLocation>"
$zone = "<yourAvailabilityZone>"

New-AzVM `
    -ResourceGroupName $rgName `
    -Name $vmName `
    -Location $region `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -Size "Standard_D4s_v3" `
    -Zone $zone
```

To create and attach an Ultra Disk that has a 512-byte sector size, you can use the following script:

```powershell
# Set parameters and select subscription
$subscriptionId = "<yourSubscriptionID>"
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$region = "<yourLocation>"
$zone = "<yourAvailabilityZone>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscriptionId

# Create the disk
$diskConfig = New-AzDiskConfig `
    -Location $region `
    -DiskSizeGB 8 `
    -DiskIOPSReadWrite 1000 `
    -DiskMBpsReadWrite 100 `
    -LogicalSectorSize 512 `
    -AccountType UltraSSD_LRS `
    -CreateOption Empty `
    -Zone $zone

New-AzDisk `
    -ResourceGroupName $rgName `
    -DiskName $diskName `
    -Disk $diskConfig

# Add disk to VM
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $rgName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $rgName
```

Use [`Get-AzDisk`](/powershell/module/az.compute/get-azdisk) to confirm that `ProvisioningState` is `Succeeded` and `LogicalSectorSize` is `512`:

```powershell
Get-AzDisk -ResourceGroupName '<yourResourceGroup>' -DiskName '<yourDiskName>' |
    Select-Object ProvisioningState, LogicalSectorSize
```
---
## Attach an Ultra Disk

### Additional limitations for regional Ultra Disks in regions with availability zones

When you attach a regional Ultra Disk to a regional VM in a region with availability zones, Azure might run a background copy to align the disk with the VM's availability zone and optimize latency. The copy can take up to 24 hours.

The following additional limitations apply during the background copy:

- You can't attach a nonzonal disk created from a snapshot, including an [instant access snapshot](disks-instant-access-snapshots.md), to a nonzonal VM in a region with availability zones until the copy finishes. To check the copy status, see [Performance impact of background copy](scripts/create-managed-disk-from-snapshot.md#performance-impact---background-copy-process).
- You can't resize the disk or change its customer-managed key.

Only one background copy can run on a nonzonal disk at a time. While a background copy is in progress, attaching the disk to a running nonzonal VM might fail. Restarting a stopped or deallocated nonzonal VM with the disk attached might also fail because the restart can trigger a second background copy.

# [Portal](#tab/azure-portal)

Alternatively, if your existing VM is in a region/availability zone that is capable of using Ultra Disks, you can make use of Ultra Disks without having to create a new VM. By enabling Ultra Disks on your existing VM, then attaching them as data disks. To enable Ultra Disk compatibility, you must stop the VM. After you stop the VM, you can enable compatibility, then restart the VM. Once compatibility is enabled, you can attach an Ultra Disk:

1. Navigate to your VM and stop it, wait for it to deallocate.
1. Once your VM has been deallocated, select **Disks**.
1. Select **Additional settings**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-disk-additional-settings.png" alt-text="Screenshot of the virtual machine Disks toolbar with Additional settings highlighted." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-disk-additional-settings.png":::

1. Select **Yes** for **Enable Ultra Disk compatibility**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/enable-ultra-disks-existing-vm.png" alt-text="Screenshot of the Ultra disk settings with Enable Ultra disk compatibility set to Yes." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/enable-ultra-disks-existing-vm.png":::

1. Select **Save**.
1. Select **Create and attach a new disk** and fill in a name for your new disk.
1. For **Storage type** select **Ultra Disk**.
1. Change the values of **Size (GiB)**, **Max IOPS**, and **Max throughput** to ones of your choice.
1. After you're returned to your disk's blade, select **Save**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-create-ultra-disk-existing-vm.png" alt-text="Screenshot of the virtual machine Disks pane with a 4-GiB Ultra Disk data disk configured for 120 IOPS and 25 MB/s throughput, and Save highlighted." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-create-ultra-disk-existing-vm.png":::

1. Start your VM again.

# [Azure CLI](#tab/azure-cli)

Alternatively, if your existing VM is in a region/availability zone that is capable of using Ultra Disks, you can make use of Ultra Disks without having to create a new VM.

### Enable Ultra Disk compatibility on an existing VM with Azure CLI

If your VM meets the requirements outlined in [GA scope and limitations](#ga-scope-and-limitations) and is in the [appropriate zone for your account](#determine-vm-size-and-region-availability), then you can enable Ultra Disk compatibility on your VM.

To enable Ultra Disk compatibility, you must stop the VM. After you stop the VM, you can enable compatibility, then restart the VM. Once compatibility is enabled, you can attach an Ultra Disk:

```azurecli
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"

az vm deallocate -n $vmName -g $rgName
az vm update -n $vmName -g $rgName --ultra-ssd-enabled true
az vm start -n $vmName -g $rgName
```

### Create an Ultra Disk with Azure CLI

Now that you have a VM that is capable of attaching Ultra Disks, you can create and attach an Ultra Disk to it.

```azurecli-interactive
subscriptionId="<yourSubscriptionID>"
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
region="<yourLocation>"
zone="<yourAvailabilityZone>"

# Create an Ultra Disk
az disk create \
--subscription $subscriptionId \
-n $diskName \
-g $rgName \
--size-gb 4 \
--location $region \
--zone $zone \
--sku UltraSSD_LRS \
--disk-iops-read-write 1000 \
--disk-mbps-read-write 50
```

### Attach the Ultra Disk with Azure CLI

```azurecli
subscriptionId="<yourSubscriptionID>"
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"

az vm disk attach -g $rgName --vm-name $vmName --disk $diskName --subscription $subscriptionId
```

Use `az vm show` to confirm that the Ultra Disk appears in the VM's data disks:

```azurecli
az vm show -g <yourResourceGroupName> -n <yourVMName> --query "storageProfile.dataDisks[].name"
```

# [Azure PowerShell](#tab/azure-powershell)

Alternatively, if your existing VM is in a region/availability zone that is capable of using Ultra Disks, you can make use of Ultra Disks without having to create a new VM.

### Enable Ultra Disk compatibility on an existing VM with Azure PowerShell

If your VM meets the requirements outlined in [GA scope and limitations](#ga-scope-and-limitations) and is in the [appropriate zone for your account](#determine-vm-size-and-region-availability), then you can enable Ultra Disk compatibility on your VM.

To enable Ultra Disk compatibility, you must stop the VM. After you stop the VM, you can enable compatibility, then restart the VM. Once compatibility is enabled, you can attach an Ultra Disk:

```powershell
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"

# Stop the VM
Stop-AzVM -Name $vmName -ResourceGroupName $rgName
# Enable Ultra Disk compatibility
$vm = Get-AzVM -name $vmName -ResourceGroupName $rgName
Update-AzVM -ResourceGroupName $rgName -VM $vm -UltraSSDEnabled $True
# Start the VM
Start-AzVM -Name $vmName -ResourceGroupName $rgName
```

### Create and attach an Ultra Disk with Azure PowerShell

Now that you have a VM that is capable of using Ultra Disks, you can create and attach an Ultra Disk to it:

```powershell
# Set parameters and select subscription
$subscriptionId = "<yourSubscriptionID>"
$rgName = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$region = "<yourLocation>"
$zone = "<yourAvailabilityZone>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscriptionId

# Create the disk
$diskConfig = New-AzDiskConfig `
    -Location $region `
    -DiskSizeGB 8 `
    -DiskIOPSReadWrite 1000 `
    -DiskMBpsReadWrite 100 `
    -AccountType UltraSSD_LRS `
    -CreateOption Empty `
    -zone $zone

New-AzDisk `
    -ResourceGroupName $rgName `
    -DiskName $diskName `
    -Disk $diskConfig

# Add disk to VM
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $rgName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $rgName
```

Use `Get-AzVM` to confirm that the Ultra Disk appears in the VM's data disks:

```powershell
(Get-AzVM -ResourceGroupName '<yourResourceGroup>' -Name '<yourVMName>').StorageProfile.DataDisks |
    Select-Object Name, Lun
```

---
## Adjust the performance of an Ultra Disk

# [Portal](#tab/azure-portal)

Ultra Disks offer a unique capability that allows you to adjust their performance. You can adjust the performance of an Ultra Disk four times within a 24 hour period.

1. Navigate to your VM and select **Disks**.
1. Select the Ultra Disk you'd like to modify the performance of.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-ultra-disk-to-modify.png" alt-text="Screenshot of the virtual machine Data disks list with the Ultra Disk named ultra-disk-name highlighted at LUN 0." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/select-ultra-disk-to-modify.png":::

1. Select **Size + performance** and then make your modifications.
1. Select **Save**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/modify-ultra-disk-performance.png" alt-text="Screenshot of the Ultra Disk Size + performance pane with custom disk size set to 32 GiB, disk IOPS set to 125, and disk throughput set to 25 MB/s." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/modify-ultra-disk-performance.png":::

# [Azure CLI](#tab/azure-cli)

Ultra Disks offer a unique capability that allows you to adjust their performance. You can adjust the performance of an Ultra Disk four times within a 24 hour period. The following command depicts how to use this feature:

```azurecli-interactive
subscriptionId="<yourSubscriptionID>"
rgName="<yourResourceGroupName>"
diskName="<yourDiskName>"

az disk update --subscription $subscriptionId --resource-group $rgName --name $diskName --disk-iops-read-write=5000 --disk-mbps-read-write=200
```

Use `az disk show` to confirm that `diskIOPSReadWrite` is `5000` and `diskMBpsReadWrite` is `200`:

```azurecli-interactive
az disk show -g <yourResourceGroupName> -n <yourDiskName> --query "{diskIOPSReadWrite:diskIOPSReadWrite,diskMBpsReadWrite:diskMBpsReadWrite}"
```

# [Azure PowerShell](#tab/azure-powershell)

Ultra Disks have a unique capability that allows you to adjust their performance. You can adjust the performance of an Ultra Disk four times within a 24 hour period. The following command is an example that adjusts the performance without having to detach the disk:

```powershell
$rgName = "<yourResourceGroup>"
$diskName = "<yourDiskName>"

$diskUpdateConfig = New-AzDiskUpdateConfig -DiskMBpsReadWrite 2000
Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName -DiskUpdate $diskUpdateConfig
```

Use `Get-AzDisk` to confirm that `DiskMBpsReadWrite` is `2000`:

```powershell
(Get-AzDisk -ResourceGroupName '<yourResourceGroup>' -DiskName '<yourDiskName>').DiskMBpsReadWrite
```
---

## Next steps

- [Use Azure Ultra Disks on Azure Kubernetes Service (preview)](/azure/aks/use-ultra-disks).
- [Migrate log disk to an Ultra Disk](/azure/azure-sql/virtual-machines/windows/storage-migrate-to-ultradisk).
- For more questions on Ultra Disks, see the [Ultra Disks](/azure/virtual-machines/faq-for-disks#ultra-disks) section of the FAQ.
