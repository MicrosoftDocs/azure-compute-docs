---
title: Explore Azure Hybrid Benefit for Windows VMs
description: Learn how to maximize your Windows Software Assurance benefits to bring on-premises licenses to Azure.
ms.service: azure-virtual-machines
ms.subservice: billing
ms.collection: windows
ms.topic: how-to
ms.date: 12/02/2024
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.devlang: azurecli
# Customer intent: As a cloud administrator, I want to leverage Azure Hybrid Benefit for Windows Server licenses to deploy and manage Windows VMs, so that I can optimize licensing costs while ensuring compliance and maximizing resource efficiency in our cloud environment.
---
# Explore Azure Hybrid Benefit for Windows VMs

Maximize your on-premises core licenses for Windows Server to get Windows virtual machines (VMs) on Azure at a reduced cost through Azure Hybrid Benefit for Windows Server. You also can use Azure Hybrid Benefit for Windows Server to deploy new VMs that run the Windows OS.

This article describes the steps to deploy new VMs with Azure Hybrid Benefit for Windows Server and how to update existing running VMs.

To qualify for Azure Hybrid Benefit for Windows Server, you need on-premises core licenses for Windows Server from an applicable program with active Software Assurance or qualifying subscription licenses. Software Assurance and qualifying subscription licenses are available only as part of certain commercial licensing agreements. To learn more about commercial licensing, see [Microsoft licensing resources](https://www.microsoft.com/licensing/default). To learn more about Windows Server core licenses, see [Windows Server product licensing](https://www.microsoft.com/licensing/product-licensing/windows-server).

You can use Azure Hybrid Benefit for Windows Server with any VMs running Windows Server OS in all regions, including VMs that have additional software, such as SQL Server or third-party Azure Marketplace software.

## Limitations

To use Azure Hybrid Benefit for Windows Server, you must have a minimum of 8 core licenses (Datacenter edition or Standard edition) per VM. For example, even if you run a 4-core instance, 8 core licenses are required. You also can run instances larger than 8 cores by allocating licenses equal to the core size of the instance. For example, 12 core licenses are required for a 12-core instance.

For customers who have processor licenses, each processor license is equivalent to 16 core licenses.

> [!IMPORTANT]
>
> - Workloads that use Azure Hybrid Benefit for Windows Server can run only during the Software Assurance or subscription license term. When the Software Assurance or subscription license term approaches expiration, you must renew your agreement with either Software Assurance or a subscription license, disable the Azure Hybrid Benefit for Windows Server functionality, or deprovision workloads that use Azure Hybrid Benefit for Windows Server.
>
> - The Microsoft Product Terms for your program take precedence over the information that's presented in this article. For more information, see [Microsoft Azure Product Terms](https://www.microsoft.com/licensing/terms/productoffering/MicrosoftAzure) and select your program to show the terms.

## Classic VMs

For classic VMs, the only supported option is deploying a new VM from an on-premises custom image. To take advantage of the Azure Hybrid Benefit for Windows Server capabilities that this article describes, you must first migrate classic VMs to Azure Resource Manager model VMs.

[!INCLUDE [classic-vm-deprecation](../includes/classic-vm-deprecation.md)]

## How to use Azure Hybrid Benefit for Windows Server

You have several options to use Windows virtual machines with Azure Hybrid Benefit for Windows Server. You can:

- Deploy VMs from one of the provided Windows Server images on Azure Marketplace.
- Upload a custom VM and deploy by using an Azure Resource Manager template or Azure PowerShell.
- Toggle and convert an existing VM between from running with Azure Hybrid Benefit or the pay-on-demand cost for Windows Server.
- Apply Azure Hybrid Benefit for Windows Server on a virtual machine scale set.

## Create a VM that uses Azure Hybrid Benefit for Windows Server

All Windows Server OS-based images are supported for Azure Hybrid Benefit for Windows Server. You can use Azure platform-supported images or upload your own custom Windows Server image.

### Azure portal

To create a VM that uses Azure Hybrid Benefit for Windows Server, when you create your VM, on the **Basics** tab under **Licensing**, select the checkbox to use an existing Windows Server license.

### Azure PowerShell

```azurepowershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -ImageName "Win2016Datacenter" `
    -LicenseType "Windows_Server"
```

### Azure CLI

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --license-type Windows_Server
```

### Resource Manager template

In your Resource Manager template, set the `licenseType` parameter. For more information, see [Authoring Azure Resource Manager templates](/azure/azure-resource-manager/templates/syntax).

```json
"properties": {
    "licenseType": "Windows_Server",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
}    
```

## Convert an existing VM to use Azure Hybrid Benefit for Windows Server

To convert an existing VM to use Azure Hybrid Benefit for Windows Server, update your VM's license type.

> [!NOTE]
> Changing the license type on the VM doesn't cause the system to restart, and service is not interrupted. The process changes a metadata licensing flag only.
>

### Azure portal

On the VM service menu, select **Operating system**, and then set **Azure Hybrid Benefit** to **Enable**.

### Azure PowerShell

- To convert an existing Windows Server VM to Azure Hybrid Benefit for Windows Server:

    ```azurepowershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "Windows_Server"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```

- To convert a Windows Server VM that uses Azure Hybrid Benefit for Windows Server back to pay-as-you-go:

    ```azurepowershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "None"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```

### Azure CLI

- To convert an existing Windows Server VM to use Azure Hybrid Benefit for Windows Server:

    ```azurecli
    az vm update --resource-group myResourceGroup --name myVM --set licenseType=Windows_Server
    ```

### Verify that your VM uses the licensing benefit

After you deploy your VM by using either Azure PowerShell, a Resource Manager template, or the Azure portal, you can verify the setting by using one of the following methods.

### Azure portal

On the VM service menu, select **Operating system**, and then view the Azure Hybrid Benefit for Windows Server setting.

### Azure PowerShell

The following example shows the license type for a single VM:

```azurepowershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Output:

```azurepowershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Server
```

The output contrasts with the following VM that's deployed *without* Azure Hybrid Benefit for Windows Server licensing:

```azurepowershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

### Azure CLI

```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVM --query "[?licenseType=='Windows_Server']" -o table
```

> [!NOTE]
> Changing the license type on the VM doesn't cause the system to restart, and service is not interrupted. The process changes a metadata licensing flag only.

## List all resources that use Azure Hybrid Benefit for Windows Server

To view and get a count of all your VMs and virtual machine scale sets that have Azure Hybrid Benefit for Windows Server enabled, you can use the following options for your subscription.

### Azure portal

On the VM or virtual machine scale sets overview pane, get a list of all your VMs and licensing types by setting the table columns to include **OS licensing benefit**. The VM might have the state **Azure Hybrid Benefit for Windows**, **Not enabled**, or **Windows client with multi-tenant hosting**.

### Azure PowerShell

For VMs:

```azurepowershell
Get-AzVM | ?{$_.LicenseType -like "Windows_Server"} | select ResourceGroupName, Name, LicenseType
```

For virtual machine scale sets:

```azurepowershell
Get-AzVmss | Select * -ExpandProperty VirtualMachineProfile | ? LicenseType -eq 'Windows_Server' | select ResourceGroupName, Name, LicenseType
```

### Azure CLI

For VMs:

```azurecli
az vm list --query "[?licenseType=='Windows_Server']" -o table
```

For virtual machine scale sets:

```azurecli
az vmss list --query "[?virtualMachineProfile.licenseType=='Windows_Server']" -o table
```

## Deploy a virtual machine scale set to use Azure Hybrid Benefit for Windows Server

Within your virtual machine scale set Resource Manager templates, the `licenseType` parameter must be set in your `VirtualMachineProfile` property. You can set this parameter when you create or update for your virtual machine scale set by using a Resource Manager template, Azure PowerShell, the Azure CLI, or REST API.

The following example uses a Resource Manager template with a Windows Server 2016 Datacenter image:

```json
"virtualMachineProfile": {
    "storageProfile": {
        "osDisk": {
            "createOption": "FromImage"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
        }
    },
    "licenseType": "Windows_Server",
    "osProfile": {
            "computerNamePrefix": "[parameters('vmssName')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
    }
}    
```

For more information, see [Modify a virtual machine scale set](../../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md).

## Related content

- [Save money with Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/)
- [FAQ for Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/faq/)
- [Azure Hybrid Benefit for Windows Server licensing detailed guidance](/windows-server/get-started/azure-hybrid-benefit)
- [Azure Hybrid Benefit for Windows Server and Azure Site Recovery make migrating applications to Azure even more cost-effective](https://azure.microsoft.com/blog/hybrid-use-benefit-migration-with-asr/)
- [Deploy Windows 11 on Azure with Multitenant Hosting Rights](./windows-desktop-multitenant-hosting-deployment.md)
- [Using Resource Manager templates](/azure/azure-resource-manager/management/overview)
