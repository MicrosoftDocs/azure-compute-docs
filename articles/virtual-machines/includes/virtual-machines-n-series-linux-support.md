---
title: include file
 description: include file
 services: virtual-machines-linux
author: cynthnan
 ms.service: virtual-machines
 ms.topic: include
 ms.date: 12/01/2025
ms.author: vikancha
ms.custom: include file, linux-related-content
# Customer intent: "As a cloud engineer managing Linux virtual machines, I want to install the appropriate NVIDIA CUDA and GRID drivers, so that I can ensure optimal performance for graphics and compute workloads in my Azure environment."
---

## Supported distributions and drivers

### NVIDIA CUDA drivers

For the latest CUDA drivers and supported operating systems, visit the [NVIDIA](https://developer.nvidia.com/cuda-zone) website. Ensure that you install or upgrade to the latest supported CUDA drivers for your distribution. 

> [!NOTE]
> The latest supported CUDA driver for original NC-series SKU VMs is currently 470.82.01. Later driver versions aren't supported on the K80 cards in NC.
>
> [!NOTE]
>The Azure NVads A10 v5 VMs only support GRID 17.x or higher driver versions. The vGPU driver for the A10 SKU is a unified driver that supports both graphics and compute workloads.

> [!CAUTION]
> Secure Boot and vTPM should be disabled because the process hangs when they're enabled.

> [!TIP]
> As an alternative to manual CUDA driver installation on a Linux VM, you can deploy an Azure [Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview) image. The DSVM edition for Ubuntu 16.04 LTS preinstalls NVIDIA CUDA drivers, the CUDA Deep Neural Network Library, and other tools.


### NVIDIA GRID drivers

> [!NOTE]
> [vGPU18](https://download.microsoft.com/download/f5fdb58d-8a8b-4894-9add-6b93a1456f58/NVIDIA-Linux-x86_64-570.133.20-grid-azure.run) is available for the NCasT4_v3-series.
>
> [vGPU18](https://download.microsoft.com/download/0541e1a5-dff2-4b8c-a79c-96a7664b1d49/NVIDIA-Linux-x86_64-570.195.03-grid-azure.run) is now available for the NVadsA10_v5-series in **public regions only**. vGPU18 for the NVadsA10_v5-series is **not** supported in the Mooncake and Fairfax regions yet. We'll provide an update once vGPU18 becomes supported for the NVadsA10_v5-series in the Mooncake and Fairfax regions.

Microsoft redistributes NVIDIA GRID driver installers for NV and NVv3-series VMs used as virtual workstations or for virtual applications. Install only these GRID drivers on Azure NV VMs, only on the operating systems listed in the following table. These drivers include licensing for GRID Virtual GPU Software in Azure. You don't need to set up an NVIDIA vGPU software license server.

The GRID drivers redistributed by Azure don't work on most non-NV series VMs, like NC, NCv2, NCv3, ND, and NDv2-series VMs, but they work on NCasT4v3 series.

For more information on the specific vGPU and driver branch versions, visit the [NVIDIA](https://docs.nvidia.com/grid/) website.

|Distribution|Driver|
|---|--|
|Ubuntu 20.04 LTS, 22.04 LTS, 24.04 LTS<br/><br/>Red Hat Enterprise Linux 8.6, 8.8, 8.9, 8.10, 9.0, 9.2, 9.3, 9.4, 9.5<br/><br/>SUSE Linux Enterprise Server 15 SP2, 12 SP2,12 SP5<br/><br/>Rocky Linux 8.4| NVIDIA vGPU 18.5, driver branch [R570](https://download.microsoft.com/download/0541e1a5-dff2-4b8c-a79c-96a7664b1d49/NVIDIA-Linux-x86_64-570.195.03-grid-azure.run) <br/><br/> NVIDIA vGPU 18.5, driver branch [R570](https://download.microsoft.com/download/0541e1a5-dff2-4b8c-a79c-96a7664b1d49/NVIDIA-Linux-x86_64-570.195.03-grid-azure.run)

> [!NOTE]
>For Azure NVads A10 v5 VMs, we recommend that you use the latest driver version. The latest NVIDIA major driver branch (n) is only backward compatibile with the previous major branch (n-1). For example, vGPU 17.x is backward compatible with vGPU 16.x only. Driver failures might occur on any VMs still running n-2 or lower when the latest drive branch is rolled out to Azure hosts.
>>
>NVs_v3 VMs only support **vGPU 16 or lower** driver versions.
>>
> GRID Driver 17.3 currently supports only the NCasT4_v3 series of VMs. To use this driver, [download and install GRID Driver 17.3 manually](https://download.microsoft.com/download/7/e/c/7ec792c9-3654-4f78-b1a0-41a48e10ca6d/NVIDIA-Linux-x86_64-550.127.05-grid-azure.run). 
>>
> GRID drivers are having issues with installation on Azure kernel 6.11. To unblock, downgrade the kernel to version 6.8. For more information, see [Known Issues](/azure/virtual-machines/extensions/hpccompute-gpu-linux#known-issues).

Visit [GitHub](https://raw.githubusercontent.com/Azure/azhpc-extensions/refs/heads/master/NvidiaGPU/Nvidia-GPU-Linux-Resources.json) for the complete list of all previous Nvidia GRID driver links.

> [!WARNING] 
> Installation of third-party software on Red Hat products can affect the Red Hat support terms. See the [Red Hat Knowledgebase article](https://access.redhat.com/articles/1067).
