---
title: Lasv5-series summary include file
description: Include file for Lasv5-series summary
author: zhousarah
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/18/2026
ms.author: zhousarah
ms.reviewer: 
ms.custom: include file
---
The Lasv5-series of Azure Virtual Machines (VMs) features high-throughput, low latency, directly mapped local NVMe storage. These VMs utilize AMD's fifth Generation EPYC™ 9005 processors that can achieve a boosted maximum frequency of 4.5 GHz. The Lasv5-series VMs are available in sizes from 2 to 160 vCPUs, with 8 GiB of memory allocated per vCPU and 240 GB of local NVMe temp disk capacity allocated per vCPU, with up to 30.7 TB (8x3.84 TB) of local temp disk capacity available on the largest size.

Lasv5-series VMs are well suited for scale-up or scale-out storage workloads that need a balance of SSD capacity, compute, and memory. These VMs are perfect for big data, relational, NoSQL databases, data analytics, and data warehousing workloads. Examples include Cassandra, MongoDB, Cloudera, Spark, Elastic Search, Redis, and other data-intensive applications.

Lasv5-series VMs support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Ultra Disk storage based on its regional availability. Remote Disk storage is billed separately from virtual machines. For more information, see pricing for disks. 
