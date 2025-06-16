---
title: Dnldsv6-series summary include file
description: Include file for Dnldsv6-series summary
services: virtual-machines
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 05/07/2025
ms.author: wwilliams
ms.custom: include file
---
Dnldsv6-series virtual machines run on the 5th Generation Intel® Xeon® Platinum 8573C (Emerald Rapids) processor reaching an all- core turbo clock speed of up to 3.0 GHz. These virtual machines offer up to 128 vCPU and 256 GiB of RAM and fast, local SSD storage up to 4x1760 GiB. These VM sizes can reduce cost when running non-memory intensive applications. Dnldsv6-series virtual machines, an extension of our standard Dldsv6-series, provide better networking performance for most general-purpose workloads. 

Network Optimized VMs enhance accelerated networking by providing hardware acceleration of initial connection setup for certain traffic types, a task previously performed in software. These enhancements reduce the end-to-end latency for initially establishing a connection or initial packet flow, and allow a VM to scale up the number of connections it manages more quickly, subject to application constraints. 

The new Network Optimized sizes make use of enhancements provided by Azure Boost to deliver increased network bandwidth per vCPU, a greater number of vNICs, and improved connection setup performance. The Dnldsv6 offers up to 200 Gbps network bandwidth, 15 vNICs and up to 400k Connection per Second (CPS).  

These VM sizes are relevant for workloads such as network virtual appliances, large-scale e-commerce applications that require the ability to handle a high number of user connections, and data transfers and media processing tasks that involve transferring large amounts of data quickly. Other applications will also benefit from increased network connection setup performance. 

Dnldsv6-series virtual machines support Standard SSD, Standard HDD, and Premium SSD disk types. You can also attach Ultra Disk storage based on its regional availability. Disk storage is billed separately from virtual machines. 
