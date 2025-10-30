---
title: NVIDIA GPU Driver Extension - Azure Linux VMs
description: Microsoft Azure extension for installing NVIDIA GPU drivers on N-series compute VMs running Linux.
services: virtual-machines
manager: gwallace
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: concept-article
ms.tgt_pltfrm: vm-linux
ms.custom: linux-related-content
ms.date: 06/30/2025
ms.author: jushiman
author: ju-shim
# Customer intent: "As a cloud administrator, I want to install NVIDIA GPU drivers on Linux N-series virtual machines, so that I can optimize GPU performance for compute-intensive applications."
---
# NVIDIA GPU Driver Extension for Linux

This extension installs NVIDIA GPU drivers on Linux N-series virtual machines (VMs). Depending on the VM family, the extension installs CUDA or GRID drivers. When you install NVIDIA drivers by using this extension, you're accepting and agreeing to the terms of the [NVIDIA End-User License Agreement](https://www.nvidia.com/en-us/data-center/products/nvidia-ai-enterprise/eula/). During the installation process, the VM might reboot to complete the driver setup.

Instructions on manual installation of the drivers and the current supported versions are available. An extension is also available to install NVIDIA GPU drivers on [Windows N-series VMs](hpccompute-gpu-windows.md).

> [!NOTE]
> With Secure Boot enabled, all OS boot components (boot loader, kernel, kernel drivers) must be signed by trusted publishers (key trusted by the system). Secure Boot is not supported using Windows or Linux extensions. For more information on manually installing GPU drivers with Secure Boot enabled, see [Azure N-series GPU driver setup for Linux](../linux/n-series-driver-setup.md).
>
> [!Note]
> The GPU driver extensions do not automatically update the driver after the extension is installed. If you need to move to a newer driver version then either manually download and install the driver or remove and add the extension again.
>

## Prerequisites

### Operating system

This extension supports the following OS distros, depending on driver support for the specific OS version:

|Driver | Linux: Ubuntu| Linux: Red Hat Enterprise Linux |
|:---|:---|:---|
|CUDA | 20.04 LTS | 7.9 |
|GRID | 22.04 LTS <br> 24.04 LTS| 8.2 |

> [!NOTE]
> The latest supported CUDA drivers for NC-series VMs are currently 470.82.01. Later driver versions aren't supported on the K80 cards in NC. While the extension is being updated with this end of support for NC, install CUDA drivers manually for K80 cards on the NC-series.

> [!IMPORTANT]
> This document references a release version of Linux that is nearing or at, End of Life (EOL). Please consider updating to a more current version.

### Internet connectivity

The Microsoft Azure Extension for NVIDIA GPU Drivers requires that the target VM is connected to the internet and has access.

## Extension schema

The following JSON shows the schema for the extension:

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.6",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### Properties

| Name | Value/Example | Data type |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.HpcCompute | string |
| type | NvidiaGpuDriverLinux | string |
| typeHandlerVersion | 1.6 | int |

### Settings

All settings are optional. The default behavior is to not update the kernel if not required for driver installation and install the latest supported driver and the CUDA toolkit (as applicable).

| Name | Description | Default value | Valid values | Data type |
| ---- | ---- | ---- | ---- | ---- |
| updateOS | Update the kernel even if not required for driver installation. | false | true, false | boolean |
| driverVersion | NV: GRID driver version.<br> NC/ND: CUDA toolkit version. The latest drivers for the chosen CUDA are installed automatically. | latest | [List](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json) of supported driver versions | string |
| installCUDA | Install CUDA toolkit. Only relevant for NC/ND series VMs. | true | true, false | boolean |

## Deployment

### Azure portal

You can deploy Azure NVIDIA VM extensions in the Azure portal.

1. In a browser, go to the [Azure portal](https://portal.azure.com).

1. Go to the virtual machine on which you want to install the driver.

1. On the left menu, select **Extensions**.

    :::image type="content" source="./media/nvidia-ext-portal/extensions-menu-linux.png" alt-text="Screenshot that shows selecting Extensions in the Azure portal menu.":::

1. Select **Add**.

    :::image type="content" source="./media/nvidia-ext-portal/add-extension-linux.png" alt-text="Screenshot that shows adding a V M extension for the selected V M.":::

1. Scroll to find and select **NVIDIA GPU Driver Extension**, and then select **Next**.

    :::image type="content" source="./media/nvidia-ext-portal/select-nvidia-extension-linux.png" alt-text="Screenshot that shows selecting NVIDIA G P U Driver Extension.":::

1. Select **Review + create**, and select **Create**. Wait a few minutes for the driver to deploy.

    :::image type="content" source="./media/nvidia-ext-portal/create-nvidia-extension-linux.png" alt-text="Screenshot that shows selecting the Review + create button.":::

1. Verify that the extension was added to the list of installed extensions.

    :::image type="content" source="./media/nvidia-ext-portal/verify-extension-linux.png" alt-text="Screenshot that shows the new extension in the list of extensions for the V M.":::

### Azure Resource Manager template

You can use Azure Resource Manager templates to deploy Azure VM extensions. Templates are ideal when you deploy one or more virtual machines that require post-deployment configuration.

The JSON configuration for a virtual machine extension can be nested inside the virtual machine resource or placed at the root or top level of a Resource Manager JSON template. The placement of the JSON configuration affects the value of the resource name and type. For more information, see [Set name and type for child resources](/azure/azure-resource-manager/templates/child-resource-name-type).

The following example assumes the extension is nested inside the virtual machine resource. When the extension resource is nested, the JSON is placed in the `"resources": []` object of the virtual machine.

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.6",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### PowerShell

```powershell
Set-AzVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "NvidiaGpuDriverLinux" `
    -ExtensionType "NvidiaGpuDriverLinux" `
    -TypeHandlerVersion 1.6 `
    -SettingString '{ `
	}'
```

### Azure CLI

The following example mirrors the preceding Resource Manager and PowerShell examples:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.6
```

The following example also adds two optional custom settings as an example for nondefault driver installation. Specifically, it updates the OS kernel to the latest and installs a specific CUDA toolkit version driver. Again, note the `--settings` are optional and default. Updating the kernel might increase the extension installation times. Also, choosing a specific (older) CUDA toolkit version might not always be compatible with newer kernels.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.6 \
  --settings '{ \
    "updateOS": true, \
    "driverVersion": "10.0.130" \
  }'
```

## Troubleshoot and support

### Troubleshoot

You can retrieve data about the state of extension deployments from the Azure portal and by using Azure PowerShell and the Azure CLI. To see the deployment state of extensions for a given VM, run the following command:

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Extension execution output is logged to the following file. Refer to this file to track the status of any long-running installation and for troubleshooting any failures.

```bash
/var/log/azure/nvidia-vmext-status
```

### Exit codes

| Exit code | Meaning | Possible action |
| :---: | --- | --- |
| 0 | Operation successful |
| 1 | Incorrect usage of extension | Check the execution output log. |
| 10 | Linux Integration Services for Hyper-V and Azure not available or installed | Check the output of lspci. |
| 11 | NVIDIA GPU not found on this VM size | Use a [supported VM size and OS](../linux/n-series-driver-setup.md). |
| 12 | Image offer not supported |
| 13 | VM size not supported | Use an N-series VM to deploy. |
| 14 | Operation unsuccessful | Check the execution output log. |

### Known issues
1. GRID driver 16.x and 17.x are having installation issues on Azure kernel 6.11. Nvidia is working on solving this issue, meanwhile, downgrade the Azure kernel to 6.8 by following these steps. Try to reinstall the drivers manually or by using an extension after downgrading the kernel to 6.8.
```
// Get the installed kernel. If kernel 6.11 is installed,  downgrade it to 6.8.
uname -a

// Install  kernel 6.8. Note that kernel  6.11  is not supported.
$ sudo apt install linux-image-6.8.0-1015-azure

// Get the list of installed kernels.
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'

// Uninstall any 6.11 kernels.
sudo apt purge linux-headers-6.11.0-1013-azure  linux-image-6.11.0-1013-azure  linux-modules-6.11.0-1013-azure

// Run the following command to ensure only 6.8 images, headers, and modules are installed and no other versions are present.
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'

// Results from the previous command:
linux-headers-6.8.0-1015-azure
linux-image-6.8.0-1015-azure
linux-modules-6.8.0-1015-azure

// Open the grub settings and modify the GRUB_DEFAULT="0" to GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.8.0-1015-azure".
$ sudo vim /etc/default/grub 
 
// The grub file will look like the following:
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.8.0-1015-azure"
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
///////////////////

// Update GRUB and reboot.
$ sudo update-grub && sudo update-grub2
$ sudo reboot

// Reinstall the driver after reboot.
```

2. `NvidiaGpuDriverLinux` currently installs the latest `17.5` GRID drivers, which is having issues with CUDA on A10 series. NVIDIA is working on solving this issue, meanwhile, use GRID driver `16.5` by passing a runtime setting to the extension.

```azurecli
az vm extension set  --resource-group <rg-name> --vm-name <vm-name>  --name NvidiaGpuDriverLinux --publisher Microsoft.HpcCompute --settings "{'driverVersion':'535.161'}"
```

```ARM templates
{
  "name": "NvidiaGpuDriverLinux",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.11",
    "autoUpgradeMinorVersion": true,
    "settings": {
         "driverVersion": "535.161"
    }
  }
}
```
3. GRID Driver version `17.x` is incompatible on NVv3 (NVIDIA Tesla M60). GRID drivers up to version `16.5` are supported. `NvidiaGpuDriverLinux` installs the latest drivers which are incompatible on NVv3 SKU. Instead, use the following runtime settings to force the extension to install an older version of the driver. For more information on driver versions, see [NVIDIA GPU resources](https://raw.githubusercontent.com/Azure/azhpc-extensions/master/NvidiaGPU/resources.json).

```azurecli
az vm extension set  --resource-group <rg-name> --vm-name <vm-name>  --name NvidiaGpuDriverLinux --publisher Microsoft.HpcCompute --settings "{'driverVersion':'535.161'}"
```

```ARM templates
{
  "name": "NvidiaGpuDriverLinux",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.11",
    "autoUpgradeMinorVersion": true,
    "settings": {
         "driverVersion": "535.161"
    }
  }
}
```
4. Grid 17.5 linux driver has a bug where it impacts CUDA related workload. Error signature typically involves CUDA devices unavailable. While Azure is working to resolve this issue, use GRID driver 16.5 to continue running your workload. 
### Support

If you need more help at any point in this article, contact the Azure experts on the [MSDN Azure and Stack Overflow forums](https://azure.microsoft.com/support/community/). Alternatively, you can file an Azure support incident. Go to [Azure support](https://azure.microsoft.com/support/options/) and select **Get support**. For information about using Azure support, read the [Azure support FAQ](https://azure.microsoft.com/support/faq/).

## Next steps

- For more information about extensions, see [Virtual machine extensions and features for Linux](features-linux.md).
- For more information about N-series VMs, see [GPU optimized virtual machine sizes](../sizes-gpu.md).
