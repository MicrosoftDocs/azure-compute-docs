---
title: Azure N-series NVIDIA GPU driver setup for Windows 
description: How to set up NVIDIA GPU drivers for N-series VMs running Windows Server or Windows in Azure
author: yangnicole-ml
manager: jkabat
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.collection: windows
ms.topic: how-to
ms.date: 05/06/2026
ms.author: yangnicole
ms.custom: H1Hack27Feb2017
# Customer intent: As a cloud administrator, I want to install and verify NVIDIA GPU drivers on N-series VMs running Windows, so that I can ensure optimal performance for GPU-accelerated applications in my Azure environment.
---
# Install NVIDIA GPU Drivers on N-series VMs Running Windows 

**Applies to:** :heavy_check_mark: Windows VMs 

To take advantage of the graphics processing unit (GPU) capabilities of an Azure N-series virtual machine (VM) backed by NVIDIA GPUs, you must install NVIDIA GPU drivers. The [NVIDIA GPU Driver Extension](../extensions/hpccompute-gpu-windows.md) installs appropriate NVIDIA Compute Unified Device Architecture (CUDA) or GRID drivers on an N-series VM. You can install and manage the extension using the Azure portal or tools such as Azure PowerShell. See the [NVIDIA GPU Driver Extension documentation](../extensions/hpccompute-gpu-windows.md) for supported operating systems (OS) and deployment steps.

To install NVIDIA GPU drivers manually, follow the steps outlined in this article. Manual driver setup information is also available for [Linux VMs](../linux/n-series-driver-setup.md).

For technical specifications, see [GPU Windows VM sizes](../sizes-gpu.md?toc=/azure/virtual-machines/windows/toc.json). 

> [!WARNING]
> Installing NVIDIA drivers using methods other than those outlined in this guide may result in failure of the intended driver installation, and is unsupported by Microsoft and NVIDIA. To ensure proper functionality and support, please follow only the installation steps and use the driver versions specified in this article. 

## CUDA Drivers

CUDA drivers are distributed by NVIDIA for NCasT4_v3, NC_A100_v4, and NCads_H100_v5 VMs. 

### Supported CUDA Drivers

For the latest CUDA drivers and supported OS, visit the [NVIDIA website](https://developer.nvidia.com/cuda-zone). Ensure that you install or upgrade to the latest supported CUDA driver for your distribution. 

### CUDA Driver Installation

For CUDA driver installation instructions, visit the [NVIDIA website](https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html#axzz4ZcwJvqYi).

## GRID Drivers

Microsoft redistributes NVIDIA GRID driver installers for NVv3, NCasT4_v3, NVadsA10_v5, and NCv6 RTX PRO 6000 BSE VMs used as virtual workstations or for virtual applications.

### Supported GRID Drivers

 Install these GRID drivers only on these VMs and only on the operating systems listed in the following table. These drivers include licensing for GRID Virtual GPU (vGPU) software in Azure. You do not need to set up an NVIDIA vGPU software license server.

|VM Series|GPU|GRID Driver|Supported Windows OS Versions|
|---|---|---|---|
| NCv6 RTX PRO 6000 BSE | NVIDIA RTX PRO 6000 Blackwell Server Edition GPU (96GB) | [vGPU20](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session isn't supported). <br> •	Windows Server 2025 <br> •	Windows Server 2022 |
| NCasT4_v3 | NVIDIA Tesla T4 GPU (16GB) | [vGPU20](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session isn't supported). <br> •	Windows Server 2025 <br> •	Windows Server 2022 | 
| NVadsA10_v5 | NVIDIA A10 GPU (24GB)| [vGPU18](https://download.microsoft.com/download/2a04ca6a-9eec-40d9-9564-9cdea1ab795f/NVIDIA-Linux-x86_64-570.211.01-grid-azure.run) | •	Windows 11 25H2 (including all supported Windows 11 releases up to 25H2. Windows Enterprise multi-session isn't supported). <br> •	Windows Server 2025 <br> •	Windows Server 2022 | 
| NVv3 ([retiring September 30th, 2026](/azure/virtual-machines/sizes/gpu-accelerated/nvv3-series-retirement))|NVIDIA Tesla M60 GPU (16GB)| [vGPU16 (LTS)](https://download.microsoft.com/download/a/3/1/a3186ac9-1f9f-4351-a8e7-b5b34ea4e4ea/538.46_grid_win10_win11_server2019_server2022_dch_64bit_international_azure_swl.exe) | • Win 11 22H2 AVD <br> • Win 11 23H2 AVD <br> •	Windows Server 2019 <br> •	Windows Server 2022| 

For more information on the specific vGPU and driver branch versions, visit the [NVIDIA website](https://docs.nvidia.com/grid/).

### GRID Driver Installation

1. Connect by remote desktop to each N-series VM.

2. Within the virtual machine, download, extract, and install the supported driver for your Windows operating system.

After GRID driver installation is complete, restart the VM to ensure the proper functionality. 

### Verify GRID Driver Installation

1. Open a command prompt and change to the C:\Program Files\NVIDIA Corporation\NVSMI directory.

2. Run `nvidia-smi`. If the driver is installed, Nvidia SMI will list the **GPU-Util** as N/A until you run a GPU workload on the VM.


