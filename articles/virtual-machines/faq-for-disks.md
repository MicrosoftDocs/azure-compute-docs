---
title: "Frequently asked questions about Azure IaaS VM disks and Premium SSD managed disks"
description: "Frequently asked questions about Azure IaaS VM disks and Premium SSD managed disks"
author: roygara
ms.service: azure-disk-storage
ms.topic: faq
ms.date: 07/15/2026
ms.author: rogarana
ms.custom: references_regions
---

# Frequently asked questions about Azure IaaS VM disks and Premium SSD managed disks

This article answers some frequently asked questions about Azure managed disks and Azure Premium SSDs.

## Managed disks

### If I create a standard managed disk from an existing VHD that's 80 GiB, how much will that cost me?

A standard managed disk created from an 80-GiB VHD is treated as the next available standard disk size, which is an S10 disk. You're charged according to the S10 disk pricing. For more information, see the [pricing page](https://azure.microsoft.com/pricing/details/managed-disks/).

### For a standard managed disk, will I be charged for the actual size of the data on the disk or for the provisioned capacity of the disk?

You're charged based on the provisioned capacity of the disk. For more information, see the [pricing page](https://azure.microsoft.com/pricing/details/managed-disks/).

### Can I use a VHD file in an Azure storage account to create a managed disk with a different subscription?

Yes.

### Can I use a VHD file in an Azure storage account to create a managed disk in a different region?

No.

### Are there limits on how many managed disks I can have?

Yes. The maximum limit is 50,000 managed disks per region and per disk type for a subscription.

### Can VMs in an availability set include a mix of managed and unmanaged disks?

No. VMs in an availability set can't mix disk management models. Unmanaged disks are retired, so use managed disks for all VMs in the availability set.

### Can I create an empty managed disk?

Yes. You can create an empty disk. You can create a managed disk independently of a VM, for example, without attaching it to a VM.

### What is the supported fault domain count for an availability set that uses managed disks?

Depending on the region where the availability set that uses managed disks is located, the supported fault domain count is two or three.

### What kind of Azure role-based access control support is available for managed disks?

Managed disks supports three key default roles:

* Owner: Can manage everything, including access
* Contributor: Can manage everything except access
* Reader: Can view everything, but can't make changes

### Can I copy or export a managed disk to a private storage account?

You can generate a read-only shared access signature (SAS) URI for the managed disk and use it to copy the contents to a private storage account or on-premises storage. You can use the SAS URI by using the Azure portal, Azure PowerShell, the Azure CLI, or [AzCopy](/azure/storage/common/storage-use-azcopy-v10).

### Can I create a copy of my managed disk?

You can take a snapshot of a managed disk and then use the snapshot to create another managed disk. You can also create a new managed disk from an existing managed disk.

### Are unmanaged disks supported?

Unmanaged disks were fully retired on [March 31, 2026](unmanaged-disks-deprecation.md) (extended from September 30, 2025). If you still have workloads that use unmanaged disks, migrate them to managed disks as soon as possible. This retirement affects page blobs only when they're attached directly to VMs as virtual disks. Standalone page blobs accessed through Storage REST APIs aren't affected. See migration guidance for [Windows](windows/convert-unmanaged-to-managed-disks.md) and [Linux](linux/convert-unmanaged-to-managed-disks.md) VMs, or refer to the [migration overview](windows/migrate-to-managed-disks.md).

### Can I colocate unmanaged and managed disks on the same VM?

No.

### Can I shrink or downsize my managed disks?

No.

### Can I change the computer name property when I use a specialized (not created by using the System Preparation tool or generalized) operating system disk to provision a VM?

No. You can't update the computer name property. The new VM inherits it from the parent VM, which was used to create the operating system disk. 

### Where can I find sample Azure Resource Manager templates to create VMs with managed disks?

* [List of templates using managed disks](https://github.com/Azure/azure-quickstart-templates/)
* https://github.com/chagarw/MDPP

### When I create a disk from a blob, is there any continually existing relationship with that source blob?

No. When you create the new disk, you make a full standalone copy of that blob at that time and there's no connection between the two. If you like, you can delete the source blob without affecting the newly created disk in any way.

### Can I rename a managed or unmanaged disk after I create it?

You can't rename managed disks. However, you can rename an unmanaged disk as long as it isn't currently attached to a VHD or VM.

### Can I use GPT partitioning on an Azure Disk?

Generation 1 images can only use GPT partitioning on data disks, not OS disks. OS disks must use the MBR partition style.

[Generation 2 images](./generation-2.md) can use GPT partitioning on the OS disk and the data disks.

### What options does Azure disk reservation offer?

Azure disk reservation provides the option to purchase Premium SSDs in the specified SKUs from P30 (1 TiB) up to P80 (32 TiB) for a one-year term. There's no limitation on the minimum number of disks necessary to purchase a disk reservation. Additionally, you can choose to pay with a single, upfront payment or monthly payments. There's no additional transactional cost applied for Premium SSD managed disks.    

Reservations are made in the form of disks, not capacity. In other words, when you reserve a P80 (32 TiB) disk, you get a single P80 disk, you can't then divide that specific reservation up into two smaller P70 (16 TiB) disks. You can reserve as many or as few disks as you like, including two separate P70 (16 TiB) disks.

### How is Azure disk reservation applied?

Disks reservation follows a model similar to reserved virtual machine (VM) instances. The difference is that a disk reservation can't be applied to different SKUs, while a VM instance can. See [Save costs with Azure Reserved VM Instances](./prepay-reserved-vm-instances.md) for more information on VM instances.     

### Can I use my data storage purchased through Azure disks reservation across multiple regions?

Azure disks reservations are purchased for a specific region and SKU (like P30 in East US 2), and can't be used outside these constructs. You can always purchase an additional Azure Disks Reservation for your disk storage needs in other regions or SKUs.    

### Do managed disks support "single instance VM SLA"?

Yes, all disk types support single instance VM SLA.

### Can I attach a disk to a VM in another region?

No. All managed disks, even shared disks, must be in the same region as the VM they're attaching to.

### For Premium SSD v2, why does the Azure pricing calculator show different pricing?

Premium SSD v2 provides a baseline performance of 3,000 IOPS and 125 MB/s for any size at no extra cost. Currently, in the following regions, the pricing calculator shows an incorrect price. Until the pricing calculator is corrected, deduct 3,000 IOPS from target IOPS and 125 MB/s for target bandwidth for these regions to correctly estimate the cost: North Central US, West Central US, West US, UK West, UAE Central, UAE North, Switzerland North, Switzerland West, Sweden South, Korea Central, Korea South, Japan East, Japan West, Italy North, Central India, South India, Germany North, Germany West Central, France Central, Canada Central, Canada East, Brazil South, US GOV Arizona, US GOV Virginia, Australia Central, Australia Central 2, Australia Southeast, South Africa North, South Arica West.

### Why should I use the data disk to store applications and data instead of the OS disk?

- By frequently backing up your data disks, you can have more efficient backup and recovery operations, which reduces downtime in the event of data loss or system failure. If the data disk experiences an issue, it's easier to recover since it's separate from the OS disk.

- You can expand your storage capacity by adding or expanding data disks without affecting the OS disk. Only data disks support live resize. You can't increase the size of OS disks without stopping the VM. 

- Separating the OS from your applications and data onto different disks helps ensure that disks IOPS from the operating system and the application data don't interfere with each other. This separation can enhance overall system performance and prevent contention issues that might happen when both the OS and applications are competing for disk resources.

- If there's a system failure or corruption on the OS disk, you can reimage or reinstall the operating system without affecting the data disk. This approach reduces the risk of data loss and makes it easier when troubleshooting and for maintenance procedures.

- You can apply separate access controls and permissions to the OS disk and the data disk. This separation limits access to critical system files on the OS disk, mitigating potential security risks.

## Snapshots

### Can I copy an encrypted incremental snapshot across regions?

Yes.

### Can I copy snapshots in an order other than their order of creation to another region?

No. You must copy snapshots to other regions in creation order.

### If a source incremental snapshot is deleted before a copy across regions completes, what happens?

The copy fails.

### Are managed snapshots and images encrypted?

Yes. All managed snapshots and images are automatically encrypted. 

### What disk types support snapshots?

All disk types support some form of snapshot. For Ultra Disks and Premium SSD v2 disks, they only support incremental snapshots and have some limitations. For details, see [Create an incremental snapshot for managed disks](disks-incremental-snapshots.md). The other disk types support both types of snapshots for all their disk sizes.

### What happens if I have multiple incremental snapshots and delete one of them?

Deleting one of your incremental snapshots doesn't affect subsequent incremental snapshots. The system merges the data occupied by the first snapshot with the next snapshot under the hood to ensure that the subsequent snapshots aren't impacted due to the deletion of the first snapshot.

## Azure shared disks

### If I have an existing disk, can I enable shared disks on it?

Compatible managed disks created with API version 2019-07-01 or newer can enable shared disks. To do this, you need to unmount the disk from all VMs that it's attached to. Then, edit the **maxShares** property on the disk.

### If I no longer want to use a disk in shared mode, how do I disable it?

Unmount the disk from all VMs that it's attached to. Then edit the maxShare property on the disk to 1.

### Can you increase the size of a shared disk?

Yes.

## Ultra Disks

### What should I set my Ultra Disk throughput to?

If you're unsure what to set your disk throughput to, start by assuming an IO size of 16 KiB and adjust the performance as you monitor your application. Use the following formula: Throughput in MB/s = number of IOPS * 16 / 1000.

### I configured my disk to 40,000 IOPS but I see only 12,800 IOPS. Why am I not seeing the performance of the disk?

In addition to the disk throttle, there's an IO throttle that gets imposed at the VM level. Ensure that the VM size you're using can support the levels that are configured on your disks. For details regarding IO limits imposed by your VM, see [Sizes for virtual machines in Azure](sizes.md).

### Can I use caching levels with an Ultra Disk?

No, Ultra Disks don't support the different caching methods that other disk types support. Set the disk caching to **None**.

### Can I attach an Ultra Disk to my existing VM?

Maybe. Your VM has to be in a region and availability zone pair that supports Ultra Disks. See [getting started with Ultra Disks](disks-enable-ultra-ssd.md) for details.

### Can I use an Ultra Disk as the OS disk for my VM?

No, Ultra Disks are only supported as data disks. You can migrate data from an existing data disk to an Ultra Disk. Attach both disks to the same VM and directly copy the data to the Ultra Disk, or use a third party solution for data migration.

### Can I convert an existing disk to an Ultra Disk?

No, but you can migrate the data from an existing disk to an Ultra Disk. To migrate an existing disk to an Ultra Disk, attach both disks to the same VM, and copy the disk's data from one disk to the other or use a third party solution for data migration.

### Can I attach an Ultra Disk to a VM running in an availability set?

No, this feature isn't currently supported.

## Uploading to a managed disk

### Can I upload data to an existing managed disk?

No, upload can only be used during the creation of a new empty disk with the **ReadyToUpload** state.

### Can I attach a disk to a VM while it's in an upload state?

No.

## Migrate to managed disks

### Does migration affect managed disk performance?

Migration moves the disk from one storage location to another. The process uses a background copy of data, which can take several hours to complete, typically less than 24 hours depending on the amount of data in the disks. During that time, your application might experience higher than usual read latency as some reads get redirected to the original location, and can take longer to complete. There's no impact on write latency during this period.  

### What changes are required in a pre-existing Azure Backup service configuration before or after migration to managed disks?

No changes are required.

### Will my VM backups created through Azure Backup service before the migration continue to work?

Yes, backups work seamlessly.

### What changes are required in a pre-existing Azure Disks Encryption configuration before or after migration to managed disks?

No changes are required.

### Is automated migration of an existing virtual machine scale set from unmanaged disks to managed disks supported?

No. You can create a new scale set with managed disks by using the image from your old scale set with unmanaged disks.

### Can I create a managed disk from a page blob snapshot taken before migrating to managed disks?

No. You can export a page blob snapshot as a page blob and then create a managed disk from the exported page blob.

### Can I fail over my on-premises machines protected by Azure Site Recovery to a VM with managed disks?

Yes, you can choose to fail over to a VM with managed disks.

### Is there any impact of migration on Azure VMs protected by Azure Site Recovery via Azure to Azure replication?

No. Azure Site Recovery Azure to Azure protection for VMs with managed disks is available.

### Can I migrate VMs with unmanaged disks that are located on storage accounts that are or were previously encrypted to managed disks?

Yes.

## Managed disks and Storage Service Encryption

### Is server-side encryption enabled by default when I create a managed disk?

Yes. Managed disks are encrypted by using server-side encryption with platform-managed keys.

### Is the boot volume encrypted by default on a managed disk?

Yes. By default, all managed disks are encrypted, including the OS disk.

### Can I disable Server-side Encryption for my managed disks?

No.

### Does Azure Site Recovery support server-side encryption with customer-managed key for on-premises to Azure and Azure to Azure disaster recovery scenarios?

Yes. 

### Can I backup managed disks encrypted with server-side encryption with customer-managed key using Azure Backup service?

Yes.

### Can I convert VMs with unmanaged disks that are located on storage accounts that are or were previously encrypted to managed disks?

Yes.

### Will an exported VHD from a managed disk or a snapshot also be encrypted?

No. But if you export a VHD to an encrypted storage account from an encrypted managed disk or snapshot, then it's encrypted.

### Can I switch from Azure Disk Encryption to server-side encryption with either customer-managed keys or encryption at host?

Yes, it's possible to migrate from Azure Disk Encryption to encryption at host, though it requires creating new disks and VMs rather than in-place conversion. For detailed step-by-step instructions, see [Migrate from Azure Disk Encryption to encryption at host](disk-encryption-migrate.md).

### Can I switch from server-side encryption with customer-managed keys to Azure Disk Encryption?

Yes. First, switch your disk to use platform-managed keys with one of the following steps:

# [Portal](#tab/azure-portal)

1. Sign in to the Azure portal.
1. Select the disk you'd like to change the encryption type of.
1. Select **Encryption**.
1. For **Key management** select **Platform-managed key** and select save.

Your managed disk has successfully switched from being secured with your own customer-managed key to a platform-managed key.

# [Azure CLI](#tab/azure-cli)

Your existing disks must not be attached to a running VM in order for you to encrypt them using the following script:

```azurecli
rgName=yourResourceGroupName
diskName=yourDiskName

az disk update -n $diskName -g $rgName --encryption-type EncryptionAtRestWithPlatformKey
```

# [Azure PowerShell](#tab/azure-powershell)

Your existing disks must not be attached to a running VM in order for you to encrypt them using the following script:

```PowerShell
$rgName = "yourResourceGroupName"
$diskName = "yourDiskName"

New-AzDiskUpdateConfig -EncryptionType "EncryptionAtRestWithPlatformKey" | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
```
---
Then, encrypt your current disk with Azure Disk Encryption.

## Premium SSD managed disks

### If a VM uses a size series that supports Premium SSDs, such as a DSv2, can I attach both premium and standard data disks?

Yes.

### Can I deploy a VM with an unmanaged disk in the Azure portal?

No. Unmanaged disks are retired, so VM deployments should use managed disks. If you have existing workloads that were using unmanaged disks, migrate them to managed disks. For migration guidance, see [Convert a Windows VM from unmanaged disks to managed disks](windows/convert-unmanaged-to-managed-disks.md) and [Convert a Linux VM from unmanaged disks to managed disks](linux/convert-unmanaged-to-managed-disks.md).

### Can I attach both premium and standard data disks to a size series that doesn't support Premium SSDs, such as D, Dv2, G, or F series?

No. You can attach only standard data disks to VMs that don't use a size series that supports Premium SSDs.

### Are there transaction costs to use Premium SSDs?

There's a fixed cost for each disk size, which comes provisioned with specific limits on IOPS and throughput. The other costs are outbound bandwidth, snapshot capacity, and costs associated with [on-demand bursting](disk-bursting.md#billing), if applicable. For more information, see the [pricing page](https://azure.microsoft.com/pricing/details/managed-disks/).

### What are the limits for IOPS and throughput that I can get from the disk cache?

The combined limits for cache and local SSD for a DS series are 4,000 IOPS per core and 33 MB/s per core. The GS series offers 5,000 IOPS per core and 50 MB/s per core.

### Is the local SSD supported for a VM using managed disks?

The local SSD is temporary storage that is included with a VM using managed disks. There's no extra cost for this temporary storage. You shouldn't use this local SSD to store application data because it isn't persisted in Azure Storage.

### Are there any repercussions for the use of TRIM on premium disks?

There's no downside to the use of TRIM on Azure disks on either premium or standard disks.

## New disk sizes

### What is the largest managed disk size supported for operating system and data disks on Gen1 VMs?

The partition type that Azure supports for Gen1 operating system disks is the master boot record (MBR). Although Gen1 OS disks only support MBR the data disks support GPT. While you can allocate up to a 4-TiB OS disk, the MBR partition type can only use up to 2 TiB of this disk space for the operating system. Azure supports up to 32 TiB for managed data disks.

### What is the largest managed disk size supported for operating system and data disks on Gen2 VMs?

The partition type that Azure supports for Gen2 operating system disks is GUID Partition Table (GPT). Gen2 VMs support up to a 4-TiB OS disk. Azure supports up to 32 TiB for managed data disks.

### What is the largest unmanaged disk size supported for operating system and data disks?

Unmanaged disks are retired. Before retirement, the partition type for an operating system disk using unmanaged disks was master boot record (MBR). While you could allocate up to 4 TiB for an OS disk, the MBR partition type could only use up to 2 TiB of this disk space for the operating system. Unmanaged data disks supported up to 4 TiB.

### What is the largest supported page blob size?

The largest page blob size that Azure supports is 8 TiB (8,191 GiB). The maximum page blob size when attached to a VM as data or operating system disks is 4 TiB (4,095 GiB).

### Are P4 and P6 disk sizes supported for unmanaged disks or page blobs?

Unmanaged disks are retired. For page blobs, P4 (32 GiB) and P6 (64 GiB) aren't the default blob tiers. To use them, explicitly [set the Blob Tier](/rest/api/storageservices/set-blob-tier) to P4 or P6. Otherwise, page blobs with content length less than 32 GiB or between 32 GiB and 64 GiB map to P10 with 500 IOPS and 100 MB/s and the corresponding pricing tier.

### If my existing premium managed disk less than 64 GiB was created before the small disk was enabled (around June 15, 2017), how is it billed?

Existing small premium disks less than 64 GiB continue to be billed according to the P10 pricing tier.

### How can I switch the disk tier of small premium disks less than 64 GiB from P10 to P4 or P6?

You can take a snapshot of your small disks and then create a disk to automatically switch the pricing tier to P4 or P6 based on the provisioned size. You can also use performance tiers, see the articles for changing performance tiers with either the [Azure CLI/PowerShell module](disks-performance-tiers.md) or the [Azure portal](disks-performance-tiers-portal.md).

### Can I resize existing managed disks from sizes fewer than 4 tebibytes (TiB) to 32 TiB?

Yes.

### What are the largest disk sizes supported by Azure Backup and Azure Site Recovery service?

The largest disk size supported by Azure Backup is 32 TiB (4 TiB for encrypted disks). The largest disk size supported by Azure Site Recovery is 8 TiB. Support for the larger disks up to 32 TiB isn't yet available in Azure Site Recovery.

### What are the recommended VM sizes for larger disk sizes (>4 TiB) for Standard SSD and Standard HDDs to achieve optimized disk IOPS and Bandwidth?

To achieve the disk throughput of Standard SSD and Standard HDD large disk sizes (>4 TiB) beyond 500 IOPS and 60 MB/s, we recommend you deploy a new VM from one of the following VM sizes to optimize your performance: B-series, DSv2-series, Dsv3-Series, ESv3-Series, Fs-series, Fsv2-series, M-series, GS-series, NCv2-series, NCv3-series, or Ls-series VMs. Attaching large disks to existing VMs or VMs that aren't using the recommended sizes above may experience lower performance.

### How can I upgrade my disks (>4 TiB) which were deployed during the larger disk sizes preview in order to get the higher IOPS & bandwidth at GA?

You can either stop and start the VM that the disk is attached to or, detach and reattach your disk. The performance targets of larger disk sizes have been increased for both Premium SSDs and Standard SSDs at GA.

### Do we support enabling Host Caching on all disk sizes?

Host Caching (**ReadOnly** and **Read/Write**) is supported on disk sizes less than 4 TiB. This means any disk that is provisioned up to 4,095 GiB can take advantage of Host Caching. Host caching isn't supported for disk sizes more than or equal to 4,096 GiB. For example, a P50 premium disk provisioned at 4,095 GiB can take advantage of Host caching and a P50 disk provisioned at 4,096 GiB can't take advantage of Host Caching. We recommend using caching for smaller disk sizes where you can expect to observe better performance boost with data cached to the VM.

## Private Links for managed disks

### How can I ensure that a disk can be exported or imported only via Private Links?

Set the `DiskAccessId` property to an instance of a disk access object and set the NetworkAccessPolicy property to `AllowPrivate`.

### Can I use the SAS URI of a disk or snapshot to download the underlying VHD of a VM in the same subnet as the subnet of the private endpoint associated with the disk?

Yes.

### Can I use a SAS URI of a disk/snapshot to download the underlying VHD of a VM not in the same subnet as the subnet of the private endpoint not associated with the disk?

No.

## What if my question isn't answered here?

If your question isn't listed here, let us know and we'll help you find an answer. You can post a question at the end of this article in the comments. To engage with the Azure Storage team and other community members about this article, use the [Microsoft Q&A question page for Azure Storage](/answers/products/azure?product=storage).

To request features, submit your requests and ideas to the [Azure Storage feedback forum](https://feedback.azure.com/d365community/forum/a8bb4a47-3525-ec11-b6e6-000d3a4f0f84).
