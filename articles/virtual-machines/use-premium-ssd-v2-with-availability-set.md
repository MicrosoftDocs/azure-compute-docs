---
title: Use Premium SSD v2 with VMs in an availability set
description: Learn how to deploy Premium SSD v2 disks with VMs in an availability set.
author: vishalprayag
ms.author: rogarana 
ms.date: 09/21/2026
ms.topic: how-to
ms.service: azure-disk-storage
ai-usage: ai-assisted
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, innovation-engine
# Customer intent: As a cloud architect, I want to deploy Premium SSD v2 disks with VMs in an availability set so that I can ensure high availability and minimize the risk of downtime due to correlated failures.

---

# Use Premium SSD v2 with VMs in an availability set

## How Premium SSD v2 aligns availability set fault domains

Premium SSD v2 managed disks are supported with Azure Virtual Machines in availability sets to enhance the high availability of your applications. When VMs using Premium SSD v2 are part of an availability set, the platform ensures that their disks are automatically distributed across multiple storage fault domains. This distribution minimizes the risk of a single point of failure. 

:::image type="content" source="media/availability-set-alignment-setup.png" alt-text="Diagram of three VMs and Premium SSD v2 disks aligned across three compute and storage fault domains." lightbox="media/availability-set-alignment-setup.png":::

Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability.  If there's a failure in one storage fault domain, only the virtual machine (VM) instances with Premium SSD v2 disks on that specific fault domain are affected. Other VM instances, whose disks are placed on separate fault domains, remain unaffected and continue to operate normally. Availability sets are susceptible to certain shared infrastructure failures, like datacenter network failures, physical hardware failures or power interruptions which can affect multiple fault domains. 

When a Premium SSD v2 located in one fault domain is attached to a VM in another fault domain, the system triggers a background copy. This process moves the disk to match the VM’s fault domain, helping ensure consistent alignment between compute and storage for better reliability and availability. 

:::image type="content" source="media/availability-set-disk-move.png" alt-text="Diagram of a Premium SSD v2 disk moving from storage fault domain 1 to storage fault domain 2 after attachment to a VM in compute fault domain 2." lightbox="media/availability-set-disk-move.png":::

As the previous diagram illustrates, if a disk located in fault domain 1 is attached to a VM in fault domain 1 and is later detached and attached to a VM in fault domain 2, the system automatically triggers a background copy of the disk to move it from fault domain 1 to fault domain 2 for compute and storage fault domain alignment. This background move process can take up to 24 hours to complete.    

## Regional availability

Premium SSD v2 support for VMs in an availability set is currently limited to the following regions that lack availability zone support: 

- Australia Southeast
- Canada East
- North Central US
- UK West
- West Central US
- West US

## Limitations

- Premium SSD v2 supports VMs in an availability set in [select regions without availability zones](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#regional-availability).
- You must register your subscription to use this feature. Follow the steps in [Register Premium SSD v2 for availability sets in regions without availability zones](#register-premium-ssd-v2-for-availability-sets-in-regions-without-availability-zones).
- Only one background data copy can run per disk at a time. When attaching a disk to a VM in an availability set, the system might start a background copy to align with the fault domain. If you try to detach and reattach the disk while this move is in progress, the operation fails with an error. To prevent operation failure, wait until the move finishes, or set the [OptimizedForFrequentAttach](/dotnet/api/microsoft.azure.management.compute.models.diskupdate.optimizedforfrequentattach) property on the disk. This setting skips fault domain-alignment background copies for future attachments. For more information on OptimizedForFrequentAttach, follow the instructions [Optimize background data copy of the disk](#optimize-background-data-copy-of-the-disk).
- You can’t attach a disk created from a snapshot to VMs in an availability set while it’s still copying data in the background. Wait until the copy process finishes before attaching the disk. To check the status of background data copy from a snapshot, follow the instructions [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot).
- Disk size increase and changing customer-managed key aren't supported while a background data copy for fault domain alignment is in progress.
- Premium SSD v2 managed disks have their own [separate set of limitations](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli#limitations), as well.

## Register Premium SSD v2 for availability sets in regions without availability zones

This feature is only available in regions that don't support availability zones.
If you're targeting a [supported region without availability zones](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#regional-availability), ensure your subscription is registered for the required feature.

To proceed, register the feature manually:
- Use the following command to register the feature with your subscription:
  ```azurecli-interactive
  az feature registration create --namespace Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```
- Use the following command to verify whether the feature is registered:
  ```azurecli-interactive
  az feature registration show --provider Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```

## Deploy a VM and a Premium SSD v2 within an availability set

### [Azure CLI](#tab/CLI)
 
* Create a resource group:
 
```azurecli-interactive
rgName="myResourceGroup"
location="westus"

az group create --name $rgName --location $location
```
* The number of storage fault domains varies by region. The following command retrieves a list of maximum supported fault domains per region:

```azurecli-interactive
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location,  MaximumFaultDomainCount:capabilities[0].value}' -o Table 
```

* Create an availability set. Set `faultDomainCount` to the `MaximumFaultDomainCount` value returned for your region in the previous step:
 
```azurecli-interactive
rgName="myResourceGroup"
availabilitySetName="myAvailabilitySet"
faultDomainCount=2
updateDomainCount=5

az vm availability-set create -n $availabilitySetName -g $rgName --platform-fault-domain-count $faultDomainCount --platform-update-domain-count $updateDomainCount
```
 

 
* Create a VM:
 
```azurecli-interactive
rgName="myResourceGroup"
availabilitySetName="myAvailabilitySet"
vmName="myVM"
vmCount=1

az vm create -n $vmName -g $rgName --availability-set $availabilitySetName --image Win2016Datacenter --count $vmCount
```
 
* Attach a new Premium SSD v2 to existing VMs in an availability set
 
```azurecli-interactive
rgName="myResourceGroup"
vmName="myVM"
diskName="myDataDisk"
diskSizeGiB=128

az vm disk attach -g $rgName --vm-name $vmName --name $diskName --new --sku PremiumV2_LRS --size-gb $diskSizeGiB
```
 
* Attach an existing Premium SSD v2 disk to existing VMs in an availability set:
 
```azurecli-interactive
rgName="myResourceGroup"
vmName="myVM"
diskName="myDataDisk"

az vm disk attach -g $rgName --vm-name $vmName --disks $diskName
```
 

### [PowerShell](#tab/PowerShell)
 
* Create a resource group:
 
```powershell
$resourceGroupName = "myResourceGroup"
$location = "West US"

New-AzResourceGroup -Name $resourceGroupName -Location $location
```
 
* The number of storage fault domains varies by region. The following command retrieves a list of maximum supported fault domains per region:
 
```powershell
Get-AzComputeResourceSku | Where-Object {$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'} | Select-Object @{Name='Location'; Expression={$_.locationInfo[0].location}}, @{Name='MaximumFaultDomainCount'; Expression={$_.capabilities[0].value}}
```

* Create the availability set. Set `PlatformFaultDomainCount` to the `MaximumFaultDomainCount` value returned for your region in the previous step:

```powershell
$availabilitySetName = "myAvailabilitySet"
$resourceGroupName = "myResourceGroup"
$location = "West US"
$faultDomainCount = 2
$updateDomainCount = 5

New-AzAvailabilitySet -Name $availabilitySetName -ResourceGroupName $resourceGroupName -Sku aligned -PlatformFaultDomainCount $faultDomainCount -PlatformUpdateDomainCount $updateDomainCount -Location $location
```

 
* Create a VM:
```powershell
$resourceGroupName = "myResourceGroup"
$vmName = "myVM"
$location = "West US"
$vmImage = "Win2016Datacenter"
$vmSize = "Standard_D4s_v3"
$availabilitySetName = "myAvailabilitySet"
$credential = Get-Credential

New-AzVm `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName `
  -Location $location `
  -Image $vmImage `
  -Size $vmSize `
  -AvailabilitySetName $availabilitySetName `
  -Credential $credential
```

* Attach a new Premium SSD v2 to existing VMs in an availability set: 
```powershell
$resourceGroupName = "myResourceGroup"
$vmName = "myVM"
$diskName = "myDataDisk"
$diskSizeInGiB = 128
$lun = 0

$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -DiskSizeInGB $diskSizeInGiB -StorageAccountType PremiumV2_LRS -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

* Attach existing Premium SSD v2 to existing VMs in an availability set:
 
```powershell
$resourceGroupName = "myResourceGroup"
$vmName = "myVM"
$diskName = "myDataDisk"
$lun = 0

$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

### [Portal](#tab/Portal)
 
* Sign in to the [Azure portal](https://portal.azure.com/).
 
* Create an availability set with **Use managed disk** set to **Yes (Aligned)**.
 
:::image type="content" source="media/create-an-availability-set.png" alt-text="Screenshot of the Create availability set pane with North Central US, two fault domains, 20 update domains, and Use managed disks set to Yes (Aligned)." lightbox="media/create-an-availability-set.png":::
 
* Follow the default process for VM creation.
 
* On the **Basics** page, select a supported region and set **Availability options** to **Availability set**.
 
* Select an availability set.
 
:::image type="content" source="media/select-availability-set.png" alt-text="Screenshot of the Create a virtual machine Basics pane with Availability options set to Availability set and AvSet_w_Pv2 selected." lightbox="media/select-availability-set.png":::
 
* Complete the rest of the fields with inputs and navigate to the **Disks** page.
 
* Under **Data disks** select **Create and attach a new disk**.
 
:::image type="content" source="media/attach-a-new-disk.png" alt-text="Screenshot of the Data disks section for Avset-Vm1 with Create and attach a new disk available." lightbox="media/attach-a-new-disk.png":::
 
* Select the **Disk SKU** and select **Premium SSD v2**.
 
:::image type="content" source="media/select-disk-sku.png" alt-text="Screenshot of the Storage type list with Premium SSD v2 selected." lightbox="media/select-disk-sku.png":::
 
* Select **4096** or **512** for **Logical sector size (bytes)**.
 
:::image type="content" source="media/select-logical-sector.png" alt-text="Screenshot of disk settings with Premium SSD v2, a 4-GiB disk size, and logical sector size set to 4,096 bytes." lightbox="media/select-logical-sector.png":::
 
* Continue through the rest of the VM deployment.
  
You have now deployed a VM and a Premium SSD v2 within an availability set.

---

## Optimize background data copy of the disk

### Change optimized-for-frequent-attach disk property

If your workload often moves disks between VMs in the same or different availability sets, turn on the `optimized-for-frequent-attach` property. Setting this property to true prevents the system from triggering a background copy of the disk for fault domain alignment during reattachments. You can set `optimized-for-frequent-attach` when you create a new unattached disk or update it later for an existing disk. If the disk is currently attached to a VM, first detach the disk. Update the `optimized-for-frequent-attach` disk property, and then reattach the disk to the VM.

To set the property while creating a new unattached disk:

 ```azurecli-interactive
 az disk create --name myDiskName --resource-group myResourceGroup --location myLocation --sku PremiumV2_LRS --size-gb myGB --optimized-for-frequent-attach true 
 ```

To update the property on an existing unattached disk:

 ```azurecli-interactive
 az disk update --name myDiskName --resource-group myResourceGroup --set optimizedForFrequentAttach=true
 ```

## Next steps

- [Deploy a Premium SSD v2](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli)
- [Availability sets overview](/azure/virtual-machines/availability-set-overview)
- [Best practices for achieving high availability with Azure virtual machines and managed disks](/azure/virtual-machines/disks-high-availability)
