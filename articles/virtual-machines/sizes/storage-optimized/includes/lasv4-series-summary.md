---
title: Lasv4-series summary include file
description: Include file for Lasv4-series summary
author: rrgomatam1
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/18/2025
ms.author: rishigomatam
ms.reviewer: mattmcinnes
ms.custom: include file
---
The Lasv4-series of Azure Virtual Machines (VMs) features high-throughput, low latency, directly mapped local NVMe storage. These VMs utilize AMD's fourth Generation EPYC™ 9004 processors that can achieve a boosted maximum frequency of 3.7GHz. The Lasv4-series VMs are available in sizes from 2 to 96 vCPUs, with 8 GiB of memory allocated per vCPU and 240GB of local NVMe temp disk capacity allocated per vCPU, with up to 23TB (12x1.92TB) of local temp disk capacity available on the L96as_v4 size.

Lasv4-series VMs are well suited for scale-up or scale-out storage workloads that need a balance of SSD capacity, compute, and memory. These VMs are perfect for big data, relational, NoSQL databases, data analytics, and data warehousing workloads. Examples include Cassandra, MongoDB, Cloudera, Spark, Elastic Search, Redis, and other data-intensive applications.

Lasv4-series VMs support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Ultra Disk storage based on its regional availability. Remote Disk storage is billed separately from virtual machines. For more information, see pricing for disks. 