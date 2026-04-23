---
title: NC_RTXPRO6000BSE_v6 Size Series Overview
description: Overview and FAQs for NC_RTXPRO6000BSE_v6-series sizes
author: yangnicole-ml
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 04/23/2026
ms.author: yangnicole
ms.reviewer: yangnicole
---
 
# NC RTX PRO 6000 Blackwell Server Edition v6 Sizes Series Overview
 
The NCv6-series is a single VM family for VDI, Gaming, and 1-node AI workloads. Below we outline general information and onboarding for the NCv6-series.
 
---
 
## Availability and Pricing
The NCv6-series is currently in public preview and is available in the West US 2 and Southeast Asia regions. To request public preview access, sign up [here](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9s7orOb3OJJnwABCNj_8JdUMzlLSzJFTTdRRE8yU0UxWFFYQlpYV1hDVy4u).

Please refer to the [Azure.com pricing page](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/) to view preview pricing rates. 

For more information, please refer to our [blog on the upcoming transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578). 
 
---
 
## VM Software Driver Installation
 
**Linux**
 
For information on installing NVIDIA GPU drivers on NCv6-series VMs running Linux - please refer to [the documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms).  
 
**Windows**
 
For information on installing NVIDIA GPU drivers on N-series VMs running Windows  – please refer to [the documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup).
 
---
 
## Supported Operating Systems and Versions
| OS Family | Versions |
| --- | --- |
| Windows | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session is not supported). |
| Windows Server | •	Windows Server 2025 <br> •	Windows Server 2022|
| Ubuntu | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS|
| Red Hat Enterprise Linux | •	Red Hat Enterprise Linux 9.4, 9.6, 9.7 <br> •	Red Hat Enterprise Linux 8.8, 8.10|
| SUSE Linux Enterprise Server | •	SUSE Linux Enterprise Server 15 SP2|
 
---
 
## FAQs

**1) Where can I share feedback on my experience?** <br>

   Please send all feedback through [Azure GPU Feedback](azuregpufeedback@service.microsoft.com). 

**2) Can MIG be disabled?** <br>

   No, MIG cannot be disabled. It is enabled in server firmware, and cannot be modified by end users. MIG is required to offer partial GPU support.

**3) Is the NCv6-series offered in passthrough mode?** <br>

   No, this SKU is offered only in SRIOV mode and is not available in passthrough mode.

**4) Is the NCv6-series supported on Azure Kubernetes Service (AKS) and Azure Batch?** <br>

   NCv6 does not support AKS and Azure Batch at this time. For more information, contact [Azure GPU Feedback](azuregpufeedback@service.microsoft.com).

**5) Where can I sign up for the preview?** <br> 

   To request public preview access, sign up [here](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9s7orOb3OJJnwABCNj_8JdUMzlLSzJFTTdRRE8yU0UxWFFYQlpYV1hDVy4u).

**6) When is the NCv6-series coming to additional regions?** <br> 

   For information on region expansion, please refer to our [blog on the upcoming transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578). 
    

**7) What is the latest recommended NVIDIA driver for the NCv6-series?** <br> 

  As of April 23, 2026, the latest recommended driver is the v20.x (R595) driver. <br> 
  
  [Linux vGPU20 driver](https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run) 
  [Windows vGPU20 driver](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe)

  For information on installing NVIDIA GPU drivers on NCv6-series VMs running Linux - please refer to [the documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms).  
  For information on installing NVIDIA GPU drivers on N-series VMs running Windows  – please refer to [the documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup).


     
     


