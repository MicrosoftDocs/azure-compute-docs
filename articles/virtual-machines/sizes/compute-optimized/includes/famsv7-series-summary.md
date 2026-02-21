---
title: Famsv7-series summary include file
description: Include file for Famsv7-series summary
author: archatC
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/29/2025
ms.author: archat
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to evaluate the Famsv7 VM series specifications, so that I can determine if they meet the performance needs of our CPU-intensive workloads and optimize our compute costs.
---
The Famsv7-series utilizes AMD's 5th Generation EPYC™ 9005 processor that can achieve a boosted maximum frequency of up to 4.5 GHz. The Famsv7-series VM comes without Simultaneous Multithreading (SMT), meaning a vCPU is now mapped to a full physical core, allowing software processes to run on dedicated and uncontested resources. These new full core VMs suit workloads demanding the highest CPU performance. Famsv7-series offers up to 80 full core vCPUs and 640 GiB of RAM. These VMs offer a combination of core and memory that is ideal for memory-intensive enterprise applications. The new VMs with no local disk provide a better value proposition for workloads that do not require local temporary storage. This series is optimized for scientific simulations, financial and risk analysis, gaming, rendering and other workloads able to take advantage of the exceptional performance. 
