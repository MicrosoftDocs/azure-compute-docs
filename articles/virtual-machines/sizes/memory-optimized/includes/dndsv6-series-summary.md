---
title: Dndsv6-series summary include file
description: Include file for Dnsv6-series summary
services: virtual-machines
author: iamwilliew
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 05/07/2025
ms.author: wwilliams
ms.custom: include file
# Customer intent: "As a cloud architect, I want to assess the Dndsv6-series virtual machines, so that I can determine their suitability for high-performance networking and large-scale e-commerce applications in my infrastructure."
---
Dndsv6-series virtual machines run on the 5th Generation Intel® Xeon® Platinum 8573C (Emerald Rapids) processor reaching an all- core turbo clock speed of 3.0 GHz. These virtual machines offer up to 128 vCPU and 512 GiB of RAM. Dndsv6-series virtual machines, an extension of our standard Ddsv6-series, provide better networking performance for most general-purpose workloads. It also provides increased scalability, upgraded CPU, elevated memory bandwidth, and faster remote storage access compared to the prior generation. 

Network Optimized VMs enhance accelerated networking by providing hardware acceleration of initial connection setup for certain traffic types, a task previously performed in software. These enhancements reduce the end-to-end latency for initially establishing a connection or initial packet flow and allow a VM to scale up the number of connections it manages more quickly, subject to application constraints. 

The new Network Optimized sizes make use of enhancements provided by Azure Boost to deliver increased network bandwidth per vCPU, a greater number of vNICs, and improved connection setup performance. The Dndsv6 offers up to 200 Gbps network bandwidth, 15 vNICs and up to 400k Connection per Second (CPS).  

These VM sizes are relevant for workloads such as network virtual appliances, large-scale e-commerce applications that require the ability to handle a high number of user connections, and data transfers and media processing tasks that involve transferring large amounts of data quickly. Other applications will also benefit from increased network connection setup performance.  

Dndsv6-series virtual machines offer local storage disks. You can attach Standard SSDs, Standard HDDs, Premium SSDs, and Premium SSD v2 storage to these virtual machines. You can also attach Ultra Disk storage based on its regional availability. Disk storage is billed separately from virtual machines. 
