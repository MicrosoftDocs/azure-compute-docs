---
title: Azure N-series GPU driver setup for Linux
description: How to set up NVIDIA GPU drivers for N-series VMs running Linux in Azure
services: virtual-machines
author: yangnicole-ml
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.collection: linux
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 05/06/2026
ms.author: yangnicole
ms.reviewer: padmalathas, mattmcinnes
# Customer intent: "As a cloud administrator, I want to install and configure NVIDIA GPU drivers on Linux-based N-series VMs, so that I can fully utilize their GPU capabilities for high-performance computing applications."
---

# Install NVIDIA GPU drivers on N-series VMs running Linux

**Applies to:** :heavy_check_mark: Linux VMs

> [!IMPORTANT]
> To align with inclusive language practices, this documentation replaces the term "blacklist" with "blocklist". This change reflects a commitment to avoiding terminology that might carry unintended negative connotations or perceived racial bias.
> However, in code snippets and technical references where "blacklist" is part of established syntax or tooling (for example, configuration files, command-line parameters), the original term is retained to preserve functional accuracy. This usage is strictly technical and doesn't imply any discriminatory intent.

To take advantage of the graphics processing unit (GPU) capabilities of an Azure N-series virtual machine (VM) backed by NVIDIA GPUs, you must install NVIDIA GPU drivers. The [NVIDIA GPU Driver Extension](../extensions/hpccompute-gpu-linux.md) installs appropriate NVIDIA Compute Unified Device Architecture (CUDA) or GRID drivers on an N-series VM. You can install and manage the extension by using the Azure portal or tools such as Azure PowerShell. For supported operating systems (OS) and deployment steps, see the [NVIDIA GPU Driver Extension documentation](../extensions/hpccompute-gpu-linux.md).

To install NVIDIA GPU drivers manually, follow the steps outlined in this article. Manual driver setup information is also available for [Windows VMs](../windows/n-series-driver-setup.md).

For technical specifications, see [GPU Linux VM sizes](../sizes-gpu.md?toc=/azure/virtual-machines/linux/toc.json).

> [!WARNING]
> Installing NVIDIA drivers by using methods other than those outlined in this article might result in failure of the intended driver installation. Microsoft and NVIDIA don't support this installation. To ensure proper functionality and support, follow only the installation steps and use the driver versions specified in this article.

## CUDA drivers

NVIDIA distributes CUDA drivers for NCasT4_v3, NC_A100_v4, and NCads_H100_v5 VMs. 

### Supported CUDA drivers

For the latest CUDA drivers and supported OS, visit the [NVIDIA website](https://developer.nvidia.com/cuda-zone). Ensure that you install or upgrade to the latest supported CUDA driver for your distribution. 

### CUDA driver installation

For CUDA driver installation, visit the [NVIDIA website](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html).

### Verify CUDA driver installation

Run `nvidia-smi`. If the driver is installed, NVIDIA SMI lists the **GPU-Util** as N/A until you run a GPU workload on the VM.

## GRID drivers

Microsoft redistributes NVIDIA GRID driver installers for NVv3, NCasT4_v3, NVadsA10_v5, and NCv6 RTX PRO 6000 BSE VMs used as virtual workstations or for virtual applications. 

### Supported GRID drivers

Install these GRID drivers only on these VMs and only on the operating systems listed in the following table. These drivers include licensing for GRID Virtual GPU (vGPU) software in Azure. You don't need to set up an NVIDIA vGPU software license server.

The NCasT4_v3 and NCv6 RTX PRO 6000 BSE series are the only non-NV GPU VM series that support GRID drivers.

|VM Series|GPU|GRID Driver|Supported Linux OS Versions| 
|---|---|---|---|
| NCv6 RTX PRO 6000 BSE |NVIDIA RTX PRO 6000 Blackwell Server Edition GPU (96GB)| [vGPU20](https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run) | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS <br> •	Red Hat Enterprise Linux (RHEL) 9.4, 9.6, 9.7 <br> •	Red Hat Enterprise Linux (RHEL) 8.10 <br> •	SUSE Linux Enterprise Server 15 SP7| 
| NCasT4_v3 |NVIDIA Tesla T4 GPU (16GB)| [vGPU20](https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run) | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS <br> •	Red Hat Enterprise Linux (RHEL) 9.4, 9.6, 9.7 <br> •	Red Hat Enterprise Linux (RHEL) 8.10 <br> | 
| NVadsA10_v5 |NVIDIA A10 GPU (24GB)| [vGPU18](https://download.microsoft.com/download/2a04ca6a-9eec-40d9-9564-9cdea1ab795f/NVIDIA-Linux-x86_64-570.211.01-grid-azure.run) | •	Ubuntu 24.04 LTS <br> •	Ubuntu 22.04 LTS <br> •	Ubuntu 20.04 LTS <br> •	Red Hat Enterprise Linux (RHEL) 9.4, 9.6, 9.7 <br> •	Red Hat Enterprise Linux (RHEL) 8.10 | 
| NVv3 ([retiring September 30th, 2026](/azure/virtual-machines/sizes/gpu-accelerated/nvv3-series-retirement))|NVIDIA Tesla M60 GPU (16GB)| [vGPU16 (LTS)](https://download.microsoft.com/download/8/d/a/8da4fb8e-3a9b-4e6a-bc9a-72ff64d7a13c/NVIDIA-Linux-x86_64-535.161.08-grid-azure.run)| •	Ubuntu 20.04 LTS <br> • Red Hat Enterprise Linux (RHEL) 8.6, 8.8, 8.9 | 

For more information on the specific vGPU and driver branch versions, visit the [NVIDIA](https://docs.nvidia.com/grid/) website.


### GRID driver installation

1. Ensure that Secure Boot and vTPM are disabled.
2. Install prerequisites.
   
   Ubuntu:
   ```
   sudo apt update 
   sudo apt install -y build-essential 
   ```
   RHEL (8.10, 9.4, 9.6, 9.7):
   ```
   sudo yum check-update 
   sudo yum install -y make automake gcc gcc-c++ kernel-devel-$(uname -r) kernel-headers-$(uname -r) 
   ```
1. Download the Linux driver.

   a. For NCv6 RTX PRO 6000 BSE, NCasT4_v3: 

    ```
   wget -O ./NVIDIA-Linux-x86_64-595.58.03-grid-azure.run https://download.microsoft.com/download/51239696-ec04-4c02-a6b3-1d9c608fb57c/NVIDIA-Linux-x86_64-595.58.03-grid-azure.run
   ```

   b. For NVadsA10_v5: 

    ```
   wget -O ./NVIDIA-Linux-x86_64-570.211.01-grid-azure.run https://download.microsoft.com/download/2a04ca6a-9eec-40d9-9564-9cdea1ab795f/NVIDIA-Linux-x86_64-570.211.01-grid-azure.run
   ```

   c. For NVv3: 
   
   ```
   wget -O ./NVIDIA-Linux-x86_64-535.161.08-grid-azure.run https://download.microsoft.com/download/8/d/a/8da4fb8e-3a9b-4e6a-bc9a-72ff64d7a13c/NVIDIA-Linux-x86_64-535.161.08-grid-azure.run
   ```
1. Install the driver.

   a. For NCv6 RTX PRO 6000 BSE:

   ```
   sudo chmod +x ./NVIDIA-Linux-x86_64-595.58.03-grid-azure.run
   sudo ./NVIDIA-Linux-x86_64-595.58.03-grid-azure.run -M open
   ```
   
   b. For NCasT4_v3:
   ```
   sudo chmod +x ./NVIDIA-Linux-x86_64-595.58.03-grid-azure.run
   sudo ./NVIDIA-Linux-x86_64-595.58.03-grid-azure.run
   ```
   
   c. For NVadsA10_v5:

   ```
   sudo chmod +x ./NVIDIA-Linux-x86_64-570.211.01-grid-azure.run
   sudo ./NVIDIA-Linux-x86_64-570.211.01-grid-azure.run
   ```

   d. For NVv3:

   ```
   sudo chmod +x ./NVIDIA-Linux-x86_64-535.161.08-grid-azure.run
   sudo ./NVIDIA-Linux-x86_64-535.161.08-grid-azure.run
   ```


### Verify GRID driver installation

Run `nvidia-smi`. If the driver is installed, NVIDIA SMI lists the **GPU-Util** as N/A until you run a GPU workload on the VM.


## Troubleshooting

* You can set persistence mode using `nvidia-smi` so the output of the command is faster when you need to query cards. To set persistence mode, run `nvidia-smi -pm 1`. If the VM is restarted, the mode setting goes away. You can always script the mode setting to run upon startup.

* If jobs are interrupted by ECC errors on the GPU (either correctable or uncorrectable), first check to see if the GPU meets any of Nvidia's [RMA criteria for ECC errors](https://docs.nvidia.com/deploy/dynamic-page-retirement/index.html#faq-pre). If the GPU is eligible for RMA, contact support about getting it serviced; otherwise, reboot your VM to reattach the GPU as described [here](https://docs.nvidia.com/deploy/dynamic-page-retirement/index.html#bl_reset_reboot). Less invasive methods, such as `nvidia-smi -r`, don't work with the virtualization solution deployed in Azure.

## Next steps

* To capture a Linux VM image with your installed NVIDIA drivers, see [How to generalize and capture a Linux virtual machine](capture-image.md).
