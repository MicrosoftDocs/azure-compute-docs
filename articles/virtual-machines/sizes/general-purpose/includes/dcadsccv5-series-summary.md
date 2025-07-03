---
title: DCads_cc_v5-series summary include file
description: Include file for DCads_cc_v5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/31/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to understand the capabilities of confidential child capable VMs, so that I can design workloads with heightened isolation and resource efficiency leveraging the Azure confidential VM infrastructure."
---
Confidential child capable VMs allow you to borrow resources from the parent VM you deploy, to create AMD SEV-SNP protected child VMs. The parent VM has almost complete feature parity with any other general purpose Azure VM (for example, D-series VMs). This parent-child deployment model can help you achieve higher levels of isolation from the Azure host and parent VM. These confidential child capable VMs are built on the same hardware that powers our Azure confidential VMs. Azure confidential VMs are now generally available. The DCads_cc_v5-series sizes offer a combination of vCPU, memory and temporary storage for most production workloads.
