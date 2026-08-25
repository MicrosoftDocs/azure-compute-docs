---
title: Create and encrypt a Windows VM by using Azure CLI
description: In this quickstart, you learn how to use Azure CLI to create and encrypt a Windows virtual machine.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: windows
ms.topic: quickstart
ms.date: 07/14/2026
ai-usage: ai-assisted
ms.custom: devx-track-azurecli, mode-api
# Customer intent: "As a cloud administrator, I want to create and encrypt a Windows virtual machine using command-line tools, so that I can enhance the security of my Azure resources efficiently."
---

# Quickstart: Create and encrypt a Windows VM by using the Azure CLI

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets

Use the Azure CLI to create and manage Azure resources from the command line or in scripts. This quickstart shows how to use the Azure CLI to create and encrypt a Windows Server 2016 virtual machine (VM).

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0.30 or later of the Azure CLI. If you use Azure Cloud Shell, the latest version is already installed.

## Create a resource group

Create a resource group with the [az group create](/cli/azure/group#az-group-create) command. An Azure resource group is a logical container into which Azure resources are deployed and managed. The following example creates a resource group named *myResourceGroup* in the *eastus* location:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## Create a virtual machine

Create a VM with [az vm create](/cli/azure/vm#az-vm-create). The following example creates a VM named *myVM*. The following example uses *azureuser* for an administrative user name and *myPassword12* as the password.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword12
```

It takes a few minutes to create the VM and supporting resources. The following example output shows a successful VM create operation.

```console
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

## Create a key vault configured for encryption keys

Azure Disk Encryption stores its encryption key in an Azure Key Vault. Create a key vault with [az keyvault create](/cli/azure/keyvault#az-keyvault-create). To enable the key vault to store encryption keys, use the --enabled-for-disk-encryption parameter.
> [!IMPORTANT]
> Each key vault must have a unique name. The following example creates a key vault named *myKV*, but you must name yours something different.

```azurecli-interactive
az keyvault create --name "myKV" --resource-group "myResourceGroup" --location eastus --enabled-for-disk-encryption
```

## Encrypt the virtual machine

Encrypt your VM with [az vm encryption](/cli/azure/vm/encryption), providing your unique key vault name to the --disk-encryption-keyvault parameter.

```azurecli-interactive
az vm encryption enable -g MyResourceGroup --name MyVM --disk-encryption-keyvault myKV
```

Verify that encryption is enabled on your VM with [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show)

```azurecli-interactive
az vm encryption show --name MyVM -g MyResourceGroup
```

You see the following in the returned output:

```console
"EncryptionOperation": "EnableEncryption"
```

## Clean up resources

When you no longer need these resources, use the [az group delete](/cli/azure/group) command to remove the resource group, VM, and Key Vault.

```azurecli-interactive
az group delete --name myResourceGroup
```

## Next steps

In this quickstart, you created a virtual machine, created a key vault that was enabled for encryption keys, and encrypted the VM. Go to the next article to learn more about Azure Disk Encryption prerequisites for IaaS VMs.

> [!div class="nextstepaction"]
> [Azure Disk Encryption overview](disk-encryption-overview.md)
