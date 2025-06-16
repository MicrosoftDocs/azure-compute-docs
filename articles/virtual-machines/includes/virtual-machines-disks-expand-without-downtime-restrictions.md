---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 05/28/2025
 ms.author: rogarana
 ms.custom: include file, references_regions
---
> [!IMPORTANT]
> This limitation doesn't apply to Premium SSD v2 or Ultra Disks:
>
> If a Standard HDD, Standard SSD, or Premium SSD disk is 4 TiB or less, deallocate your VM and detach the disk before expanding it beyond 4 TiB. If one of those disk types is already greater than 4 TiB, you can expand it without deallocating the VM and detaching the disk.
    
- Only supported for data disks.
- Not supported for shared disks.
- Install and use either:
    - The [latest Azure CLI](/cli/azure/install-azure-cli)
    - The [latest Azure PowerShell module](/powershell/azure/install-azure-powershell)
    - The [Azure portal](https://portal.azure.com/)
    - Or an Azure Resource Manager template with an API version that's `2021-04-01` or newer.
- Not available on some classic VMs. Use [this script](#expanding-without-downtime-classic-vm-sku-support) to get a list of classic VM SKUs that support expanding without downtime.


### Expand with Ultra Disk and Premium SSD v2

Expanding Ultra Disks and Premium SSD v2 disks without downtime has the following additional limitations:

- You can't expand a disk while a [background copy](../scripts/create-managed-disk-from-snapshot.md#performance-impact---background-copy-process) of data is also occurring on that disk, like when a disk is being backfilled from [snapshots](/azure/virtual-machines/disks-incremental-snapshots?tabs=azure-cli).

- As a public preview, you can expand VMs using NVMe controllers with Ultra Disks or Premium SSD v2 disks without downtime. Since this is a public preview, you should only use this to test the functionality of expanding without downtime, don't expand VMs in production. 

In the following regions you can expand VMs using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime, using either the Azure portal, Azure CLI, or Azure PowerShell module:    
- Southeast Asia
- Brazil South
- Canada Central
- Germany West Central
- Central India (Not currently supported on V6 VMs)

In the following regions, you can only expand VMs using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime, using the Azure CLI or Azure PowerShell module, you can't currently use the Azure portal:
- East Asia
- West Central US (Not currently supported on V6 VMs)

> [!IMPORTANT]
> Allow up to 10 minutes for the correct size to be reflected in Windows VMs and Linux VMs. For Linux VMs, you must perform a [Linux rescan function](/azure/virtual-machines/linux/expand-disks?tabs=ubuntu#detecting-a-changed-disk-size). For Windows VM that doesn't have a workload, you must perform a [Windows rescan function](/windows-hardware/drivers/devtest/devcon-rescan). You can rescan immediately, but if it's within 10 minutes, you might need to rescan again to display the correct size.
