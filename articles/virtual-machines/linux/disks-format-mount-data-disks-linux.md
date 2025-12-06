---
title: Format and mount managed disks to Azure Linux VMs
description: Learn to format, mount, and persist managed disks to Linux VMs with both SCSI and NVMe interfaces
author: roygara
ms.service: azure-disk-storage
ms.custom: linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 09/03/2025
ms.author: rogarana
---

# Format and mount managed disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article covers how to format, mount, and persist managed disks on Azure Linux virtual machines (VMs). Managed disks are persistent storage attached to your VM that can use either SCSI or NVMe interfaces depending on your VM size.

## Prerequisites

Before formatting and mounting a data disk, ensure you have:

- [Identify the correct disk](./add-disk.md#identifying-disks) to avoid data loss
- SSH access to your VM
- Root or sudo privileges

> [!WARNING]
> Always verify you're working with the correct disk before formatting. Formatting the wrong disk can result in data loss.

## Format the disk

Use the latest version of `parted` available for your distribution. If the disk size is 2 tebibytes (TiB) or larger, use GPT partitioning. If the disk size is under 2 TiB, then you can use either MBR or GPT partitioning.

Once you've identified the correct disk (for example, `/dev/nvme0n2` for a data disk), you can format it:

### [SCSI](#tab/scsi)

The following example uses `parted` on `/dev/sdc`, which is where the first data disk typically is on most VMs. Replace `sdc` with the correct option for your disk. It also formats the disk by using the [XFS](https://xfs.wiki.kernel.org/) file system.

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/sdc
sudo mkfs.xfs /dev/sdc1
```

### [NVMe](#tab/nvme)

The following examples assume you used [azure-vm-utils](azure-virtual-machine-utilities.md) to identify disks as shown in the [identifying disks](./add-disk.md#identifying-disks) section.

```bash
# Format the data disk (replace nvme0n2 with your identified disk from azure-nvme-id)
sudo parted /dev/nvme0n2 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/nvme0n2
sudo mkfs.xfs /dev/nvme0n2p1
```
---

Use the [partprobe](https://linux.die.net/man/8/partprobe) utility to ensure the kernel is aware of the new partition and file system. If you don't use `partprobe`, then the `blkid` and the `lsblk` commands won't immediately return the UUID for the new file system.


## Mount the disk

Now, create a directory to mount the file system by using `mkdir`. The following example creates a directory at `/datadrive`:

```bash
sudo mkdir /datadrive
```
Use `mount` to then mount the file system. The following example mounts the `/dev/sdc1` (for SCSI) or `/dev/nvme0n2p1` (for NVMe) partition to the `/datadrive` mount point:

### [SCSI](#tab/scsi)

```bash
# Mount the disk identified earlier (replace sdc1 with your identified disk's partition)
sudo mount /dev/sdc1 /datadrive
```

You can also use the Azure device path:

```bash
sudo mount /dev/disk/azure/scsi1/lun0-part1 /datadrive
```

### [NVMe](#tab/nvme)

```bash
# Mount the disk identified earlier (replace nvme0n2p1 with your identified disk's partition)
sudo mount /dev/nvme0n2p1 /datadrive
```
---


## Persist the mount

To ensure that the drive is remounted automatically after a reboot, add it to the `/etc/fstab` file.

It's highly recommended that the UUID (Universally Unique Identifier) is used in `/etc/fstab` to refer to the drive rather than the device path (such as /dev/sdc1).  Device paths aren't persistent and change on reboot. To find the UUID of the new drive, use the `blkid` utility:

```bash
sudo blkid
```

The output looks similar to the following example:

```
/dev/sda1: LABEL="cloudimg-rootfs" UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4" PARTUUID="1a1b1c1d-11aa-1234-1a1a1a1a1a1a"
/dev/sda15: LABEL="UEFI" UUID="BCD7-96A6" TYPE="vfat" PARTUUID="1e1g1cg1h-11aa-1234-1u1u1a1a1u1u"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4" TYPE="ext4" PARTUUID="1a2b3c4d-01"
/dev/sda14: PARTUUID="2e2g2cg2h-11aa-1234-1u1u1a1a1u1u"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs" PARTLABEL="xfspart" PARTUUID="c1c2c3c4-1234-cdef-asdf3456ghjk"
```

> [!WARNING]
> Improperly editing the `/etc/fstab` file could result in an unbootable system. If you're unsure, see your distribution's documentation for information on how to properly edit this file. You should also create a backup of the `/etc/fstab` file before editing.

Next, open the `/etc/fstab` file in a text editor. Add a line to the end of the file, by using the UUID value for the disk that was created in the previous steps, and the mountpoint of `/datadrive`. Using the example from this article, the new line would look like the following:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2
```

When you're done editing the file, save and close the editor.

Alternatively, you can run the following command to add the disk to the `/etc/fstab` file:

```bash
# For SCSI disks
UUID=$(sudo blkid -s UUID -o value /dev/sdc1)
echo "UUID=$UUID   /datadrive   xfs   defaults,nofail   1   2" | sudo tee -a /etc/fstab

# For NVMe disks
UID=$(sudo blkid -s UUID -o value /dev/nvme1n1p1)
echo "UUID=$UUID   /datadrive   xfs   defaults,nofail   1   2" | sudo tee -a /etc/fstab
```

> [!NOTE]
> Later removing a data disk without editing fstab could cause the VM to fail to boot. Most distributions provide either the `nofail` and/or `nobootwait` fstab options. These options allow a system to boot even if the disk fails to mount at boot time. See your distribution's documentation for details on these parameters.

The `nofail` option ensures that the VM starts even if the filesystem is corrupt or the disk doesn't exist at boot time. Without this option, you might encounter behavior as described in [Cannot SSH to Linux VM due to FSTAB errors](/troubleshoot/azure/virtual-machines/linux-virtual-machine-cannot-start-fstab-errors)

The Azure VM Serial Console can be used for console access to your VM if modifying fstab results in a boot failure. More details are available in the [Serial Console documentation](/troubleshoot/azure/virtual-machines/serial-console-linux).


## TRIM/UNMAP support for Linux in Azure

Some Linux kernels support TRIM/UNMAP operations to discard unused blocks on the disk. This feature is primarily useful to inform Azure that deleted pages are no longer valid and can be discarded. This feature can save money on disks that are billed based on the amount of consumed storage, such as unmanaged standard disks and disk snapshots.

There are two ways to enable TRIM support in your Linux VM. As usual, consult your distribution for the recommended approach:

• Use the `discard` mount option in `/etc/fstab`, for example:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,discard   1   2
```

• In some cases, the `discard` option might have performance implications. Alternatively, you can run the `fstrim` command manually from the command line, or add it to your crontab to run regularly:

### [Ubuntu](#tab/ubuntu)

```bash
sudo apt install util-linux
sudo fstrim /datadrive
```

### [RHEL](#tab/rhel)

```bash
sudo yum install util-linux
sudo fstrim /datadrive
```

### [SLES](#tab/suse)

```bash
sudo zypper in util-linux
sudo fstrim /datadrive
```
---

## Troubleshooting

[!INCLUDE [virtual-machines-linux-lunzero](../includes/virtual-machines-linux-lunzero.md)]
