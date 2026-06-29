---
title: include
 description: include
 services: virtual-machines-windows
author: cynthnan
 ms.service: virtual-machines
 ms.topic: include
 ms.date: 12/01/2025
ms.author: wwilliams
 ms.custom: include
 # Customer intent: "As a cloud administrator using Azure virtual machines, I want to install and manage the appropriate NVIDIA drivers for different VM series, so that I can ensure optimal performance and compatibility for graphics and compute workloads."
---

## Supported operating systems and drivers

### NVIDIA Tesla (CUDA) drivers

> [!Note]
> The Azure NVads A10 v5 VMs only support vGPU 17.x or higher driver version. The vGPU driver for the A10 SKU is a unified driver that supports both graphics and compute workloads.
>

NVIDIA Tesla (CUDA) drivers for all NC* and ND*-series VMs (optional for NV*-series) are generic and not Azure specific. For the latest drivers, visit the [NVIDIA](https://www.nvidia.com/) website.

> [!TIP]
> As an alternative to manual CUDA driver installation on a Windows Server VM, you can deploy an Azure [Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview) image. The DSVM editions for Windows Server 2016 preinstall NVIDIA CUDA drivers, the CUDA Deep Neural Network Library, and other tools.


### NVIDIA GRID/vGPU drivers
> [!NOTE]
> [vGPU20](https://download.microsoft.com/download/169e58c8-9099-481e-a9a9-c237a189710c/595.97_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) is now available for the NCv6 RTX PRO 6000 BSE and NCasT4_v3 series. vGPU20 is **not** yet supported on the NVadsA10_v5-series. We will provide an update for when vGPU20 is supported on the NVadsA10_v5-series. 
>
> [vGPU18](https://download.microsoft.com/download/dcf4d002-3a53-469d-91af-04bddf57a9d7/573.76_grid_win10_win11_server2019_server2022_server2025_dch_64bit_international_azure_swl.exe) is now available for the NVadsA10_v5-series in **Public, China, and Azure Government regions only**.
>

> [!Note]
>For Azure NVads A10 v5 and NC RTX Pro 6000 BSE v6 VMs, we recommend customers to always be on the latest driver version. The latest NVIDIA major driver branch(n) is only backward compatbile with the previous major branch(n-1). For eg, vGPU 17.x is backward compatible with vGPU 16.x only. Any VMs still runnig n-2 or lower may see driver failures when the latest drive branch is rolled out to Azure hosts.
>>
>NVs_v3 VMs only support **vGPU 16 or lower** driver version.
>
>>
>Windows server 2019 support is only available till vGPU 16.x.
>

Microsoft redistributes NVIDIA GRID driver installers for NVv3, NCasT4_v3, NVadsA10_v5, and NCv6 RTX PRO 6000 BSE VMs used as virtual workstations or for virtual applications. Install these GRID drivers only on these VMs and only on the operating systems listed in the following table. These drivers include licensing for GRID Virtual GPU Software in Azure. You don't need to set up an NVIDIA vGPU software license server.

The NCasT4_v3 and NCv6 RTX PRO 6000 BSE series are the only non-NV GPU VM Series that support the GRID Drivers.

The Nvidia extension always installs the latest driver. 

For Windows 11 up to and including 25H2, Windows 10 up to and including 22H2, Server 2019, Server 2022, Server 2025:

- [GRID 18.6 (573.76)](https://download.microsoft.com/download/f7ca3c43-8f24-4365-a4cc-d528574858a8/573.96_grid_win10_win11_server2022_server2025_dch_64bit_international_azure_swl.exe) (.exe)

The following links to previous versions are provided to support dependencies on older driver versions.

For Windows Server 2016 1607, 1709:
- [GRID 14.1 (512.78)](https://download.microsoft.com/download/7/3/6/7361d1b9-08c8-4571-87aa-18cf671e71a0/512.78_grid_win10_win11_server2016_server2019_server2022_64bit_azure_swl.exe) (.exe) is the last supported driver from NVIDIA. The newer 15.x and above don't support Windows Server 2016. 

For Windows Server 2012 R2: 
- [GRID 13.1 (472.39)](https://download.microsoft.com/download/7/3/5/735a46dd-7d61-4852-8e34-28bce7f68727/472.39_grid_win8_win7_64bit_Azure-SWL.exe) (.exe)
- [GRID 13 (471.68)](https://download.microsoft.com/download/9/b/4/9b4d4f8d-7962-4a67-839b-37cc95756759/471.68_grid_winserver2012R2_64bit_azure_swl.exe) (.exe)


For links to all previous Nvidia GRID driver versions, visit [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json).
