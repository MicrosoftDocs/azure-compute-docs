---
title: Troubleshoot per-disk resiliency (preview)
description: Troubleshoot configuration, detach, reattach, and Linux filesystem issues with per-disk resiliency for Azure managed disks.
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: troubleshooting
ms.date: 08/28/2026
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to troubleshoot per-disk resiliency so that data disks detach and reattach as expected and applications resume I/O.
---

# Troubleshoot per-disk resiliency (preview)

This article provides solutions for common configuration, detach, reattach, and Linux filesystem issues with per-disk resiliency for Azure managed disks.

> [!IMPORTANT]
> Per-disk resiliency is currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Per-disk resiliency doesn't take effect

Per-disk resiliency applies only to a disk attached to a supported VM that was newly created in a [supported region](disks-per-disk-resiliency-configure.md#regions-that-support-per-disk-resiliency). If it doesn't take effect, check that:

- The `AllowDiskAvailabilityPolicy` feature shows `Registered` on your subscription.
- The disk's `actionOnDiskDelay` is set to `AutomaticReattach`. See [Verify per-disk resiliency](disks-per-disk-resiliency-configure.md#verify-per-disk-resiliency).
- The disk is attached as a **data disk**. The feature has no effect on OS disks.
- The disk is attached to a supported VM. See [Per-disk resiliency limitations](disks-per-disk-resiliency-configure.md#per-disk-resiliency-limitations).

## You can't change the per-disk resiliency setting

The property can be changed only when the disk isn't in active use. Deallocate the VM or detach the disk, change the setting, and then reattach the disk or restart the VM.

## Writes succeeded while the disk was detached, but the data is missing after it's reattached (Linux)

The filesystem was unmounted, so the writes landed on the parent filesystem—the OS disk or the temporary disk—instead of the data disk. See [Protect the mount point](disks-per-disk-resiliency-configure.md#protect-the-mount-point).

## I/O still fails after the disk is reattached (Linux)

The filesystem might remain unmounted or be remounted as read-only if Linux detects an error. Don't attempt to mount it before you determine whether it requires repair. See [Check and recover the filesystem after reattachment](disks-per-disk-resiliency-configure.md#check-and-recover-the-filesystem-after-reattachment).

## Remounting an XFS filesystem fails with a duplicate UUID error (Linux)

Mount the disk with the `nouuid` option: `sudo mount -o nouuid /dev/<device> /mnt/datadrive`. Don't regenerate the filesystem UUID, because that breaks any `/etc/fstab` entry that mounts by UUID.

If the problem persists, [create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).

## Related per-disk resiliency content

- [Configure per-disk resiliency](disks-per-disk-resiliency-configure.md)
- [Improve workload availability with per-disk resiliency](disks-per-disk-resiliency.md)
- [Resource Health overview](/azure/service-health/resource-health-overview)