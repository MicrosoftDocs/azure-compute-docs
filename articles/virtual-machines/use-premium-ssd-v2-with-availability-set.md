---
title: Use Premium SSD v2 with VMs in availability set
description: Instructions on how to use Premium SSD v2 with VMs in availability set
author: vishalprayag
ms.author: rogarana 
ms.date: 05/12/2025
ms.topic: how-to
ms.service: azure-disk-storage
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell, innovation-engine
---
# Use premium SSD v2 with VMs in availability set

## Introduction

Premium SSD v2 managed disks are supported with Virtual Machine(VM)s in availability sets to enhance the high availability and resilience of your applications. When VMs using Premium SSD v2 are part of an Availability Set(AvSet), the platform ensures that their disks are automatically distributed across multiple storage Fault Domains (FDs). This distribution minimizes the risk of a single point of failure. 

:::image type="content" source="media/availability-set-alignment-setup.png" alt-text="Diagram Showing Availability Set with Managed Disk FD alignment Setup." lightbox="media/availability-set-alignment-setup.png":::

Availability sets have fault isolation for many possible failures, to minimize single points of failure and to offer high availability.  If there's a failure in one storage FD, only the VM instances with Premium SSD v2 disks on that specific FD is affected. The other VM instances, whose disks are placed on separate Fault Domains, remain unaffected and continue to operate normally. AvSets are susceptible to certain shared infrastructure failures, like datacenter network failures, physical hardware failures or power interruptions which can affect multiple fault domains. 

In, a production scenario with three VMs deployed in an AvSets using Premium SSD v2 with Fault Domain 3, VMs and disks are distributed across multiple fault domains. When a storage fault domain fails, only the VMs in that domain are affected. The other VMs continue running, which helps maintain application availability. 

When a Premium SSD v2 disk starts in one fault domain and is attached to a VM in another, the system triggers a background copy. This process moves the disk to match the VM’s fault domain, helping ensure consistent alignment between compute and storage for better reliability and availability. 

:::image type="content" source="media/availability-set-disk-move.png" alt-text="Diagram Showing Availability Set with Managed Disk FD alignment Disk Move." lightbox="media/availability-set-disk-move.png":::

For example, as illustrated in the accompanying diagram, when you attach a disk located in FD1 to a VM in FD1, the system automatically triggers a background copy of the disk. If you later detach the disk and attach it to a VM in FD2, the system will move the disk from FD1 to FD2. This process ensures alignment between compute and storage fault domains.  

## Regional availability

Premium SSD v2 support for VMs in an Availability Set is currently limited to the following regions that lack Availability Zone support: 

- Australia Southeast
- Canada East
- North Central US
- UK West
- West Central US
- West US

## Limitations

-  A subscription must be registered for the required feature to use Premium SSD v2 with VMs in Availability Sets in non-zonal regions. Follow the instructions [here](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#register-premium-ssd-v2-with-vms-in-availability-sets-in-supported-non-zonal-regions) to complete the registration.
- Only one background data copy can run per disk at a time. When attaching a disk to a VM in an Availability Set (AvSet), the system might start a background copy to align with the Fault Domain (FD). If you try to detach and reattach the disk while this move is in progress, the operation fails with an error. To prevent operation failure, wait until the move finishes, or set the [OptimizedForFrequentAttach](/dotnet/api/microsoft.azure.management.compute.models.diskupdate.optimizedforfrequentattach) property on the disk. This setting skips FD-alignment background copies for future attachments. For more information on OptimizedForFrequentAttach, follow the instructions [here](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#optimize-background-data-copy-of-the-disk).
- You can’t attach a disk created from a snapshot to VMs in an Availability Set while it’s still copying data in the background. Wait until the copy process finishes before attaching the disk. To check the status of background data copy from a snapshot, follow the instructions [here](/azure/virtual-machines/scripts/create-managed-disk-from-snapshot#performance-impact---background-copy-process).
- Disk size increase and changing customer-managed key (CMK) are not supported while a background data copy for Fault Domain alignment is in progress.

### [Azure CLI](#tab/CLI)
 
* Create a resource group:
 
```azurecli-interactive
az group create --name myResourceGroup --location myLocation 
```
* The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region:

```azurecli-interactive
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location,  MaximumFaultDomainCount:capabilities[0].value}' -o Table 
```

* Create the availability set:  
 
```azurecli-interactive
az vm availability-set create -n myAvSet -g myResourceGroup --platform-fault-domain-count 3 --platform-update-domain-count 20
```
 
> [!Note]
> The value for *platform-fault-domain-count* should be determined based on the number of available Storage Fault Domains in a given region. Please refer to Step 2 to check the available Fault Domains per region.
 
* Create a VM:
 
```azurecli-interactive
az vm create -n myVMname -g myResourceGroupName --availability-set myAvSetName --image Win2016Datacenter --count MyCount 
```
 
* Attach a new premium SSD v2 disk to existing VMs in an availability set
 
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
 
* The number of Storage Fault domains varies by region. The following command retrieves a list of fault domains per region:
 
```powershell
Get-AzComputeResourceSku | Where-Object {$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'} | Select-Object @{Name='Location'; Expression={$_.locationInfo[0].location}}, @{Name='MaximumFaultDomainCount'; Expression={$_.capabilities[0].value}} 
```

* Create the availability set:  
```powershell
New-AzAvailabilitySet -Name myAvSetName -ResourceGroupName myResourceGroup -Sku aligned -PlatformFaultDomainCount 3 -PlatformUpdateDomainCount 20 -Location myLocation  
 ```
> [!Note]
> The value for *platform-fault-domain-count* should be determined based on the number of available Storage Fault Domains in a given region. Please refer to Step 2 to check the available Fault Domains per region.
 
* Create a VM:
```powershell
New-AzVm 
ResourceGroupName $resourceGroupName 
Name $vmName 
Location $region  
Image $vmImage `
Size $vmSize 
AvailabilitySetName $AvSetName` 
Credential $credential
```

* Attach existing Premium SSD v2 disk to existing VMs in an Availability Set:
 
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroupName
```

### [Portal](#tab/Portal)
 
* Sign in to the [Azure portal](https://portal.azure.com/).
 
* Create an Availability Set with the 'Use managed disk' option set to 'Yes (Aligned)'
 
:::image type="content" source="media/create-an-availability-set.png" alt-text="Diagram showing the Azure portal screen to create an availability set." lightbox="media/create-an-availability-set.png":::
 
* Follow the default process for VM creation.
 
* On the **Basics** page, select a supported region and set availability options to Availability set.
 
* Select an Availability set.
 
:::image type="content" source="media/select-availability-set.png" alt-text="Diagram showing the Azure portal screen to select an availability set." lightbox="media/select-availability-set.png":::
 
* Complete the rest of the fields with inputs and navigate to the **Disks** page.
 
* Under **Data disks** select **Create and attach a new disk**.
 
:::image type="content" source="media/attach-a-new-disk.png" alt-text="Diagram showing the Azure portal screen to create and attach a new disk." lightbox="media/attach-a-new-disk.png":::
 
* Select the **Disk SKU** and select **Premium SSD v2**.
 
:::image type="content" source="media/select-disk-sku.png" alt-text="Diagram showing the Azure portal screen to select disk SKU." lightbox="media/select-disk-sku.png":::
 
* Select a logical sector of your choice to deploy a 4k or 512.
 
:::image type="content" source="media/select-logical-sector.png" alt-text="Diagram showing the Azure portal screen to select logical sector." lightbox="media/select-logical-sector.png":::
 
* Continued through the rest of the VM deployment, of your choice.
  
You deployed an Availability Set VM with a premium SSD v2.

---

## Register Premium SSD v2 with VMs in availability sets in supported non-zonal regions

The feature is region-specific and only available in regions that do not support Availability Zones.
If you're targeting a [supported region without zone support](/azure/virtual-machines/use-premium-ssd-v2-with-availability-set?tabs=CLI#regional-availability), ensure your subscription is registered for the required feature.

To proceed, register the feature manually:
- You can register it with:
  ```azurecli-interactive
  az feature registration create --namespace Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```
- Use the command below to verify if the feature is registered :
  ```azurecli-interactive
  az feature registration show --provider Microsoft.Compute --name PV2WithAVSetRegionWithoutZone 
  ```

## Optimize background data copy of the disk

### Change optimized-for-frequent-attach disk property

If your workload often moves disks between VMs in the same or different Availability Sets, turn on the `optimized-for-frequent-attach` setting to avoid delays. Setting this property to true prevents the system from triggering a background copy of the disk for FD alignment during reattachments. `optimized-for-frequent-attach` can be set when creating a new unattached disk or update it later for an existing disk. If the disk is currently attached to a VM, first detach the disk. Update the `optimized-for-frequent-attach` disk property and then reattach the disk to the VM. 

#### Instructions to set or update optimized-for-frequent-attach on disks

- To set the property while creating a new unattached disk:
 ```azurecli-interactive
 az disk create --name myDiskName --resource-group myResourceGroup --location myLocation --sku PremiumV2_LRS --size-gb myGB --optimized-for-frequent-attach true 
 ```
- To update the property on an existing unattached disk:
 ```azurecli-interactive
 az disk update --name myDiskName --resource-group myResourceGroup --set optimizedForFrequentAttach=true
 ```
