---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 09/30/2024
 ms.author: rogarana
 ms.custom: include file, references_regions
---    
- Only supported for data disks.
- If a Standard HDD, Standard SSD, or Premium SSD disk is 4 TiB or less, deallocate your VM and detach the disk before expanding it beyond 4 TiB. If one of those disk types is already greater than 4 TiB, you can expand it without deallocating the VM and detaching the disk. This doesn't apply to Premium SSD v2 or Ultra Disks. (This limitation only applies to Standard HDD, Standard SSD, or Premium SSD disks)
- Not supported for shared disks.
- Install and use either:
    - The [latest Azure CLI](/cli/azure/install-azure-cli)
    - The [latest Azure PowerShell module](/powershell/azure/install-azure-powershell)
    - The [Azure portal](https://portal.azure.com/)
    - Or an Azure Resource Manager template with an API version that's `2021-04-01` or newer.
- Not available on some classic VMs. Use [this script](#expanding-without-downtime-classic-vm-sku-support) to get a list of classic VM SKUs that support expanding without downtime.

### Expand with Ultra Disk and Premium SSD v2

Expanding Ultra Disks and Premium SSD v2 disks have the following additional limitations:
- You can't expand a disk while a [background copy](../scripts/create-managed-disk-from-snapshot.md#performance-impact---background-copy-process) of data is also occurring on that disk, like when a disk is being backfilled from [snapshots](https://learn.microsoft.com/azure/virtual-machines/disks-incremental-snapshots?tabs=azure-cli).
- You can't expand a VM that's using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime.

> [!IMPORTANT]
> Allow up to 10 minutes for the correct size to be reflected in Windows VMs and Linux VMs. For Linux VMs, you must perform a [Linux rescan function](https://learn.microsoft.com/azure/virtual-machines/linux/expand-disks?tabs=ubuntu#detecting-a-changed-disk-size). For Windows VM that doesn't have a workload, you must perform a [Windows rescan function](https://learn.microsoft.com/windows-hardware/drivers/devtest/devcon-rescan). You can rescan immediately, but if it's within 10 minutes, you might need to rescan again to display the correct size.

#### Regional availability

Resizing Ultra Disks and Premium SSD v2 disks is currently available in all Premium SSD v2 and Ultra supported regions. 
