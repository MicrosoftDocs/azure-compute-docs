---
title: Prevent torn writes with managed disks on Linux virtual machines
description: Learn how to configure atomic writes for Linux virtual machines (VMs) using managed disks with NVMe controllers.
author: roygara
ms.author: rogarana
ms.date: 09/16/2026
ms.topic: concept-article
ms.service: azure-disk-storage
ai-usage: ai-assisted
# Customer intent: As an IT professional, I want to understand how to prevent torn writes on Linux virtual machines using atomic writes with managed disks, so that I can ensure data integrity and improve database performance.
---

# Prevent torn writes with Azure managed disks

Azure managed disks have native protection against torn writes for 8-KiB and 16-KiB blocks of data to ensure data integrity, which you can use to reduce your performance overhead.

A torn write (or a partial write) can occur when a power loss or system crash interrupts a disk write, leaving a data block only partially updated. A partially updated data block results in an inconsistent page containing a mix of old and new data, essentially a torn page. Torn writes compromise data integrity, and data integrity is critical for applications like databases. Databases must detect and resolve torn writes to avoid corrupt records or indexes.

On supported Linux virtual machines (VMs), applications can use managed disks' native torn write prevention through atomic writes for 8-KiB and 16-KiB blocks of data if the issued write aligns with the respective block offset. For example, a 16-KiB I/O issued to the disk at an offset that is a multiple of 16 KiB guarantees atomicity. With atomic writes, managed disks persist the entire block or nothing, so there's never a partial update. Using atomic writes with managed disks' protection against torn writes allows applications to improve transaction throughput and reduce write latency without traditional software overhead for preventing torn writes.

## Traditional torn write protection

PostgreSQL uses full page writes, so the entire page is logged to the write ahead log before updating data files. MySQL (InnoDB) uses a doublewrite buffer, writing each page twice (first to a protected area, then to main storage) to ensure at least one intact copy. Both of these methods are effective but introduce performance overhead due to extra I/Os. With managed disks' torn write prevention, there's no extra performance overhead.

## Limitations

- Your filesystem and your operating system must guarantee large atomic write support
    - Large atomic writes are supported only for Linux virtual machines using Linux kernel version 6.13 or newer
        - Linux kernel version 6.13 introduced support for large atomic writes with direct I/O for XFS and Ext4
- Atomic writes are only available when using managed disks with [NVMe controllers](/azure/virtual-machines/enable-nvme-remote-faqs#what-are-the-prerequisites-to-enable-the-remote-nvme-interface-on-my-vm-)

## Prerequisites

- Deploy a Linux virtual machine (VM) with Linux kernel version 6.13 or newer and a managed disk that uses an [NVMe controller](/azure/virtual-machines/enable-nvme-remote-faqs)
- Install the `nvme-cli` package on your VM
    - If your distribution uses the Advanced Package Tool (APT), use `sudo apt install nvme-cli`
- 8-KiB and 16-KiB issued writes must align with their respective block offsets to ensure torn writes are prevented

## Confirm your virtual machine supports atomic writes

Access your VM and run `lsblk`. Your disk should appear under the NVMe namespace, such as `nvme0n2`.

Use the device name from the `lsblk` output to check the NVMe namespace parameters with `sudo nvme id-ns <your-nvme-here>`. For example, the device path might be `/dev/nvme0n2`. Verify that the values of `nawun`, `nawupf`, and `nabspf` indicate 16-KiB atomic write support. The output of these values should be either `31 (lbads:9)` or `3 (lbads:12)`.

## Verify kernel atomic write support

Next, inspect atomic write parameters for 16-KiB atomic write size and alignment boundaries.

```bash
cat /sys/class/block/nvme0n2/queue/atomic_write_unit_max_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_unit_min_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_boundary_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_max_bytes
```

## Create an XFS filesystem for atomic writes

Once you confirm that both the VM and the kernel support atomic writes, create an XFS filesystem with a 16-KiB block size. You can create an XFS filesystem with the `mkfs.xfs` command. Include the following parameters: `-b size=16384 <your-nvme-here>`. An example NVMe device path is `/dev/nvme0n2`.

After you set up the filesystem, configure your application to write atomically with direct I/O and the same block size as the filesystem. One way of doing this is with the `fio` command. Include the following parameters: `direct=1 atomic=1 bs=16k`.
