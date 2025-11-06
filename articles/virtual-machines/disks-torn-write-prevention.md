---
title: Prevent torn writes with managed disks on Linux VMs
description: Learn how to configure atomic writes for Linux VMs using managed disks with NVMe controllers.
author: roygara
ms.author: rogarana
ms.date: 11/06/2025
ms.topic: concept-article
ms.service: azure-disk-storage
# Customer intent: As an IT professional, I want to understand how to prevent torn writes on Linux VMs using atomic write operations with managed disks, so that I can ensure data integrity and improve database performance.
---

# Prevent torn writes with Azure managed disks

Azure managed disks have native protection against torn writes to ensure data integrity, you can use this native protection to reduce performance overhead. A torn write (or a partial write) can occur when a power loss or system crash interrupts a disk write, leaving a data block only partially updated. This results in an inconsistent page containing a mix of old and new data, essentially a torn page. Torn writes compromise data integrity, and data integrity is critical for applications like databases. Databases must detect and resolve torn writes to avoid corrupt records or indexes 

Applications can use managed disk's native torn write prevention through atomic write operations for 8-KiB and 16-KiB blocks of data if the write that is issued aligns with the respective block offset. For example, a 16 KiB I/O issued to the disk at an offset that is a multiple of 16-KiB guarantees atomicity. With atomic writes, managed disks persist the entire block or nothing, so there's never a partial update. Using atomic writes with managed disk's protection against torn writes allows applications to improve transaction throughput and reduce write latency without traditional software overhead for preventing torn writes.

## Traditional torn write protection

PostgreSQL uses full page writes, so the entire page is logged to the write ahead log before updating data files. MySQL (InnoDB) uses a doublewrite buffer, writing each page twice (first to a protected area, then to main storage) to ensure at least one intact copy. Both of these methods are effective but, introduce performance overhead due to extra I/Os. With managed disk's torn write prevention, there's no extra performance overhead.

## Limitations

- Your filesystem and your operating system must guarantee large atomic write support
    - Only supported for Linux VMs using version 6.13 or newer of the Linux kernel
        - As of 6.13, the Linux kernel introduced support for large atomic writes with direct IO for XFS and Ext4
-  Atomic write operations are only available when using managed disks with [NVMe controllers](/azure/virtual-machines/enable-nvme-remote-faqs#what-are-the-prerequisites-to-enable-the-remote-nvme-interface-on-my-vm-)

## Prerequisites

- Deploy a Linux VM with Linux kernel 6.13 or newer with a managed disk that's using an [NVMe controller](/azure/virtual-machines/enable-nvme-remote-faqs)
- Install the **nvme-cli** package on your VM
    - If your distribution uses the Advanced Package Tool (APT), you would use `sudo apt install nvme-cli`
- Issued writes must align with their respective block offsets, otherwise they will be torn

## Confirm your VM supports atomic writes

Access your VM and run `lsblk`, your disk should show up under the NVMe namespace (like nvme0n2).

Check the namespace parameters of the NVMe from the `lsblk` output with `sudo nvme id-ns <your-nvme-here>` (an example would be /dev/nvme0n2) Verify that the values of `nawun`, `nawupf`, and `nabspf` report values that indicate 16-KiB atomic wrote support. For `nawun` you should see either `nawun: 31 (lbads:9)` or `nawun: 3 (lbads:12)`.

## Verify kernel atomic write support

Next, inspect atomic write parameters for 16-KiB atomic write size and alignment boundaries.

```bash
cat /sys/class/block/nvme0n2/queue/atomic_write_unit_max_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_unit_min_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_boundary_bytes
cat /sys/class/block/nvme0n2/queue/atomic_write_max_bytes
```

## Setup and use the filesystem

Once you confirmed both the VM and the kernel support atomic writes, create an XFS filesystem with 16-KiB block size. You can do this using the `mkfs.xfs` command, include the following parameters `-b size=16384 <your-nvme-here>` (an example value for your NVMe is /dev/nvme0n2).

Now that you have setup the filesystem. Configure your application to write atomically with direct IO and the same block size formatted. One example way of doing this is with the `fio` command, include the following parameters: `direct=1 atomic=1 bs=16k`.