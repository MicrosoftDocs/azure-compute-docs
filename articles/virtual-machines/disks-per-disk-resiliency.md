---
title: Improve workload availability with per-disk resiliency (preview)
description: Learn how to use per-disk resiliency on Azure managed data disks to keep a VM running when a data disk has a connectivity or availability issue.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: concept-article
ms.date: 08/28/2026
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ai-usage: ai-assisted
# Customer intent: As a cloud administrator running data-disk workloads like containers, clustered applications, or backup disks, I want a failing data disk to be detached and reattached automatically so that my VM keeps running instead of rebooting.
---

# Improve workload availability with per-disk resiliency (preview)

**Applies to:** ✔️ Linux VMs ✔️ Windows VMs

By default, when the availability or connectivity between a virtual machine (VM) and one of its attached Azure managed disks is affected for an extended period, the Azure platform forcibly shuts down the VM. The VM automatically powers back on after storage connectivity is restored. While this default behavior suits many workloads, some workloads, like if a VM with several independent data disks, would rather keep the VM running when a single data disk is temporarily affected.

To keep the VM running when a data disk experiences an availability or connectivity delay, enable per-disk resiliency by setting the `actionOnDiskDelay` property to `AutomaticReattach` on your data disks. Enabling per-disk resiliency changes the behavior of that disk so that if that data disk exceeds the I/O delay threshold, the disk goes offline and detaches without rebooting the VM. The rest of the VM keeps running. After the underlying platform issue is resolved, Azure automatically reattaches the disk, brings it back online, and makes it available to the VM again.

> [!IMPORTANT]
> Per-disk resiliency is currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How per-disk resiliency works

The `actionOnDiskDelay` property is a property of the data disk and has two acceptable values:

| Value | Behavior |
| --- | --- |
| `None` | Default. The disk uses the standard platform behavior. When I/O to the disk is delayed beyond the threshold, the VM restarts. |
| `AutomaticReattach` | On a disk I/O failure or slow response, Azure detaches the affected data disk (the VM keeps running), and then reattaches it automatically after the platform issue is resolved. |

## Example per-disk resiliency sequence

The following example sequence shows one possible flow for a data disk that has `actionOnDiskDelay` set to `AutomaticReattach` when its I/O is delayed:

1. Azure detects that I/O to the data disk is delayed beyond the platform threshold.
1. Azure takes the disk offline and detaches it from the VM. The VM keeps running, and any I/O to the detached disk returns an error.
1. After the underlying platform issue is resolved, Azure reattaches the disk and brings it back online.
1. The disk is available to the VM again.

With the default value of `None`, the same I/O delay instead causes Azure to restart the VM.

While the disk is offline, the VM's health state is reported as **Degraded**. The Azure portal continues to report the disk as attached to the VM, but the guest operating system reports it as detached. Any in-flight or new I/O to the disk returns an error until the guest operating system detects the reattached disk. Design your application to handle this transient I/O failure. For more information, see [Guest OS behavior](disks-per-disk-resiliency-configure.md#guest-os-behavior).

> [!NOTE]
> When you revoke or disable a data disk's customer-managed key, disk I/O starts to fail as described in the [customer-managed keys](/azure/virtual-machines/disk-encryption#full-control-of-your-keys) section of the server-side encryption article. With per-disk resiliency enabled, Azure detaches the disk instead of shutting down the VM, then reattaches it when you re-enable the key.

## Per-disk resiliency use cases

Per-disk resiliency is most useful for workloads where a single failing data disk shouldn't take down the whole VM:

- Multitenant workloads (for example, containers). A VM node is a runtime environment that hosts many application instances, and individual data disks are often attached to individual containers. With per-disk resiliency enabled on those data disks, a single data disk that has a connectivity or availability timeout is detached without rebooting the entire node.
- Shared disks for clustered or high-availability applications. Azure shared disks provide shared block storage for Windows and Linux clustered applications. Enabling per-disk resiliency on the shared disk helps keep the participating VMs running when connectivity to the shared disk is affected.
- Backup disks attached to a production VM. Enable per-disk resiliency on a backup data disk so that a connectivity or availability timeout on the backup disk doesn't affect the production workload running on the VM.

## Per-disk resiliency limitations

[!INCLUDE [disks-per-disk-resiliency-limitations](includes/disks-per-disk-resiliency-limitations.md)]

## Regions that support per-disk resiliency

[!INCLUDE [disks-per-disk-resiliency-regions](includes/disks-per-disk-resiliency-regions.md)]

## Related per-disk resiliency content

- [Configure per-disk resiliency](disks-per-disk-resiliency-configure.md)
