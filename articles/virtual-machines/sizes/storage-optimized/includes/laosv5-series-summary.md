---
title: Laosv5-series summary include file
description: Include file for Laosv5-series summary
author: zhousarah
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
ms.custom: include file
---
The Laosv5-series of Azure Virtual Machines (VMs) features high-throughput, low latency, directly mapped local NVMe storage. These VMs utilize AMD's fifth Generation EPYC™ 9005 processors that can achieve a boosted maximum frequency of 4.5 GHz. The Laosv5-series VMs are available in sizes from 2 to 160 vCPUs, with 8 GiB of memory allocated per vCPU and 720 GB of local NVMe temp disk capacity allocated per vCPU, with up to 138 TB (9x15.36 TB) of local temp disk capacity available on the largest size.

These VMs are perfect for distributed, scale-out workloads that require high amounts of local storage capacity per vCPU and the ability to move that data quickly over the network or to an Azure remote storage backend. Workloads like storage caching layers, Elasticsearch, distributed file systems, big data analytics, relational and NoSQL databases, and data warehouses all benefit from the dense storage capabilities of Laosv5 VMs.
 
Laosv5-series VMs support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Ultra Disk storage based on its regional availability. Remote Disk storage is billed separately from virtual machines. For more information, see pricing for disks. 
