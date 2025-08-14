---
title: Azure support for Generation 2 VMs
description: Overview of Azure support for Generation 2 VMs
author: AjKundnani
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: how-to
ms.date: 03/04/2024
ms.author: ajkundna
# Customer intent: "As a cloud architect, I want to understand the benefits and limitations of Generation 2 VMs in Azure, so that I can make informed decisions when designing scalable and secure virtual machine solutions for my organization."
---

# Support for Generation 2 VMs on Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Support for Generation 2 virtual machines (VMs) is now available on Azure. You can't change a virtual machine's generation after you've created it, so review the considerations on this page before you choose a generation.

Generation 2 VMs support key features that aren't supported in Generation 1 VMs. These features include increased memory, Intel Software Guard Extensions (Intel SGX), and virtualized persistent memory (vPMEM). Generation 2 VMs running on-premises, have some features that aren't supported in Azure yet. For more information, see the [Features and capabilities](#features-and-capabilities) section.

Generation 2 VMs use the new UEFI-based boot architecture rather than the BIOS-based architecture used by Generation 1 VMs. Compared to Generation 1 VMs, Generation 2 VMs might have improved boot and installation times. For an overview of Generation 2 VMs and some of the differences between Generation 1 and Generation 2, see [Should I create a Generation 1 or 2 virtual machine in Hyper-V?](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).

## Generation 2 VM sizes

Azure now offers Generation 2 support for the following selected VM series:

| Type | Generation 2 supported size families  | Not supported size families
|:--- |:--- |:--- |
| [General purpose](./sizes/overview.md#general-purpose) | [B-family](./sizes/general-purpose/b-family.md), [D-family](./sizes/general-purpose/d-family.md) | [A-family](./sizes/general-purpose/a-family.md)
| [Compute optimized](./sizes/overview.md#compute-optimized) | [F-family](./sizes/compute-optimized/f-family.md), [Fx-family](./sizes/compute-optimized/fx-family.md)  | 
| [Memory optimized](./sizes/overview.md#memory-optimized) | [E-family](./sizes/memory-optimized/e-family.md), [Eb-family](./sizes/memory-optimized/eb-family.md), [M-family](./sizes/memory-optimized/m-family.md)    |
| [Storage optimized](./sizes/overview.md#storage-optimized) | [L-family](./sizes/storage-optimized/l-family.md)  |
| [GPU](./sizes/overview.md#gpu-accelerated) | [NC-family](./sizes/gpu-accelerated/nc-family.md), [ND-family](./sizes/gpu-accelerated/nv-family.md), [NV-family](./sizes/gpu-accelerated/nv-family.md) | [NC-series](nc-series.md), [NV-series](nv-series.md)
| [High Performance Compute](./sizes/overview.md#high-performance-compute) |[HBv2-series](./hbv2-series-overview.md), [HBv3-series](./hbv3-series-overview.md), [HBv4-series](./hbv4-series-overview.md), [HC-series](./hc-series-overview.md), [HX-series](./hx-series-overview.md)  | 

<sup>1</sup> Mv2-series, DC-series, NDv2-series, Msv2 and Mdsv2-series Medium Memory do not support Generation 1 VM images and only support a subset of Generation 2 images. Please see [Mv2-series documentation](mv2-series.md), [ND A100 v4-series](nda100-v4-series.md), [NDv2-series](ndv2-series.md), and [Msv2 and Mdsv2 Medium Memory Series](msv2-mdsv2-series.md) for details.

## Generation 2 VM images in Azure Marketplace

Generation 2 VMs support the following Marketplace images:

* Windows Server 2025, 2022, 2019, 2016, 2012 R2, 2012
* Windows 11 Pro, Windows 11 Enterprise
* Windows 10 Pro, Windows 10 Enterprise
* SUSE Linux Enterprise Server 15 SP3, SP2
* SUSE Linux Enterprise Server 12 SP4
* Ubuntu Server 22.04 LTS, 20.04 LTS, 18.04 LTS, 16.04 LTS 
* RHEL 9,5, 9.4, 9.3, 9.2, 9.1, 9.0, 8.10, 8.9, 8.8, 8.7, 8.6, 8.5, 8.4, 8.3, 8.2, 8.1, 8.0, 7.9, 7.8, 7.7, 7.6, 7.5, 7.4, 7.0
* Cent OS 8.4, 8.3, 8.2, 8.1, 8.0, 7.7, 7.6, 7.5, 7.4
* Oracle Linux 9.3, 9.2, 9.1, 9.0, 8.9, 8.8, 8.7, 8.6, 8.5, 8.4, 8.3, 8.2, 8.1, 7.9, 7.9, 7.8, 7.7

> [!NOTE]
> Specific Virtual machine sizes like Mv2-Series, DC-series, ND A100 v4-series, NDv2-series, Msv2 and Mdsv2-series may only support a subset of these images - please look at the relevant virtual machine size documentation for complete details.

## On-premises vs. Azure Generation 2 VMs

Azure doesn't currently support some of the features that on-premises Hyper-V supports for Generation 2 VMs.

| Generation 2 feature                | On-premises Hyper-V | Azure |
|-------------------------------------|---------------------|-------|
| Secure boot                         | :heavy_check_mark:  | With [trusted launch](trusted-launch.md)   |
| Shielded VM                         | :heavy_check_mark:  | :x:   |
| vTPM                                | :heavy_check_mark:  | With [trusted launch](trusted-launch.md)  |
| Virtualization-based security (VBS) | :heavy_check_mark:  | :heavy_check_mark:   |
| VHDX format                         | :heavy_check_mark:  | :x:   |

For more information, see [Trusted launch](trusted-launch.md).

## Features and capabilities

### Generation 1 vs. Generation 2 features

| Feature | Generation 1 | Generation 2 |
|---------|--------------|--------------|
| Boot             | PCAT                      | UEFI                               |
| Disk controllers | IDE                       | SCSI                               |
| VM sizes         | All VM sizes | [See available sizes](#generation-2-vm-sizes) |

### Generation 1 vs. Generation 2 capabilities

| Capability | Generation 1 | Generation 2 |
|------------|--------------|--------------|
| OS disk > 2 TB                    | :x:                | :heavy_check_mark: |
| Custom disk/image/swap OS         | :heavy_check_mark: | :heavy_check_mark: |
| Virtual machine scale set support | :heavy_check_mark: | :heavy_check_mark: |
| Azure Site Recovery               | :heavy_check_mark: | :heavy_check_mark: |
| Backup/restore                    | :heavy_check_mark: | :heavy_check_mark: |
| Azure Compute Gallery             | :heavy_check_mark: | :heavy_check_mark: |
| [Azure disk encryption](../virtual-machines/disk-encryption-overview.md)             | :heavy_check_mark: | :heavy_check_mark:                |
| [Server-side encryption](disk-encryption.md)            | :heavy_check_mark: | :heavy_check_mark: |


## Creating a Generation 2 VM

### Azure Resource Manager Template
To create a simple Windows Generation 2 VM, see [Create a Windows virtual machine from a Resource Manager template](./windows/ps-template.md)
To create a simple Linux Generation 2 VM, see [How to create a Linux virtual machine with Azure Resource Manager templates](./linux/create-ssh-secured-vm-from-template.md)

### Marketplace image

In the Azure portal or Azure CLI, you can create Generation 2 VMs from a Marketplace image that supports UEFI boot.

#### Azure portal

Below are the steps to create a Generation 2 (Gen2) VM in Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Virtual Machines**
3. Under **Services**, select **Virtual machines**.
4. In the **Virtual machines** page, select **Add**, and then select **Virtual machine**.
5. Under **Project details**, make sure the correct subscription is selected.
6. Under **Resource group**, select **Create new** and type a name for your resource group or select an existing resource group from the dropdown.
7. Under **Instance details**, type a name for the virtual machine name and choose a region
8. Under **Image**, select a Generation 2 image from the **Marketplace images to get started**
   > [!TIP]
   > If you don't see the Generation 2 version of the image you want in the drop-down, select **See all images** and then change the **Image Type** filter to **Gen 2**.
9. Select a VM size that supports Generation 2. See a list of [supported sizes](#generation-2-vm-sizes).
10. Fill in the **Administrator account** information and then **Inbound port rules**
11.	At the bottom of the page, select **Review + Create**
12.	On the **Create a virtual machine** page, you can see the details about the VM you are about to deploy. Once validation shows as passed, select **Create**.

#### PowerShell

You can also use PowerShell to create a VM by directly referencing the Generation 1 or Generation 2 SKU.

For example, use the following PowerShell cmdlet to get a list of the SKUs in the `WindowsServer` offer.

```powershell
Get-AzVMImageSku -Location westus2 -PublisherName MicrosoftWindowsServer -Offer WindowsServer
```

If you're creating a VM with Windows Server 2019 as the OS, then you can select a Generation 2 (UEFI) image which looks like this:

```powershell
2019-datacenter-gensecond
```
If you're creating a VM with Windows 10 as the OS, then you can select a Generation 2 (UEFI) image which looks like this:

```powershell
20H2-PRO-G2
```

See the [Features and capabilities](#features-and-capabilities) section for a current list of supported Marketplace images.

#### Azure CLI

Alternatively, you can use the Azure CLI to see any available Generation 2 images, listed by **Publisher**.

```azurecli
az vm image list --publisher Canonical --sku gen2 --output table --all
```

### Managed image or managed disk

You can create a Generation 2 VM from a managed image or managed disk in the same way you would create a Generation 1 VM.

### Virtual machine scale sets

You can also create Generation 2 VMs by using virtual machine scale sets. In the Azure CLI, use Azure scale sets to create Generation 2 VMs.

## Frequently asked questions

* **Are Generation 2 VMs available in all Azure regions?**  
    Yes. But not all [generation 2 VM sizes](#generation-2-vm-sizes) are available in every region. The availability of the Generation 2 VM depends on the availability of the VM size.

* **Is there a price difference between Generation 1 and Generation 2 VMs?**  
   No.

* **I have a .vhd file from my on-premises Generation 2 VM. Can I use that .vhd file to create a Generation 2 VM in Azure?**
  Yes, you can bring your Generation 2 .vhd file to Azure and use that to create a Generation 2 VM. Use the following steps to do so:
    1. Upload the .vhd to a storage account in the same region where you'd like to create your VM.
    1. Create a managed disk from the .vhd file. Set the Hyper-V Generation property to V2. The following PowerShell commands set Hyper-V Generation property when creating managed disk.

        ```powershell
        $sourceUri = 'https://xyzstorage.blob.core.windows.net/vhd/abcd.vhd'. #<Provide location to your uploaded .vhd file>
        $osDiskName = 'gen2Diskfrmgenvhd'  #<Provide a name for your disk>
        $diskconfig = New-AzDiskConfig -Location '<location>' -DiskSizeGB 127 -AccountType Standard_LRS -OsType Windows -HyperVGeneration "V2" -SourceUri $sourceUri -CreateOption 'Import'
        New-AzDisk -DiskName $osDiskName -ResourceGroupName '<Your Resource Group>' -Disk $diskconfig
        ```

    1. Once the disk is available, create a VM by attaching this disk. The VM created will be a Generation 2 VM.
    When the Generation 2 VM is created, you can optionally generalize the image of this VM. By generalizing the image, you can use it to create multiple VMs.

* **How do I increase the OS disk size?**  

  OS disks larger than 2 TiB are new to Generation 2 VMs. By default, OS disks are smaller than 2 TiB for Generation 2 VMs. You can increase the disk size up to a recommended maximum of 4 TiB. Use the Azure CLI or the Azure portal to increase the OS disk size. For information about how to expand disks programmatically, see **Resize a disk** for [Windows](./windows/expand-os-disk.md) or [Linux](./linux/resize-os-disk-gpt-partition.md).

  To increase the OS disk size from the Azure portal:

  1. In the Azure portal, go to the VM properties page.
  1. To shut down and deallocate the VM, select the **Stop** button.
  1. In the **Disks** section, select the OS disk you want to increase.
  1. In the **Disks** section, select **Configuration**, and update the **Size** to the value you want.
  1. Go back to the VM properties page and **Start** the VM.
  
  You might see a warning for OS disks larger than 2 TiB. The warning doesn't apply to Generation 2 VMs. However, OS disk sizes larger than 4 TiB are not supported.

* **Do Generation 2 VMs support accelerated networking?**  
    Yes. For more information, see [Create a VM with accelerated networking](/azure/virtual-network/create-vm-accelerated-networking-cli).

* **Do Generation 2 VMs support Secure Boot or vTPM in Azure?**
    Both vTPM and Secure Boot are features of trusted launch for Generation 2 VMs. For more information, see [Trusted launch](trusted-launch.md).
    
* **Is VHDX supported on Generation 2?**  
    No, Generation 2 VMs on Azure support only VHD.

* **Do Generation 2 VMs support Azure Ultra Disk Storage?**  
    Yes.

* **Can I migrate a VM from Generation 1 to Generation 2?**  
    Azure Virtual Machines supports upgrading Generation 1 virtual machines (VM) to Generation 2 by upgrading to the [trusted launch security type](trusted-launch-existing-vm-gen-1.md). 

* **Why is my VM size not enabled in the size selector when I try to create a Generation 2 VM?**

    This may be solved by doing the following:

    1. Verify that the **VM Generation** property is set to **Gen 2**.
    1. Verify you are searching for a [VM size which supports Generation 2 VMs](#generation-2-vm-sizes).

## Next steps

Learn more about the [trusted launch](trusted-launch-portal.md) with Generation 2 VMs.

Learn about [Generation 2 virtual machines in Hyper-V](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).
