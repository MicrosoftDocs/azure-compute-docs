---
title: Install VM Watch
description: Get started by installing VM watch on virtual machines.
author: iamwilliew 
ms.author: wwilliams
ms.service: azure-virtual-machines
ms.topic: get-started
ms.date:     09/23/2024
ms.subservice: monitoring
---

# Install VM watch (preview)

You can enable VM watch by using an [Azure Resource Manager template (ARM template)](/azure/azure-resource-manager/templates/), [PowerShell](/powershell/), or the [Azure CLI](/cli/azure/) on Azure virtual machines (VMs) and Azure virtual machine scale sets. You can enable VM watch on both Linux and Windows virtual machines. VM watch is delivered through the [Application Health VM extension](/azure/virtual-machines/extensions/health-extension?tabs=rest-api) for ease of adoption.

The code in this article details the steps to install the Application Health VM extension and enable VM watch. Note that the code segments require user input. Any labels within angle brackets (`<>`) in the code need to be replaced with values that are specific to your installation. Here's a list of parameters with instructions on what to replace them with.

| **Parameter** | **Description** |
|---|---|
| `<your subscription id>` | The Azure subscription ID in which you want to install VM watch. |
| `<your vm name>` | The name of the virtual machine to which the extension is being installed.  |
| `<your resource group name>` | The name of the resource group in your Azure subscription that your VM will be assigned to. |
| `<your location>` | The Azure region in which your VM is installed. |
| `<your extension name` | The name that will be assigned to the Application Health VM extension that you're installing. |
| `<application health extension type>` | Specifies whether the Windows or Linux Application Health extension will be installed.|
| `<your vm scale set name>` | The name of the virtual machine scale set in which you want to install VM watch. |

## Prerequisites

### 1. Register the feature
  
Register for adopting VM watch by running the following commands via the Azure CLI:
  
```bash
az feature register --name VMWatchPreview --namespace Microsoft.Compute --subscription <your subscription id>
az provider register --namespace Microsoft.Compute --subscription <your subscription id>
```

---

#### Validate feature registration

Validate that you successfully registered for the VM watch feature by running the following command:

```bash
az feature show --namespace Microsoft.Compute --name VMWatchPreview --subscription <your subscription id>
```

---

### 2. Ensure that a VM is installed

For information on how to create a VM and/or virtual machine scale set, see the [quickstart guide for Windows](/azure/virtual-machines/windows/quick-create-portal) and the [quickstart guide for Linux](/azure/virtual-machines/linux/quick-create-portal?tabs=ubuntu).

> [!IMPORTANT]
> If the Application Health extension is already installed on the VM, ensure that the settings `autoUpgradeMinorVersion` and `enableAutomaticUpgrade` are set to `true`.

## Install VM watch on an Azure virtual machine

> [!IMPORTANT]
> The code segment is identical for both Windows and Linux, except for the value of the parameter `<application health extension type>` passed in to the extension type. Replace `<application health extension type>` with `"ApplicationHealthLinux"` for Linux installations and `"ApplicationHealthWindows"` for Windows installations.

#### [CLI](#tab/cli-1)

```bash
az vm extension set --resource-group <your resource group> --vm-name <your vm name> --name <application health extension type> --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable-auto-upgrade true 
```

#### [PowerShell](#tab/powershell-1)

```powershell
Set-AzVMExtension -ResourceGroupName "<your resource group>" -Location "<your vm region>" -VMName "<your vm name>" -Name "<your extension name>" -Publisher "Microsoft.ManagedServices" -ExtensionType "<application health extension type>" -TypeHandlerVersion "2.0" -Settings @{"vmWatchSettings" = @{"enabled" = $True}} -EnableAutomaticUpgrade $True 
```

#### [ARM template](#tab/ARM-template-1)

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "apiVersion": "2021-04-01",
  "name": "[concat('<your vm name>', '/', '<your extension name')]",
  "location": "<your vm region>",
  "dependsOn": [
    "[resourceId('Microsoft.Compute/virtualMachines',parameters('<your vm name>'))]"
  ],
  "properties": {
    "publisher": "Microsoft.ManagedServices",
    "type": "ApplicationHealthWindows",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "vmWatchSettings": {
        "enabled": true
      }
    }
  }
}
```

#### [Bicep](#tab/Bicep-1)

```bicep
resource extension 'Microsoft.Compute/virtualMachines/extensions@2022-11-01' = {
  name: '<your vm name>/<your extension name>'
  location: location
  properties: {
    publisher: 'Microsoft.ManagedServices'
    type: 'ApplicationHealthWindows'
    typeHandlerVersion: '2.0'
    settings: {
      vmWatchSettings: {
        enabled: true
      }
    }
  }
  dependsOn: [
    '<your vm>'
  ]
}
```

---
### Validate that the Application Health VM extension is installed on the Azure VM

Go to the [Azure portal](https://portal.azure.com) and confirm that the Application Health VM extension was successfully installed.

The following screenshot shows a Windows installation.

:::image type="content" source="./media/vm-watch/windows-vm-watch-virtual-machine.png" alt-text="Screenshot that shows a Windows VM installation of the Application Health extension." lightbox="./media/vm-watch/windows-vm-watch-virtual-machine.png":::

The following screenshot shows a Linux installation.

:::image type="content" source="./media/vm-watch/linux-vm-watch-virtual-machine.png" alt-text="Screenshot that shows a Linux VM installation of the Application Health extension."lightbox="./media/vm-watch/linux-vm-watch-virtual-machine.png":::

To confirm that VM watch was enabled on this VM, go back to the overview page and select the JSON view for the VM. Ensure that the configuration exists in the JSON.

```json
"settings": {  
    "vmWatchSettings": {  
        "enabled": true  
    }
}
```

You can also use the Azure CLI or Azure PowerShell.

#### [CLI](#tab/cli-2)

```bash
az vm extension show --resource-group '<your resource group name>' --vm-name 'your vm name>' --name '<your extension name>'
```

#### [PowerShell](#tab/powershell-2)

```powershell
Get-AzVmExtension -ResourceGroupName '<your resource group name>' -VMName 'your vm name>' -Name '<your extension name>'
```

---

## Install VM watch on an Azure virtual machine scale set

> [!IMPORTANT]
> The code segment is identical for both Windows and Linux, except for the value of the parameter `<application health extension type>` passed in to the extension type. Replace `<application health extension type>` with `"ApplicationHealthLinux"` for Linux installations and `"ApplicationHealthWindows"` for Windows installations.

#### [CLI](#tab/cli-3)

```bash
az vmss extension set --resource-group '<your resource group name>' --vmss-name '<your vm scale set name>' --name <application health extension type> --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable-auto-upgrade true
```

#### [PowerShell](#tab/powershell-3)

```powershell
### Define the scale set variables 
$vmScaleSetName = "<your vm scale set name>" 
$vmScaleSetResourceGroup = "<your resource group name>" 

### Define the setting to enable VM watch 
$publicConfig = @{"vmWatchSettings" = @{"enabled" = $true}} 
$extensionName = "<your extension name>" 
$extensionType = "<application health extension type>" 
$publisher = "Microsoft.ManagedServices" 

### Get the scale set object 
$vmScaleSet = Get-AzVmss ` 
  -ResourceGroupName $vmScaleSetResourceGroup ` 
  -VMScaleSetName $vmScaleSetName

### Add the Application Health extension to the scale set model 
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet ` 
  -Name $extensionName ` 
  -Publisher $publisher ` 
  -Setting $publicConfig ` 
  -Type $extensionType ` 
  -TypeHandlerVersion "2.0" ` 
  -EnableAutomaticUpgrade $True

### Update the scale set 
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup ` 
  -Name $vmScaleSetName ` 
  -VirtualMachineScaleSet $vmScaleSet

### Upgrade instances to install the extension 
Update-AzVmssInstance -ResourceGroupName $vmScaleSetResourceGroup ` 
  -VMScaleSetName $vmScaleSetName ` 
  -InstanceId '*' 
```

#### [ARM template](#tab/ARM-template-3)

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "apiVersion": "2024-07-01",
  "name": "<your vm scale set name>",
  "location": "<your vm region>",
  "properties": {
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat(variables('<your vm scale set name>'), '/', '<your extension name>')]",
            "properties": {
              "publisher": "Microsoft.ManagedServices",
              "type": "<application health extension type>",
              "typeHandlerVersion": "2.0",
              "autoUpgradeMinorVersion": true,
              "enableAutomaticUpgrade": true,
              "settings": {
                "vmWatchSettings": {
                  "enabled": true
                }
              }
            }
          }
        ]
      }
    }
  }
}    
```

#### [Bicep](#tab/Bicep-3)

```bicep
resource extension 'Microsoft.Compute/virtualMachineScaleSets@2024-07-01' = {
  name: '<your vm scale set name>'
  location: '<your vm region>'
  properties: {
    virtualMachineProfile: {
      extensionProfile: {
        extensions: [
          {
            name: '${_your_vm_scale_set_name_}/<your extension name>'
            properties: {
              publisher: 'Microsoft.ManagedServices'
              type: '<application health extension type>'
              typeHandlerVersion: '2.0'
              autoUpgradeMinorVersion: true
              enableAutomaticUpgrade: true
              settings: {
                vmWatchSettings: {
                  enabled: true
                }
              }
            }
          }
        ]
      }
    }
  }
}
```

---

### Validate that the Application Health VM extension is installed in the virtual machine scale set

Go to the [Azure portal](https://portal.azure.com) and confirm that the Application Health VM extension was successfully installed.

The following screenshot shows a Windows installation.

:::image type="content" source="./media/vm-watch/windows-vm-watch-vm-scale-sets.png" alt-text="Screenshot that shows installation of the Application Health extension in a Windows virtual machine scale set."lightbox="./media/vm-watch/windows-vm-watch-vm-scale-sets.png":::

The following screenshot shows a Linux installation.

:::image type="content" source="./media/vm-watch/linux-vm-watch-vm-scale-sets.png" alt-text="Screenshot that shows installation of the Application Health extension in a Linux virtual machine scale set."lightbox="./media/vm-watch/linux-vm-watch-vm-scale-sets.png":::

---

To confirm that VM watch was enabled on this scale set, go back to the overview page and select the JSON view for the scale set. Ensure that the configuration exists in the JSON.

```json
"settings": {  
    "vmWatchSettings": {  
        "enabled": true  
    }
}
```

You can also use the Azure CLI or Azure PowerShell.

#### [CLI](#tab/cli-3)

```bash
az vmss extension show --resource-group '<your resource group name>' --vmss-name 'your vm scale set name>' --name '<your extension name>'
```

#### [PowerShell](#tab/powershell-3)

```powershell
(Get-AzVmss -ResourceGroupName '<your resource group name>' -VMScaleSetName 'your vm scale set name>').ExtensionProfile.Extensions
```

## Related content

- [Application Health extension](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension?tabs=azure-cli)
- [Azure CLI](/cli/azure/install-azure-cli)
- [Azure portal](https://portal.azure.com/)
