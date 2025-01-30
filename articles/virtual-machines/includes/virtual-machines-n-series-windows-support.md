---
 title: include
 description: include
 services: virtual-machines-windows
author: jushiman
 ms.service: virtual-machines
 ms.topic: include
 ms.date: 04/18/2023
ms.author: jushiman
 ms.custom: include
---

## Supported operating systems and drivers

### NVIDIA Tesla (CUDA) drivers

> [!Note]
> The Azure NVads A10 v5 VMs only support vGPU 16.x(536.25) or higher driver version. The vGPU driver for the A10 SKU is a unified driver that supports both graphics and compute workloads.
>

NVIDIA Tesla (CUDA) drivers for all NC* and ND*-series VMs (optional for NV*-series) are generic and not Azure specific. For the latest drivers, visit the [NVIDIA](https://www.nvidia.com/) website.

> [!TIP]
> As an alternative to manual CUDA driver installation on a Windows Server VM, you can deploy an Azure [Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview) image. The DSVM editions for Windows Server 2016 pre-install NVIDIA CUDA drivers, the CUDA Deep Neural Network Library, and other tools.


### NVIDIA GRID/vGPU drivers
> [!Note]
>For Azure NVads A10 v5 VMs we recommend customers to always be on the latest driver version. The latest NVIDIA major driver branch(n) is only backward compatbile with the previous major branch(n-1). For eg, vGPU 17.x is backward compatible with vGPU 16.x only. Any VMs still runnig n-2 or lower may see driver failures when the latest drive branch is rolled out to Azure hosts.
>>
>NVs_v3 VMs only support **vGPU 16 or lower** driver version.
>
>>
>Windows server 2019 support will go away from vGPU 17.x onwards.
>
Microsoft redistributes NVIDIA GRID driver installers for NV, NVv3 and NVads A10 v5-series VMs used as virtual workstations or for virtual applications. Install only these GRID drivers on Azure NV-series VMs, only on the operating systems listed in the following table. These drivers include licensing for GRID Virtual GPU Software in Azure. You don't need to set up a NVIDIA vGPU software license server.

The GRID drivers redistributed by Azure don't work on non-NV series VMs like NCv2, NCv3, ND, and NDv2-series VMs. The one exception is the NCas_T4_V3 VM series where the GRID drivers enable the graphics functionalities similar to NV-series.

The Nvidia extension always installs the latest driver. 

For Windows 11 up to and including 23H2, Windows 10 up to and including  22H2, Server 2022:

- [GRID 17.3 (552.74)](https://go.microsoft.com/fwlink/?linkid=874181) (.exe)

The following links to previous versions are provided to support dependencies on older driver versions.

For Windows Server 2016 1607, 1709:
- [GRID 14.1 (512.78)](https://download.microsoft.com/download/7/3/6/7361d1b9-08c8-4571-87aa-18cf671e71a0/512.78_grid_win10_win11_server2016_server2019_server2022_64bit_azure_swl.exe) (.exe) is the last supported driver from NVIDIA. The newer 15.x and above don't support Windows Server 2016. 

For Windows Server 2012 R2: 
- [GRID 13.1 (472.39)](https://download.microsoft.com/download/7/3/5/735a46dd-7d61-4852-8e34-28bce7f68727/472.39_grid_win8_win7_64bit_Azure-SWL.exe) (.exe)
- [GRID 13 (471.68)](https://download.microsoft.com/download/9/b/4/9b4d4f8d-7962-4a67-839b-37cc95756759/471.68_grid_winserver2012R2_64bit_azure_swl.exe) (.exe)


For links to all previous Nvidia GRID driver versions, visit [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json).
