---
title: Ephemeral OS disks
description: Learn more about ephemeral OS disks for Azure VMs.
author: viveksingla08
ms.service: azure-virtual-machines
ms.custom:
ms.topic: how-to
ms.date: 07/23/2020
ms.author: viveksingla
ms.subservice: disks
---

# Ephemeral OS disks for Azure VMs

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

> [!NOTE]
> NVMe disk placement for Ephemeral OS disks is now Generally Available (GA). Customers can use Ephemeral OS disks for production workloads on supported v6 VM series.

Ephemeral OS disks are created on the local virtual machine (VM) storage and not saved to the remote Azure Storage. Ephemeral OS disks are ideal for stateless workloads, where applications can tolerate individual VM failures but are sensitive to VM deployment times or the reimaging of individual VM instances. With Ephemeral OS disk, you get lower read/write latency to the OS disk and faster VM reimage.

The key features of ephemeral disks are:

- Ephemeral OS disks are supported on VM sizes with local SSD storage, including Premium SSD, NVMe, and temp disks. The performance and reliability of the ephemeral OS disk are directly tied to the underlying local SSD storage of the VM. For best results, select VM sizes that offer Premium SSD or NVMe-based local storage.
- Designed for stateless applications.
- Supported on all images - Marketplace, custom images, and [Azure Compute Gallery](./shared-image-galleries.md) (formerly known as Shared Image Gallery).
- Provides fast reimage to reset virtual machines (VMs) and scale set instances to their original boot state.
- Offers lower latency, similar to a temporary disk.
- Supports Premium SSD & Standard SSD for higher SLA
- Supported in all Azure regions.

Key differences between persistent and ephemeral OS disks:

|   | Persistent OS Disk | Ephemeral OS Disk |
|---|---|---|
| **Size limit for OS disk** | 4* TiB | Cache, temp, or NVMe disk size for the VM size or 2,040 GiB, whichever is smaller. For the **cache, temp or NVMe size in GiB**, see [DSv3](sizes-general.md), [Esv3](sizes-memory.md), [M](sizes-memory.md), [FS](sizes-compute.md), and [GS](sizes-previous-gen.md#gs-series) |
| **VM sizes supported** | All | VM sizes with local storage such as D(a)dsv4, D(a)dsv5, D(a)dsv6, FXmdsv2, E(a)dsv5, L(a)sv3, etc.  |
| **Disk type support**| Managed and unmanaged OS disk| Managed OS disk only|
| **Region support**| All regions| All regions|
| **Data persistence**| OS disk data written to OS disk are stored in Azure Storage| Data written to OS disk is stored on local VM storage and isn't persisted to Azure Storage. |
| **Stop-deallocated state**| VMs and scale set instances can be stop-deallocated and restarted from the stop-deallocated state | Not Supported |
| **Specialized OS disk support** | Yes| No|
| **OS disk resize**| Supported during VM creation and after VM is stop-deallocated| Supported during VM creation only|
| **Resizing to a new VM size**| OS disk data is preserved| Data on the OS disk is deleted, OS is reprovisioned |
| **Redeploy** | OS disk data is preserved | Data on the OS disk is deleted, OS is reprovisioned |
| **Stop/ Start of VM** | OS disk data is preserved | Not Supported |
| **Page file placement**| For Windows, page file is stored on the temp disk| For Windows, page file is stored on the OS disk (for cache placement, Temp disk placement & NVMe disk placement).|
| **Maintenance of VM/VMSS using [healing](understand-vm-reboots.md#unexpected-downtime)** | OS disk data is preserved | OS disk data isn't preserved  |
| **Maintenance of VM/VMSS using [Live Migration](maintenance-and-updates.md#live-migration)** | OS disk data is preserved | OS disk data is preserved  |

\* 4 TiB is the maximum supported OS disk size for managed (persistent) disks. However, many OS disks are partitioned with master boot record (MBR) by default and are limited to 2 TiB. For details, see [OS disk](managed-disks-overview.md#os-disk).

## Placement options for Ephemeral OS disks

Ephemeral OS Disk utilizes local storage within the VM. Since different VMs have different types of local storage (cache disk, temp disk, and NVMe disk), the placement option defines where the Ephemeral OS Disk is stored. Placement option however doesn't impact the performance or cost of Ephemeral OS disk. Its performance is dependent upon the VM's local storage. Depending upon the VM type, we offer three different types of placement:
- **NVMe Disk Placement (Generally Available)**  - NVMe disk placement type is now generally available (GA) on the latest generation v6 VM series onwards like Dadsv6, Ddsv6, Dpdsv6, etc.
- **Temp Disk Placement (also known as Resource Disk Placement)**  - Temp disk placement type is available on VMs with Temp disk like Dadsv5, Ddsv5, etc.
- **Cache Disk Placement**  - Cache disk placement type is available on older VMs that had cache disk like Dsv2, Dsv3, etc.

[DiffDiskPlacement](/rest/api/compute/virtualmachines/list#diffdiskplacement) is the property that can be used to specify where you want to place the Ephemeral OS disk. By default, Azure will pick up the right placement type, depending on the VM SKU. The customers are recommended to use the latest VM series (v5/v6) with either Temp Disk or NVMe Disk placement.

## Size requirements

You can choose to deploy Ephemeral OS Disk on NVMe disk, temp disk, or cache on the VM.
The image OS disk’s size should be less than or equal to the NVMe/temp/cache size of the VM size chosen.

For **OS cache placement**: Standard Windows Server images from the marketplace are about 127 GiB, which means that you need a VM size that has a cache equal to or larger than 127 GiB. The Standard_DS3_v2 has a cache size of 127 GiB, which is large enough. In this case, the Standard_DS3_v2 is the smallest size in the DSv2 series that you can use with this image.

For **Temp disk placement**: Standard Ubuntu server image from marketplace is about 30 GiB. To enable Ephemeral OS disk on temp, the temp disk size must be equal to or larger than 30 GiB. Standard_B4ms has a temp size of 32 GiB, which can fit the 30-GiB OS disk. Upon creation of the VM, the temp disk space would be 2 GiB.

For **NVMe disk placement (GA)**: Standard Ubuntu server image from marketplace is about 30 GiB. To enable Ephemeral OS disk on NVMe, the NVMe disk size must be equal to or larger than 30 GiB. Standard_D2ads_v6 has a Nvme disk size of 110 GiB, which can easily fit the 30-GiB OS disk. However, Ephemeral OS disk occupies the entire NVMe disk and there's no NVMe disk space given back. One way to maximize the use of NVMe disk is by maximizing the OS disk Size property to 110 GiB.


> [!IMPORTANT]
> If opting for temp disk placement the Final Temp disk size = (Initial temp disk size - OS image size).
> 
> If opting for NVMe disk placement (GA), Final NVMe Disk size = (Total no. of NVMe disks - NVMe Disks used for OS) * Size of each NVMe disk. Where NVMe Disks used for OS is the minimum number of disks required for OS disk depending on the size of OS disk and the size of each NVMe disk.

If Ephemeral OS disk is using **Temp Disk Placement**, it shares the IOPS(input/output operations per second) with temp disk. If Ephemeral OS disk is using **NVMe Disk Placement**, it provides the IOPS(input/output operations per second) of NVMe disks being used.

Basic Linux and Windows Server images in the Marketplace that are denoted with `[smallsize]` tend to be around 30 GiB and can use most of the available VM sizes.

> [!NOTE]
>
> Ephemeral disks aren't accessible through the portal. You receive a "Resource not Found" or "404" error when accessing the ephemeral disk which is expected.
>

## Unsupported features

- VM Image Capture
- Disk snapshots
- Azure Disk Encryption
- Azure Backup
- Azure Site Recovery
- OS Disk Swaps

## Trusted Launch for Ephemeral OS disks

Ephemeral OS disks can be created with Trusted launch. All regions are supported for Trusted Launch; not all virtual machines sizes are supported. Check [Virtual machines sizes supported](trusted-launch.md#virtual-machines-sizes) for supported sizes.
VM guest state (VMGS) is specific to trusted launch VMs. It's an Azure-managed blob and contains the unified extensible firmware interface (UEFI) secure boot signature databases and other security information. VMs using trusted launch by default reserve **1 GiB** from the **OS cache** or **temp disk** or **Nvme Disk** based on the chosen placement option for VMGS. The lifecycle of the VMGS blob is tied to that of the OS Disk.

For example, If you try to create a Trusted launch Ephemeral OS disk VM using OS image of size 75 GiB with VM size [Standard_D2ads_v5](.\sizes\general-purpose\dadsv5-series.md) using temp disk placement you would get an error as
**"OS disk of Ephemeral VM with size greater than 74 GB is not allowed for VM size Standard_Dads_v5 when the DiffDiskPlacement is ResourceDisk."**
This error occurs because the temp disk for [Standard_D2ads_v5](.\sizes\general-purpose\dadsv5-series.md) is 75 GiB, and 1 GiB is reserved for VMGS when using trusted launch.
For the same example, if you create a standard Ephemeral OS disk VM you wouldn't get any errors and it would be a successful operation.

> [!IMPORTANT]
>
> If you use ephemeral disks with Trusted Launch VMs, any keys or secrets that the vTPM generates or seals after the VM is created might not be saved. As a result, these keys and secrets could be lost during actions such as reimaging or service healing events.
>
For more information on [how to deploy a trusted launch VM](trusted-launch-portal.md)

## Confidential VMs using Ephemeral OS disks

AMD-based Confidential VMs cater to high security and confidentiality requirements of customers. These VMs provide a strong, hardware-enforced boundary to help meet your security needs. There are limitations to use Confidential VMs. Check the [region](/azure/confidential-computing/confidential-vm-overview#regions), [size](/azure/confidential-computing/confidential-vm-overview#size-support), and [OS supported](/azure/confidential-computing/confidential-vm-overview#os-support) limitations for confidential VMs.
Virtual machine guest state (VMGS) blob contains the security information of the confidential VM.
Confidential VMs using Ephemeral OS disks by default **1 GiB** from the **OS cache** or **temp storage** based on the chosen placement option is reserved for VMGS. The lifecycle of the VMGS blob is tied to that of the OS Disk.
> [!IMPORTANT]
>
> When choosing a confidential VM with full OS disk encryption before VM deployment that uses a customer-managed key (CMK). [Updating a CMK key version](/azure/storage/common/customer-managed-keys-overview#update-the-key-version) or [key rotation](/azure/key-vault/keys/how-to-configure-key-rotation) isn't supported with Ephemeral OS disk. Confidential VMs using Ephemeral OS disks need to be deleted before updating or rotating the keys and can be re-created later.
>
For more information on [confidential VM](/azure/confidential-computing/confidential-vm-overview)

## Customer Managed key

You can choose to use customer managed keys or platform managed keys when you enable end-to-end encryption for VMs using Ephemeral OS disk. Currently this option is available only via [PowerShell](./windows/disks-enable-customer-managed-keys-powershell.md), [CLI](./linux/disks-enable-customer-managed-keys-cli.md), and SDK in all regions.

> [!IMPORTANT]
>
> [Updating a CMK key version](/azure/storage/common/customer-managed-keys-overview#update-the-key-version) or [key rotation](/azure/key-vault/keys/how-to-configure-key-rotation) of customer managed key isn't supported with Ephemeral OS disk. VMs using Ephemeral OS disks need to be deleted before updating or rotating the keys and can be re-created later.
>
For more information on [Encryption at host](./disk-encryption.md)

## SSD storage account support for Ephemeral OS disks

SSD support is a new option that allows customers to choose the type of base disk that is used for the ephemeral OS disk. Previously, the base disk could only be Standard HDD. Now, customers can choose between the three types of disks: Standard HDD(Standard_LRS), Standard SSD (StandardSSD_LRS) or Premium SSD (Premium_LRS). By utilizing SSD with Ephemeral OS disk, customers can benefit from the following enhancements:

- **Enhanced SLA**: VMs created with Premium SSD provide higher SLA than VMs created with Standard HDD. Customers can enhance [SLA](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services) for their Ephemeral VMs by choosing Premium SSD as base disk.
- **Improved performance**: By choosing Premium SSD as the base disk, customers can enhance the disk read performance of their VMs. While most writes occur on the local temp disk, some reads are performed from managed disks. Premium SSD disks provide 8-10 times higher IOPS than Standard HDD. 

## Next steps

Create a VM with ephemeral OS disk using [Azure Portal/CLI/PowerShell/ARM template](ephemeral-os-disks-deploy.md).
Check out the [frequently asked questions on ephemeral os disk](ephemeral-os-disks-faq.md).
