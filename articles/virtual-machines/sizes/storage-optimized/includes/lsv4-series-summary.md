---
title: Lsv4-series summary include file
description: Include file for Lsv4-series summary
author: rrgomatam1
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 03/18/2025
ms.author: rishigomatam
ms.reviewer: mattmcinnes
ms.custom: include file
---
The Lsv4-series of Azure Virtual Machines (VMs) features high-throughput, low latency, directly mapped local NVMe storage. These VMs utilize the fifth Generation Intel® Xeon® Platinum 8573C (Emerald Rapids) processor reaching an all- core turbo clock speed of 3.0 GHz. The Lsv4-series VMs are available in sizes from 2 to 96 vCPUs, with 8 GiB of memory allocated per vCPU and 240GB of local NVMe temp disk capacity allocated per vCPU, with up to 23TB (12x1.92TB) of local temp disk capacity available on the L96s_v4 size.

Lsv4-series VMs are well suited for scale-up or scale-out storage workloads that need a balance of SSD capacity, compute, and memory. These VMs are perfect for big data, relational, NoSQL databases, data analytics, and data warehousing workloads. Examples include Cassandra, MongoDB, Cloudera, Spark, Elastic Search, Redis, and other data-intensive applications.

Lsv4-series VMs support Standard SSD, Standard HDD, and Premium SSD remote disk types. You can also attach Ultra Disk storage based on its regional availability. Remote Disk storage is billed separately from virtual machines.