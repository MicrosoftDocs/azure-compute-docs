---
title: AMD GPU Driver Extension - Azure Linux VMs
description: Microsoft Azure extension for installing AMD GPU drivers on N-series compute VMs running Linux.
services: virtual-machines
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: concept-article
ms.tgt_pltfrm: vm-linux
ms.custom: linux-related-content
ms.date: 04/30/2025
ms.author: padmalathas
author: magatala-MSFT
# Customer intent: As a cloud engineer managing Linux N-series VMs, I want to install AMD GPU drivers using an extension, so that I can optimize my virtual machines for high-performance GPU workloads.
---
# AMD GPU Driver Extension for Linux

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

This extension installs AMD GPU drivers on Linux N-series virtual machines (VMs). When you install AMD drivers by using this extension, you're accepting and agreeing to the terms of the [AMD End-User License Agreement](https://www.amd.com/en/legal/eula/amd-software-eula.html). During the installation process, the VM might reboot to complete the driver setup.

Instructions on manual installation of the drivers and the current supported versions are available. An extension is also available to install AMD GPU drivers on [Linux N-series VMs](../linux/azure-n-series-amd-gpu-driver-linux-installation-guide.md).

> [!NOTE]
> With Secure Boot enabled, all OS boot components, including the boot loader, kernel, and kernel drivers, must be signed by trusted publishers whose keys are trusted by the system. For more information on manually installing GPU drivers with Secure Boot enabled, see [Azure N-series GPU driver setup for Linux](../linux/azure-n-series-amd-gpu-driver-linux-installation-guide.md).
>
> The GPU driver extensions do not automatically update the driver once the extension is installed. To upgrade to a newer driver version, you will need to either uninstall and reinstall the extension or manually install the driver.
>

## Prerequisites

### Operating system
This extension supports the following OS distros, depending on driver support for the specific OS version:

|Distribution | Version|
|------------------|-------------------|
| Linux: Ubuntu|  22.04<br> 24.04 |

>[!Note]
>For installation instructions on other Linux distributions, please visit [AMD's documentation](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.4.0/install/quick-start.html)

### Internet connectivity

The AMD GPU Drivers Extension requires that the target VM is connected to the internet and has access.

## Review the extension schema

The following JSON snippet shows the schema for the extension:

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
    "type": "AmdGpuDriverLinux",
    "typeHandlerVersion": "1.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### Properties

The JSON schema includes values for the following parameters.

| Name | Value/Example | Data type |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.HpcCompute | string |
| type | AmdGpuDriverLinux | string |
| typeHandlerVersion | 1.0 | int |

### Settings

All settings are optional. The default behavior is to not update the kernel if not required for driver installation and install the latest supported driver.

| Name | Default value | Valid values | Data type |
| ---- | ---- | ---- | ---- |
| driverVersion | Latest | [List]( https://github.com/Azure/azhpc-extensions/blob/master/AmdGPU/AMD-GPU-Linux-Resources.json) of supported driver versions | string |

## Deploy the extension

Azure VM extensions can be managed by using the Azure CLI, PowerShell, Azure Resource Manager (ARM) templates, and the Azure portal.

### Azure portal

To install the Azure AMD VM extension in the Azure portal, follow these steps:

1. In the [Azure portal](https://portal.azure.com), go to the virtual machine on which you want to install the extension.

1. Under **Settings**, select **Extensions + Applications**.

   :::image type="content" source="./media/nvidia-ext-portal/extensions-menu.png" alt-text="Screenshot that shows how to select Extensions + Applications for a virtual machine in the Azure portal." border="false":::

1. Under **Extensions**, select **+ Add**.

    :::image type="content" source="./media/amd-ext-portal/add-extension-linux.png" alt-text="Screenshot that shows adding a VM extension for the selected VM in the Azure portal." border="false":::

1. Scroll to find and select **AMD GPU Driver Extension**, and then select **Next**.

    :::image type="content" source="./media/amd-ext-portal/select-amd-extension-linux.png" alt-text="Screenshot that shows selecting AMD GPU Driver Extension.":::

1. Select **Review + create**, and select **Create**. 

   Wait a few minutes for the driver to deploy.

   :::image type="content" source="./media/amd-ext-portal/create-amd-extension-linux.png" alt-text="Screenshot that shows selecting the Review + create button.":::
  
1. Confirm the extension is listed as an installed extension for the virtual machine.

   :::image type="content" source="./media/amd-ext-portal/verify-extension-linux.png" alt-text="Screenshot that shows the new extension in the list of extensions for the V M.":::

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
    "type": "AmdGpuDriverLinux",
    "typeHandlerVersion": "1.0",
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
    -ExtensionName "AmdGpuDriverLinux" `
    -ExtensionType "AmdGpuDriverLinux" `
    -TypeHandlerVersion 1.0 `
    -SettingString '{}'
```

### Azure CLI

The following example mirrors the preceding Resource Manager and PowerShell examples:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name AmdGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.0
```

The following example also adds two optional custom settings as an example for nondefault driver installation. 
```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name AmdGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.0 \
  --settings '{ \
    "driverVersion": "6.3.3" \
  }'
```

## Troubleshooting

### Check extension status

Check the status of your extension deployment in the Azure portal, or by using PowerShell or the Azure CLI.

To see the deployment state of extensions for a given VM, run the following commands:

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```
### Review output logs

View output logs for the Azure AMD VM extension deployment. Refer to this file to track the status of any long-running installation and for troubleshooting any failures.

```bash
/var/log/azure/amd-vmext-status
```

### Respond to exit codes

The following table lists common exit codes for deployment and potential follow-up actions.

| Exit code | Meaning | Possible action |
| :----: | :---- | :---- |
| 0 | Operation successful | No required action. |
| 1 | Incorrect usage of extension | Check the execution output log. |
| 2|Python not found | Check the execution output log.|
| 10 | Linux Integration Services for Hyper-V and Azure not available or installed | Check the output of lspci. |
| 11 | AMD GPU not found on this VM size | Use a [supported VM size and OS.](../linux/azure-n-series-amd-gpu-driver-linux-installation-guide.md). |
| 14 | DPKG frontend (`/var/lib/dpkg/lock-frontend`) is locked by another process | Please try reinstalling after sometime. |
| 15 | DPKG (`/var/lib/dpkg/lock`) is locked by another process | Please try reinstalling after sometime. |
| 17 | Failed to download the driver | Check the execution output log.|
| 18 | Failed to download the driver | Check the execution output log.|
| 19 | Failed to install the driver | Check the execution output log.|
| 20 | Insufficient disk space | Check the execution output log.|
| 21 | Incompatible kernel | Check kernel validity here [AMD-GPU-Linux-Resources](https://raw.githubusercontent.com/Azure/azhpc-extensions/refs/heads/master/AmdGPU/AMD-GPU-Linux-Resources.json).|
| 22 | Compatibility check failed | Check kernel validity here [AMD-GPU-Linux-Resources](https://raw.githubusercontent.com/Azure/azhpc-extensions/refs/heads/master/AmdGPU/AMD-GPU-Linux-Resources.json).|
| 23 | Required variable is not set | Check the execution output log.|

## Next steps
- For more information about extensions, see [Virtual machine extensions and features for Linux](features-linux.md)
- For more information about N-series VMs, see [GPU optimized virtual machine sizes](../sizes-gpu.md).
