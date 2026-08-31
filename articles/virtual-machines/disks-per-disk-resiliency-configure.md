---
title: Configure per-disk resiliency for Azure managed disks (preview)
description: Configure per-disk resiliency on Azure managed data disks to keep a VM running when a data disk has a connectivity or availability issue.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 08/28/2026
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ai-usage: ai-assisted
---

# Configure per-disk resiliency for Azure managed disks (preview)

Per-disk resiliency lets you control how Azure responds when a managed data disk experiences connectivity or availability issues. This article shows you how to register for the preview, enable per-disk resiliency on new and existing disks, and verify that it's working correctly.

> [!IMPORTANT]
> Per-disk resiliency is currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Regions that support per-disk resiliency

[!INCLUDE [disks-per-disk-resiliency-regions](includes/disks-per-disk-resiliency-regions.md)]

## Per-disk resiliency limitations

[!INCLUDE [disks-per-disk-resiliency-limitations](includes/disks-per-disk-resiliency-limitations.md)]

## Prerequisites for per-disk resiliency

- For Linux VMs, a kernel that includes the [SCSI timeout fix](https://git.kernel.org/pub/scm/linux/kernel/git/mkp/scsi.git/commit/?h=6.14/scsi-fixes&id=87c4b5e8a6b65189abd9ea5010ab308941f964a4) required for correct disk detach and reattach behavior.
- To use the command-line examples, install and configure the latest [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/install-azure-powershell).
- For REST API calls, use API version `2022-07-02` or later.
- A [supported region](#regions-that-support-per-disk-resiliency).

## Register for the per-disk resiliency preview

Before you can use per-disk resiliency, register the `AllowDiskAvailabilityPolicy` feature flag on your subscription.

# [Azure CLI](#tab/azure-cli)

Register the feature by using [`az feature register`](/cli/azure/feature#az-feature-register):

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name AllowDiskAvailabilityPolicy
```

Registration can take a few minutes. Check the status until it shows `Registered`:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name AllowDiskAvailabilityPolicy --query properties.state -o tsv
```

After the feature is registered, propagate the change to the resource provider:

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```

# [Azure PowerShell](#tab/azure-powershell)

Register the feature by using [`Register-AzProviderFeature`](/powershell/module/az.resources/register-azproviderfeature):

```azurepowershell-interactive
Register-AzProviderFeature -ProviderNamespace 'Microsoft.Compute' -FeatureName 'AllowDiskAvailabilityPolicy'
```

Registration can take a few minutes. Check the status until it shows `Registered`:

```azurepowershell-interactive
Get-AzProviderFeature -ProviderNamespace 'Microsoft.Compute' -FeatureName 'AllowDiskAvailabilityPolicy'
```

After the feature is registered, propagate the change to the resource provider:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
```

# [REST API](#tab/rest)

Use either the Azure CLI or Azure PowerShell module to register the feature.

---

## Enable per-disk resiliency on a new data disk

You can enable per-disk resiliency when you create a data disk. After the disk is created, attach it to a qualifying VM—one that's newly created in a [supported region](disks-per-disk-resiliency.md#regions-that-support-per-disk-resiliency).

# [Azure CLI](#tab/azure-cli)

Use [`az disk create`](/cli/azure/disk#az-disk-create) with the `--action-on-disk-delay` parameter set to `AutomaticReattach`.

```azurecli-interactive
az disk create \
  --resource-group myResourceGroup \
  --name myDataDisk \
  --location westus \
  --size-gb 1024 \
  --sku Premium_LRS \
  --action-on-disk-delay AutomaticReattach
```

# [Azure PowerShell](#tab/azure-powershell)

Use [`New-AzDiskConfig`](/powershell/module/az.compute/new-azdiskconfig) with the `-ActionOnDiskDelay` parameter set to `AutomaticReattach`, and then create the disk with [`New-AzDisk`](/powershell/module/az.compute/new-azdisk).

```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
  -Location 'westus' `
  -DiskSizeGB 1024 `
  -SkuName 'Premium_LRS' `
  -CreateOption 'Empty' `
  -ActionOnDiskDelay 'AutomaticReattach'

New-AzDisk `
  -ResourceGroupName 'myResourceGroup' `
  -DiskName 'myDataDisk' `
  -Disk $diskConfig
```

# [REST API](#tab/rest)

Send a `PUT` request to the [Disks - Create Or Update](/rest/api/compute/disks/create-or-update) operation and include the `availabilityPolicy` property. Use API version `2022-07-02` or later.

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/disks/myDataDisk?api-version=2022-07-02

{
  "location": "westus",
  "sku": {
    "name": "Premium_LRS"
  },
  "properties": {
    "creationData": {
      "createOption": "Empty"
    },
    "diskSizeGB": 1024,
    "availabilityPolicy": {
      "actionOnDiskDelay": "AutomaticReattach"
    }
  }
}
```

---

## Enable per-disk resiliency on an existing data disk

You can also enable per-disk resiliency on a disk that already exists. Because the setting can only be changed when the disk isn't in active use, first **deallocate the VM** or **detach the disk**.

# [Azure CLI](#tab/azure-cli)

Use [`az disk update`](/cli/azure/disk#az-disk-update) with `--action-on-disk-delay AutomaticReattach`.

```azurecli-interactive
az disk update \
  --resource-group myResourceGroup \
  --name myDataDisk \
  --action-on-disk-delay AutomaticReattach
```

# [Azure PowerShell](#tab/azure-powershell)

Use [`New-AzDiskUpdateConfig`](/powershell/module/az.compute/new-azdiskupdateconfig) with `-ActionOnDiskDelay AutomaticReattach`, and then apply it with [`Update-AzDisk`](/powershell/module/az.compute/update-azdisk).

```azurepowershell-interactive
New-AzDiskUpdateConfig -ActionOnDiskDelay 'AutomaticReattach' |
  Update-AzDisk `
    -ResourceGroupName 'myResourceGroup' `
    -DiskName 'myDataDisk'
```

# [REST API](#tab/rest)

Send a `PATCH` request to the [Disks - Update](/rest/api/compute/disks/update) operation with the `availabilityPolicy` property. Use API version `2022-07-02` or later.

```rest
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/disks/myDataDisk?api-version=2022-07-02

{
  "properties": {
    "availabilityPolicy": {
      "actionOnDiskDelay": "AutomaticReattach"
    }
  }
}
```

---

## Verify per-disk resiliency

Confirm that `actionOnDiskDelay` is set to `AutomaticReattach` on the disk.

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az disk show \
  --resource-group myResourceGroup \
  --name myDataDisk \
  --query availabilityPolicy
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
(Get-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'myDataDisk').AvailabilityPolicy
```

# [REST API](#tab/rest)

Send a `GET` request to the [Disks - Get](/rest/api/compute/disks/get) operation and check `properties.availabilityPolicy.actionOnDiskDelay`.

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/disks/myDataDisk?api-version=2022-07-02
```

---

## Disable per-disk resiliency

To return the disk to the default behavior, set `actionOnDiskDelay` to `None`. As with enabling per-disk resiliency, the VM must be deallocated or the disk detached before you make the change.

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az disk update \
  --resource-group myResourceGroup \
  --name myDataDisk \
  --action-on-disk-delay None
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzDiskUpdateConfig -ActionOnDiskDelay 'None' |
  Update-AzDisk `
    -ResourceGroupName 'myResourceGroup' `
    -DiskName 'myDataDisk'
```

# [REST API](#tab/rest)

```rest
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/disks/myDataDisk?api-version=2022-07-02

{
  "properties": {
    "availabilityPolicy": {
      "actionOnDiskDelay": "None"
    }
  }
}
```

---

## Guest OS behavior

> [!WARNING]
> When Azure detaches a data disk that has per-disk resiliency enabled, the guest operating system sees the disk disappear and then reappear. Use per-disk resiliency only for workloads that can tolerate a data disk becoming temporarily unavailable. Design your applications to handle disks being unmounted with I/O errors returned, and to verify the filesystem after the disk is reattached.

### Behavior on Linux

Linux monitoring tools, such as `udev`, detect when a disk is unmounted and remounted.

The disk transitions through the following stages during a detach-and-reattach cycle.

| Stage | What happens | What you observe |
| --- | --- | --- |
| 1. Healthy | The data disk is attached and I/O succeeds. | `lsblk` lists the disk with its partitions. I/O succeeds. |
| 2. Detached | Azure detaches the disk. Pending and new I/O error out. | Block-layer error, such as `dd: failed to open '/dev/sdc': No such file or directory`. File-layer error: *File descriptor not valid*. SCSI-layer error: *LBA error / Invalid LUN*. `lsblk` no longer shows the detached disk. |
| 3. Reattached | Azure reattaches the disk. | `lsblk` lists the disk again. After the disk is reattached, the block device can become available again without the filesystem being automatically remounted, or the filesystem might require repair. [Check and recover the filesystem after reattachment](#check-and-recover-the-filesystem-after-reattachment) if it's not mounted. |

> [!WARNING]
> - For SCSI disks, a Linux detach and reattach can change the device path. To persist the mount across device path changes, see [Persist the mount](linux/disks-format-mount-data-disks-linux.md#persist-the-mount).
> - After persisting the mount, [protect the mount point](#protect-the-mount-point).
>     - If the filesystem is automatically unmounted when the disk is detached, the mount point becomes an ordinary writable directory on the parent filesystem again. An application that keeps writing to that path succeeds silently, and the data is written to the OS disk or the temporary disk instead of the data disk. The data is hidden again when the data disk is reattached. Protect the mount point before you mount the disk.

#### Protect the mount point

Set the immutable attribute on the empty mount-point directory *before* you mount the disk. While the disk is mounted, the attribute is shadowed and has no effect. If the filesystem is unmounted, the attribute is exposed again, and writes to the path fail with `Operation not permitted` instead of silently being written to the parent filesystem.

```bash
# Create the mount point and confirm that it's empty.
sudo mkdir -p /mnt/datadrive
ls -la /mnt/datadrive

# Protect the mount point before mounting the disk.
sudo chattr +i /mnt/datadrive
```

To confirm that the attribute is set on the correct inode, run `lsattr -d /mnt/datadrive`. The `i` attribute appears only while the disk is unmounted.

#### Check and recover the filesystem after reattachment

After `lsblk` shows that the disk is attached again, check whether the filesystem was automatically remounted. Replace `/mnt/datadrive` with the mount point from your `/etc/fstab` entry.

```bash
findmnt --mountpoint --output TARGET,SOURCE,FSTYPE,OPTIONS /mnt/datadrive
```

If the command shows the filesystem at the expected mount point and its options include `rw`, you don't need to remount it. Resume the workload only after you verify the mount.

If the command returns no output or the mount options include `ro`, the filesystem isn't mounted. Don't mount a filesystem that requires repair.

```bash
sudo mount /mnt/datadrive
findmnt --mountpoint --output TARGET,SOURCE,FSTYPE,OPTIONS /mnt/datadrive
```

Resume the workload only after `findmnt` confirms that the filesystem is mounted at the expected mount point with the `rw` option.

If the mount fails, the filesystem might be corrupted, don't mount a filesystem that requires repair. First, identify its device, UUID, and filesystem type:

```bash
lsblk --fs
```

Before you repair the filesystem, stop writes to the disk and take a snapshot. Keep the filesystem unmounted during repair, and use the repair utility for its filesystem type. For ext4, use `fsck`. For XFS, use `xfs_repair`. For the complete recovery procedure, see [Troubleshoot Linux VM boot issues due to filesystem errors](/troubleshoot/azure/virtual-machines/linux/linux-recovery-cannot-start-file-system-errors#perform-filesystem-repair).

After the repair utility reports that the filesystem is clean, verify that its `/etc/fstab` entry uses the UUID instead of a device path. A detach and reattach can change the device path. For configuration steps, see [Persist the mount](linux/disks-format-mount-data-disks-linux.md#persist-the-mount). Then, mount and verify the filesystem.


### Behavior on Windows

Windows monitoring, such as [Plug and Play notifications](/windows-hardware/drivers/kernel/pnp-notification-overview), detects when a disk is unmounted and remounted.

The disk transitions through the following stages during a detach-and-reattach cycle.

| Stage | What happens | What you observe |
| --- | --- | --- |
| 1. Healthy | The data disk is attached and I/O succeeds. | In Disk Management, the drive shows as allocated space. The `list disk` command in DiskPart shows the disk as **Online**. |
| 2. Detached | Azure detaches the disk. Pending and new I/O error out. | The disk no longer appears in Disk Management or in DiskPart `list disk`. I/O error: `STATUS_NO_SUCH_DEVICE` / `ERROR_NO_SUCH_DEVICE`. |
| 3. Reattached | Azure reattaches the disk. | In Disk Management, the drive shows as allocated space again. The `list disk` command in DiskPart shows the disk as **Online**. |

## Related per-disk resiliency content

- [Azure managed disk types](/azure/virtual-machines/disks-types)
- [Troubleshoot per-disk resiliency](disks-per-disk-resiliency-troubleshoot.md)
- [Attach a data disk to a Linux VM](/azure/virtual-machines/linux/attach-disk-portal)
- [Attach a data disk to a Windows VM](/azure/virtual-machines/windows/attach-managed-disk-portal)
- [Resource Health overview](/azure/service-health/resource-health-overview)
