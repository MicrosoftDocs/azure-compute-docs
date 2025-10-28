---
title: Prevent torn writes with managed disks on Linux VMs
description: Learn how to configure atomic writes for Linux VMs using managed disks with NVMe controlers.
author: roygara
ms.author: rogarana
ms.date: 10/27/2025
ms.topic: concept-article
ms.service: azure-disk-storage
ms.custom: references_regions
# Customer intent: As an IT professional, I want to understand how to prevent torn writes on Linux VMs using atomic write operations with managed disks, so that I can ensure data integrity and improve database performance.
---

# Prevent torn writes while using managed disks

A torn write (or a partial write) can occur when a power loss or system crash interrupts a disk write, leaivng a data block only partially updated. This results in an inconsistent page containing amix of old and new data, essentially a torn page. Torn pages compromise data integrity, so systems like database systems must detect and resolve them to avoid corrupt records or indexes. To prevent torn writes, Azure managed disks natively support atomic write operations for 8 KiB and 16 KiB blocks of data if the write that is issued aligns with the respective block offset. For example, a 16 KiB I/O issued to the disk at an offset that is a multiple of 16 KiB guarantees atomicity. To take advantage of this, 

## When is this helpful?

These atomic write operations are particularly helpful when using databases, as they traditionally employ precautionary measures. PostgreSQL for example, uses full page writes: when a page is modified for the first time after a checkpoint, the entire page (8KB or 16KB) is logged to the write-ahead log before writing it to the data files. Similarly, MySQL's InnoDB engine uses a doublewrite buffer, writing each page twice (first to a protected area, then to the main storage). This way, at least one copy is intact even if the other write is interrupted. These techniques are effective at preventing corruption, but they come with significant performance overhead. Extra writes to logs or duplicate writes to prevent torn writes to disk increases I/O load and can reduce throughput. Cloud benchmarks have shown that using natively supported page sized atomic disk writes in place of traditional torn-write protection can increase transaction throughput by up to 30% while cutting write latency by around 50%. An atomic write in this case means a multi-sector block (like a database page of 8KB or 16KB) is written to disk all-or-nothing. So that the disk either persits the entire block or nothing, and never has a partial update.

## Limitations

- Only supported for Linux VMs using version 6.13 or newer of the Linux kernel
    - As of 6.13, the Linux kernerl introduced support for large atomic writes with direct IO for XFS and EXt4
-  Atomic write operations are only available when using managed disks with NVMe controllers
- Your application must ensure that the filesystem and your OS also guarantee large atomic write support 

## Pre-requisites

Deploy a Linux VM with Linux kernel 6.13 or newer with a managed disk that's using an NVMe controller.

## Get started

### Confirm your VM supports atomic writes

Access your VM and run `lsblk`, your disk should show up under one of the namespaces. In this example it's nvme0n2.

Now that you have your disk, use sudo nvme id-ns to verify that nawun, nawupf, and nabspf all report as four LBAs. This means they support 16k atomic writes.

Next, verify that the kernel detects atomic write parameters for the managed disk.

### Setup and use the filesystem

Once you've confirmed all that, setup an XFS filesystem with 16KiB block size.

Now that you've setup the filesystem. Write to your disk using the pwritev() API using Direct IO and the RWF_Atomic flag.