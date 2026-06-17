--- 
title: NC_RTXPRO6000BSE_v6 Size Series Overview
description: Overview and FAQs for NC_RTXPRO6000BSE_v6-series sizes 
author: yangnicole-ml 
ms.service: azure-virtual-machines 
ms.subservice: sizes 
ms.topic: concept-article 
ms.custom: references_regions
ms.date: 06/03/2026 
ms.author: yangnicole 
ms.reviewer: yangnicole 
--- 

# NC RTX PRO 6000 Blackwell v6 sizes series overview

The NCv6-series is a single virtual machine (VM) family for single-node compute, virtual desktop infrastructure (VDI), and gaming workloads. Below we outline general information and onboarding for the NCv6-series. 


## Availability and Pricing 

The NCv6-series is now generally available (GA) in the Azure West US 2 and Southeast Asia regions. 

Refer to the [Azure.com pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) to view GA pricing rates.  

For more information, see our [blog on the transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578).  



## VM Software Driver Installation 

**Linux** 

For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) running Linux - refer to [the documentation](/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms).   

**Windows** 

For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) running Windows  – refer to [the documentation](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-gridvgpu-drivers). 



## Supported Operating Systems (OS) and Versions 
| OS Family | Versions | 
| --- | --- | 
| Windows | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session isn't supported). | 
| Windows Server | •	Windows Server 2025 <br> •	Windows Server 2022| 
| Ubuntu | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS| 
| Red Hat Enterprise Linux (RHEL) | •	RHEL 9.7, 9.6, 9.4 <br> •	RHEL 8.10| 
| SUSE Linux Enterprise Server |• SUSE Linux Enterprise Server 16 <br> •	SUSE Linux Enterprise Server 15 SP7, SP6, SP2 | 
| Debian | • Debian 12| 



## FAQs 
- **I am an NCv6 preview customer. How can I manage the transition from preview to general availability (GA)?**

    Please stop or delete all VMs deployed during the preview and create new VMs using any of the VM sizes now available:
   
    | Preview VM Size (Previous) | GA VM Size (New) | 
    | --- | --- |
    | Standard_NC128ds_xl_RTXPRO6000BSE_v6 | Standard_NC144ds_xl_RTXPRO6000BSE_v6 | 
    | Standard_NC256ds_xl_RTXPRO6000BSE_v6 | Standard_NC288ds_xl_RTXPRO6000BSE_v6 | 
    | Standard_NC320ds_xl_RTXPRO6000BSE_v6 | Standard_NC288ds_xl_RTXPRO6000BSE_v6 | 
    | Standard_NC128lds_xl_RTXPRO6000BSE_v6 | Standard_NC144lds_xl_RTXPRO6000BSE_v6 | 
    | Standard_NC256lds_xl_RTXPRO6000BSE_v6 | Standard_NC288lds_xl_RTXPRO6000BSE_v6 | 
    | Standard_NC320lds_xl_RTXPRO6000BSE_v6 | Standard_NC288lds_xl_RTXPRO6000BSE_v6 |

> [!NOTE]
> Any new VM creations with the preview VM sizes (128, 256, and 320 vCPUs) will no longer work correctly and will not be covered under Azure Service Level Agreement (SLA). Only the new VM sizes above will deploy correctly and be covered by the Azure SLA.

Additional information on VM specifications and other GA VM sizes can be found [here](/azure/virtual-machines/sizes/gpu-accelerated/nc-rtxpro6000-bse-v6-series?tabs=sizebasicgp%2Csizebasicco). 

- **How can I receive support?** <br> 

    Open a support ticket in the Azure portal. 

- **Are RTX Pro 6000 Blackwell GPUs exposed via SRIOV (also called “vGPU”) or Passthrough mode?** 

    RTX Pro 6000 Blackwell GPUs in NCv6-series are exposed via SR-IOV. This can't be modified by the end user. The use of SR-IOV does limit end users from capturing certain GPU telemetry, as documented at [Troubleshooting Known Issues with HPC and GPU VMs](/azure/virtual-machines/hb-hc-known-issues), but is necessary to offer VM sizes with partial GPU allocations. 

- **Is the NCv6-series supported on Azure Kubernetes Service (AKS) and Azure Batch?** <br> 

    NCv6 is currently not supported on AKS or Azure Batch. 


- **When is NCv6-series coming to additional regions?** <br>  

    For information on region expansion, refer to our [blog on the transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578).  

- **What is the latest recommended NVIDIA driver for the NCv6-series?** <br>  

    As of April 23, 2026, the latest recommended driver is the v20.x (R595) driver: <br>  

    [Linux vGPU20 driver](https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run) <br> 
    [Windows vGPU20 driver](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) 

    For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) - refer to [the documentation](/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms). <br>  

    For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) running Windows – refer to [the documentation](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-gridvgpu-drivers). 

- **What VM sizes are available for NCv6?** <br>  

    The VM sizes that are available are listed [here](/azure/virtual-machines/sizes/gpu-accelerated/nc-rtxpro6000-bse-v6-series?tabs=sizebasicgp%2Csizebasicco). 

 - **Is Omniverse Isaac-Sim supported on NCv6?** <br>  
 
    Yes, Omniverse Isaac-Sim 6.0 is supported on NCv6. 

- **Why can I not see power nor thermal telemetry when I use `nvidia-smi` for NCv6?** <br>  

   This is a limitation of SR-IOV mode. For more information, see [Troubleshooting Known Issues with HPC and GPU VMs](/azure/virtual-machines/hb-hc-known-issues).

