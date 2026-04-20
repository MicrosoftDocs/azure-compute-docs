---
title: Format and mount temporary disks on Azure Linux VMs
description: Learn to format and mount temporary disks (also known as resource disks) on Azure Linux VMs with both SCSI and NVMe interfaces
author: roygara
ms.service: azure-disk-storage
ms.custom: linux-related-content
ms.collection: linuxFast local storage for temporary data
ms.topic: how-to
ms.date: 09/03/2025
ms.author: rogarana
---

# Format and mount temporary disk to Azure Linux VMs

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article covers how to format and mount [temporary disks](../managed-disks-overview.md#temporary-disk) (also known as resource disks) on Azure Linux virtual machines (VMs). Depending on your VM series, temporary disks use either SCSI or NVMe interfaces. Temporary disks aren't managed disks, and aren't persistent.

Store important data on managed disks instead of local temporary disks. Temporary disks are generally meant to store items like page files, swap files, or SQL Server tempdb files.

## Prerequisites

Before formatting temporary disks:

- [Identify the correct disk](./add-disk.md#identifying-disks) to avoid data loss
- Understand that data isn't persistent across VM stops/deallocations
- Have SSH access to your VM with root or sudo privileges

## Format disks

> [!WARNING]
> Formatting will permanently erase all data on the disk. Ensure you're working with the correct disk and that no important data exists on it.

Use the latest version of `parted` available for your distribution. If the disk size is 2 tebibytes (TiB) or larger, you must use GPT partitioning. If the disk size is under 2 TiB, then you can use either MBR or GPT partitioning.

### [SCSI disks](#tab/scsi)

The following example uses `parted` on `/dev/sdb`, which is typically where the SCSI temporary disks appear. Replace `sdb` with the correct device for your disk. We're using the [XFS](https://xfs.wiki.kernel.org/) file system for better performance.

```bash
sudo parted /dev/disk/azure/resource --script mklabel gpt mkpart xfspart xfs 0% 100%  
sudo partprobe /dev/sdb
sudo mkfs.xfs /dev/sdb1
```

### [NVMe disks](#tab/nvme)

The following examples assume you identified your disk as shown in the [identifying disks](./add-disk.md#identifying-disks) section. If you have azure-vm-utils installed, use it to identify local disks.

```bash
# Example: Format local NVMe disk (replace nvme1n1 with your identified disk)
sudo parted /dev/disk/azure/local/by-index/0 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1p1
```

### [RAID](#tab/raid)

Some VM sizes provide multiple local NVMe disks. You can set up RAID for better performance or redundancy:

```bash
# Example: Create RAID 0 with two local NVMe disks for performance
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/nvme1n1 /dev/nvme2n1
sudo mkfs.xfs /dev/md0
```

---

Use the [partprobe](https://linux.die.net/man/8/partprobe) utility to make sure the kernel is aware of the new partition and file system. If you don't use `partprobe`, then `blkid` or `lsblk` commands might not return the UUID for the new file system immediately.

## Mount temporary disks

Now, create a directory to mount the file system by using `mkdir`. For temporary storage, common mount points include `/mnt`, `/tmp`, or application-specific directories.

```bash
sudo mkdir /mnt/temp
```

### [SCSI disks](#tab/scsi)

Use `mount` to mount the file system. The following example mounts the `/dev/sdb1` partition to the `/mnt/temp` mount point:

```bash
sudo mount /dev/sdb1 /mnt/temp
```

You can also use the Azure device path:

```bash
sudo mount /dev/disk/azure/resource-part1 /mnt/temp
```

### [NVMe disks](#tab/nvme)

The following examples assume you identified your disk as shown in the [identifying disks](./add-disk.md#identifying-disks) section. If you have azure-vm-utils installed, use it to identify local disks.

```bash
# Using direct device path (replace nvme1n1p1 with your identified disk's partition)
sudo mount /dev/nvme1n1p1 /mnt/temp

# Or using Azure device path
sudo mount /dev/disk/azure/local/by-index/0-part1 /mnt/temp  
```

### [RAID](#tab/raid)

For RAID arrays created as shown in the formatting section:

```bash
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid
```
---

## TRIM/UNMAP support for temporary disks

Local temporary disks support TRIM/UNMAP operations. For optimal performance:

### Use the `discard` mount option in `/etc/fstab`:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /mnt/temp   xfs   defaults,discard,nobootwait   0   0
```

### Alternatively, run `fstrim` periodically:

### [Ubuntu](#tab/ubuntu)

```bash
sudo apt install util-linux
sudo fstrim /mnt/temp
```

### [RHEL](#tab/rhel)

```bash
sudo yum install util-linux
sudo fstrim /mnt/temp
```

### [SLES](#tab/suse)

```bash
sudo zypper in util-linux
sudo fstrim /mnt/temp
```
---

## Troubleshooting

[!INCLUDE [virtual-machines-linux-lunzero](../includes/virtual-machines-linux-lunzero.md)]
