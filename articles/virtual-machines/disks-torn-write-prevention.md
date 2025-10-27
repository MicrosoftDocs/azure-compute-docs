---
title: Prevent torn writes with managed disks
description: Learn about the available Azure disk types for virtual machines, including Ultra Disks, Premium SSDs v2, Premium SSDs, standard SSDs, and Standard HDDs.
author: roygara
ms.author: rogarana
ms.date: 08/14/2025
ms.topic: concept-article
ms.service: azure-disk-storage
ms.custom: references_regions
# Customer intent: As an IT professional, I want to compare Azure managed disk types so that I can select the most suitable disk for each of my virtual machine workloads and optimize cost and performance.
---

# Prevent torn writes while using managed disks

A torn write (or a partial write) can occur when a power loss or system crash interrupts a disk write, leaivng a data block only partially updated. This results in an inconsistent page containing amix of old and new data, essentially a torn page. Torn pages compromise data integrity, so systems like database systems must detect and resolve them to avoid corrupt records or indexes. To prevent torn writes, Azure managed disks natively support atomic write operations for 8 KiB and 16 KiB blocks of data if the write that is issued aligns with the respective block offset. For example, a 16 KiB I/O issued to the disk at an offset that is a multiple of 16 KiB guarantees atomicity. To take advantage of this, 

As of version 6.13, the Linux kernel introduced support for large atomic writes with Direct IO for XFS and EXt4.

## Limitations

- Atomic write operations are only available when using managed disks with NVMe controllers
- Your application must ensure that the filesystem and your OS also guarantee large atomic write support 

## Pre-requisites

## Get started

Access your VM and run `lsblk`, your disk should show up under one of the namespaces.



