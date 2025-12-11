---
title: Use Premium SSD v2 with VMs in availability set
description: Learn how to deploy Premium SSD v2 disks with VMs in availability set
author: vishalprayag
ms.author: rogarana 
ms.date: 05/19/2025
ms.topic: how-to
ms.service: azure-disk-storage
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, innovation-engine
# Customer intent: As a cloud architect, I want to deploy Premium SSD v2 disks with VMs in an availability set so that I can ensure high availability and minimize the risk of downtime due to correlated failures.

---

# Use premium SSD v2 with VMs in availability set

## Introduction

Premium SSD v2 managed disks are supported with Azure Virtual Machines in availability sets to enhance the high availability of your applications. When VMs using Premium SSD v2 are part of an availability set, the platform ensures that their disks are automatically distributed across multiple storage fault domains. This distribution minimizes the risk of a single point of failure. 

:::image type="content" source="media/availability-set-alignment-setup.png" alt-text="Diagram showing availability set with managed disk fault domain alignment setup." lightbox="media/availability-set-alignment-setup.png":::

Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability.  If there's a failure in one storage fault domain, only the virtual machine (VM) instances with Premium SSD v2 disks on that specific fault domain are affected. Other VM instances, whose disks are placed on separate fault domains, remain unaffected and continue to operate normally. Availability sets are susceptible to certain shared infrastructure failures, like datacenter network failures, physical hardware failures or power interruptions which can affect multiple fault domains. 

When a Premium SSD v2 disk located in one fault domain is attached to a VM in another fault domain, the system triggers a background copy. This process moves the disk to match the VM’s fault domain, helping ensure consistent alignment between compute and storage for better reliability and availability. 

:::image type="content" source="media/availability-set-disk-move.png" alt-text="Diagram Showing Availability Set with Managed Disk Fault Domain alignment Disk Move." lightbox="media/availability-set-disk-move.png":::

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

-   Premium SSD v2 is supported for VMs in availability set in [select regions without availability zones](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#regional-availability).
-   You must register your subscription to use this feature. Follow the steps in [Register Premium SSD v2 with VMs in availability sets in regions without availability zones](#register-premium-ssd-v2-with-vms-in-availability-sets-in-supported-regions-without-availability-zones).
- Only one background data copy can run per disk at a time. When attaching a disk to a VM in an availability set, the system might start a background copy to align with the fault domain. If you try to detach and reattach the disk while this move is in progress, the operation fails with an error. To prevent operation failure, wait until the move finishes, or set the [OptimizedForFrequentAttach](/dotnet/api/microsoft.azure.management.compute.models.diskupdate.optimizedforfrequentattach) property on the disk. This setting skips fault domain-alignment background copies for future attachments. For more information on OptimizedForFrequentAttach, follow the instructions [Optimize background data copy of the disk](#optimize-background-data-copy-of-the-disk).
- You can’t attach a disk created from a snapshot to VMs in an availability set while it’s still copying data in the background. Wait until the copy process finishes before attaching the disk. To check the status of background data copy from a snapshot, follow the instructions [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot).
- Disk size increase and changing customer-managed key (CMK) are not supported while a background data copy for Fault Domain alignment is in progress.
- Premium SSD v2 managed disks have their own [separate set of limitations](/azure/virtual-machines/disks-deploy-premium-v2?tabs=azure-cli#limitations), as well.

## Register Premium SSD v2 with VMs in availability sets in supported regions without availability zones

The feature is only available in regions that do not support Availability Zones.
If you're targeting a [supported region without availability zones](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#regional-availability), ensure your subscription is registered for the required feature.

To proceed, register the feature manually:
- Use the following command to register the feature with your subscription:
  ```azurecli-interactive
  az feature registration create --namespace Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```
- Use the command below to verify if the feature is registered :
  ```azurecli-interactive
  az feature registration show --provider Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```

## Deploy a VM and a Premium SSD v2 disk within an availability set

### [Azure CLI](#tab/CLI)
 
* Create a resource group:
 
```azurecli-interactive
az group create --name myResourceGroup --location myLocation 
```
* The number of storage fault domains varies by region. The following command retrieves a list of maximum suported fault domains per region:

```azurecli-interactive
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location,  MaximumFaultDomainCount:capabilities[0].value}' -o Table 
```

* Create an availability set:  
 
```azurecli-interactive
az vm availability-set create -n myAvSet -g myResourceGroup --platform-fault-domain-count myFDCount --platform-update-domain-count myUDCount

## The value for platform-fault-domain-count should be determined based on the number of maximum supported fault domains in a given region. See previous step to check the available fault domains per region.
```
 

 
* Create a VM:
 
```azurecli-interactive
az vm create -n myVMname -g myResourceGroupName --availability-set myAvSetName --image Win2016Datacenter --count MyCount 
```
 
* Attach a new Premium SSD v2 disk to existing VMs in an availability set
 
```azurecli-interactive
az vm disk attach -g MyResourceGroupName --vm-name MyVMname --name MyDiskName --new --sku PremiumV2_LRS --size-gb MySize
```
 
* Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:
 
```azurecli-interactive
az vm disk attach -g MyResourceGroupName --vm-name MyVMname --disks MyDiskName
```
 

### [PowerShell](#tab/PowerShell)
 
* Create a resource group:
 
```powershell
New-AzResourceGroup -Name myResourceGroup -Location myLocation
```
 
* The number of storage fault domains varies by region. The following command retrieves a list of maximum supported fault domains per region:
 
```powershell
Get-AzComputeResourceSku | Where-Object {$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'} | Select-Object @{Name='Location'; Expression={$_.locationInfo[0].location}}, @{Name='MaximumFaultDomainCount'; Expression={$_.capabilities[0].value}}  
```

* Create the availability set:  
 ```powershell
New-AzAvailabilitySet -Name myAvSetName -ResourceGroupName myResourceGroup -Sku aligned -PlatformFaultDomainCount myFDCount -PlatformUpdateDomainCount myUDCount -Location myLocation  

## The value for PlatformUpdateDomainCount should be determined based on the number of maximum supported fault domains in a given region. See previous step to check the available fault domains per region.
```

 
* Create a VM:
```powershell
New-AzVm `
-ResourceGroupName $resourceGroupName ` 
-Name $vmName ` 
-Location $region `  
-Image $vmImage `
-Size $Size ` 
-AvailabilitySetName $AvSetName` 
-Credential $credential
```

* Attach a new Premium SSD v2 disk to existing VMs in an availability set: 
```powershell

$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName 
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -StorageAccountType PremiumV2_LRS  -Lun $lun 
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName 
```

* Attach existing Premium SSD v2 disk to existing VMs in an availability set:
 
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

### [Portal](#tab/Portal)
 
* Sign in to the [Azure portal](https://portal.azure.com/).
 
* Create an availability set with **Use managed disk** set to **Yes (Aligned)**.
 
:::image type="content" source="media/create-an-availability-set.png" alt-text="Diagram showing the Azure portal screen to create an availability set." lightbox="media/create-an-availability-set.png":::
 
* Follow the default process for VM creation.
 
* On the **Basics** page, select a supported region and set **Availability options** to **Availability set**.
 
* Select an Availability set.
 
:::image type="content" source="media/select-availability-set.png" alt-text="Diagram showing the Azure portal screen to select an availability set." lightbox="media/select-availability-set.png":::
 
* Complete the rest of the fields with inputs and navigate to the **Disks** page.
 
* Under **Data disks** select **Create and attach a new disk**.
 
:::image type="content" source="media/attach-a-new-disk.png" alt-text="Diagram showing the Azure portal screen to create and attach a new disk." lightbox="media/attach-a-new-disk.png":::
 
* Select the **Disk SKU** and select **Premium SSD v2**.
 
:::image type="content" source="media/select-disk-sku.png" alt-text="Diagram showing the Azure portal screen to select disk SKU." lightbox="media/select-disk-sku.png":::
 
* Select a logical sector of your choice to deploy a 4k or 512.
 
:::image type="content" source="media/select-logical-sector.png" alt-text="Diagram showing the Azure portal screen to select logical sector." lightbox="media/select-logical-sector.png":::
 
* Continued through the rest of the VM deployment.
  
You have now deployed a VM and a Premium SSD v2 within an availability set.

---

## Optimize background data copy of the disk

### Change optimized-for-frequent-attach disk property

If your workload often moves disks between VMs in the same or different availability sets, turn on the `optimized-for-frequent-attach` property. Setting this property to true prevents the system from triggering a background copy of the disk for fault domain alignment during reattachments. `optimized-for-frequent-attach` can be set when creating a new unattached disk or update it later for an existing disk. If the disk is currently attached to a VM, first detach the disk. Update the `optimized-for-frequent-attach` disk property and then reattach the disk to the VM.

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
