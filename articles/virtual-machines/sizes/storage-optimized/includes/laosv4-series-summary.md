---
title: Laosv4-series summary include file
description: Include file for Laosv4-series summary
author: rrgomatam1
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/18/2025
ms.author: rishigomatam
ms.reviewer: mattmcinnes
ms.custom: include file
---
The Laosv4-series of Azure Virtual Machines (VMs) features high-throughput, low latency, directly mapped local NVMe storage. These VMs utilize AMD's fourth Generation EPYC™ 9004 processors that can achieve a boosted maximum frequency of 3.7GHz. The Laosv4-series VMs are available in sizes from 2 to 32 vCPUs, with 8 GiB of memory allocated per vCPU and 720GB of local NVMe temp disk capacity allocated per vCPU, with up to 23TB (12x1.92TB) of local temp disk capacity available on the L32aos_v4 size.

These VMs are perfect for distributed, scale-out workloads that require high amounts of local storage capacity per vCPU and the ability to move that data quickly over the network or to an Azure remote storage backend. Workloads like storage caching layers, Elasticsearch, distributed file systems, big data analytics, relational and NoSQL databases, and data warehouses all benefit from the dense storage capabilities of Laosv4 VMs.
 
Laosv4-series VMs support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Ultra Disk storage based on its regional availability. Remote Disk storage is billed separately from virtual machines. For more information, see pricing for disks. 