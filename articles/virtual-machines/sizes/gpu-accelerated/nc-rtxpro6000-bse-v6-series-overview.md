--- 
title: NC_RTXPRO6000BSE_v6 Size Series Overview 
description: Overview and FAQs for NC_RTXPRO6000BSE_v6-series sizes 
author: yangnicole-ml 
ms.service: azure-virtual-machines 
ms.subservice: sizes 
ms.topic: concept-article 
ms.date: 05/07/2026 
ms.author: yangnicole 
ms.reviewer: yangnicole 
--- 

# NC RTX PRO 6000 Blackwell v6 Sizes Series Overview 

The NCv6-series is a single virtual machine (VM) family for single-node compute, virtual desktop infrastructure (VDI), and gaming workloads. Below we outline general information and onboarding for the NCv6-series. 

--- 

## Availability and Pricing 

The NCv6-series is currently in public preview and available in the Azure West US 2 and Southeast Asia regions. To request public preview access, sign up [here](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9s7orOb3OJJnwABCNj_8JdUMzlLSzJFTTdRRE8yU0UxWFFYQlpYV1hDVy4u). 

Refer to the [Azure.com pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) to view preview pricing rates.  

For more information, refer to our [blog on the upcoming transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578).  

--- 

## VM Software Driver Installation 

**Linux** 

For information on installing NVIDIA GPU drivers on NCv6-series VMs running Linux - refer to [the documentation](/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms).   

**Windows** 

For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) running Windows  – refer to [the documentation](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-gridvgpu-drivers). 

--- 

## Supported Operating Systems and Versions 
| OS Family | Versions | 
| --- | --- | 
| Windows | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session isn't supported). | 
| Windows Server | •	Windows Server 2025 <br> •	Windows Server 2022| 
| Ubuntu | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS| 
| Red Hat Enterprise Linux | •	Red Hat Enterprise Linux 9.4, 9.6, 9.7 <br> •	Red Hat Enterprise Linux 8.8, 8.10| 
| SUSE Linux Enterprise Server | •	SUSE Linux Enterprise Server 15 SP7| 
--- 

## FAQs 

**1) Where can I share feedback on my experience?** <br> 

&emsp; Send all feedback to Azure GPU Feedback (azuregpufeedback@service.microsoft.com).  

**2) Are RTX Pro 6000 Blackwell GPUs exposed via SRIOV (also called “vGPU”) or Passthrough mode?** 

&nbsp;&nbsp;&nbsp;&nbsp; RTX Pro 6000 Blackwell GPUs in NCv6-series are exposed via SRIOV. This cannot be modified by the end user. The use of SRIOV does limit end users from capturing certain GPU telemetry, as documented at [Troubleshooting Known Issues with HPC and GPU VMs](/azure/virtual-machines/hb-hc-known-issues), but is necessary to offer VM sizes with partial GPU allocations. 

**3) Is the NCv6-series supported on Azure Kubernetes Service (AKS) and Azure Batch?** <br> 

&emsp; NCv6 will be supported on Azure Batch in May 2026. NCv6 is not supported on AKS at this time. We are working to bring AKS support to this product in Q4 2026.

**4) Where can I sign up for the preview?** <br>  

&emsp; To request public preview access, sign up [here](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9s7orOb3OJJnwABCNj_8JdUMzlLSzJFTTdRRE8yU0UxWFFYQlpYV1hDVy4u). 

**5) When is NCv6-series coming to additional regions?** <br>  

&emsp; For information on region expansion, refer to our [blog on the upcoming transition to general availability](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578).  

**6) What is the latest recommended NVIDIA driver for the NCv6-series?** <br>  

&emsp; As of April 23, 2026, the latest recommended driver is the v20.x (R595) driver: <br>  

&emsp; [Linux vGPU20 driver](https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run) <br> 
&emsp; [Windows vGPU20 driver](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) 

&emsp; For information on installing NVIDIA GPU drivers on NCv6-series VMs running Linux - refer to [the documentation](/azure/virtual-machines/linux/n-series-driver-setup#install-grid-drivers-on-ncv6-rtx-pro-6000-bse-vms). <br>  

&emsp; For information on installing NVIDIA GPU drivers on N-series VMs (including NCv6-series) running Windows – refer to [the documentation](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-gridvgpu-drivers). 

**7) What VM sizes are available for NCv6?** <br>  

&emsp; The VM sizes that are available for testing during public preview are listed [here](/azure/virtual-machines/sizes/gpu-accelerated/nc-rtxpro6000-bse-v6-series?tabs=sizebasicgp%2Csizebasicco). <br>  

&emsp; The VM sizes that will be available when we GA in the coming weeks, including partial GPU VM sizes, are listed [here](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/azure-ncv6-virtual-machines-enhancements-and-ga-transition/4503578).  

 **8) Is Omniverse Isaac-Sim supported on NCv6?** <br>  
 
&emsp; There is a recently discovered bug in Isaac-Sim and Omniverse Kit preventing the application from working correctly on RTX Pro 6000 Blackwell either in SRIOV or MIG mode. <br>  

&emsp;A production fix is available in Omniverse Kit 110.1 and coming to Isaac Sim 6.0 tentatively June 2026. Until then, customers should use NVadsA10v5 with Isaac Sim 5.1 or Isaac Sim 6.0.0-dev2 pre-release and the v18 guest driver ([Windows](https://download.microsoft.com/download/dcf4d002-3a53-469d-91af-04bddf57a9d7/573.76_grid_win10_win11_server2019_server2022_server2025_dch_64bit_international_azure_swl.exe), [Linux](https://download.microsoft.com/download/0541e1a5-dff2-4b8c-a79c-96a7664b1d49/NVIDIA-Linux-x86_64-570.195.03-grid-azure.run)).  

 **9) Why can I not see power nor thermal telemetry when I use `nvidia-smi` for NCv6?** <br>  

&emsp; This is a limitation of SRIOV mode. For more information, see [Troubleshooting Known Issues with HPC and GPU VMs](/azure/virtual-machines/hb-hc-known-issues).

