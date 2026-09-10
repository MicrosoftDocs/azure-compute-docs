---
title: Swap between OS disks using the Azure CLI
description: Change the operating system disk used by an Azure virtual machine using the Azure CLI.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 04/24/2018
ms.author: rogarana
ms.custom: devx-track-azurecli, linux-related-content
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to swap the operating system disk of an Azure virtual machine using the CLI, so that I can easily update or recover the VM without needing to delete and recreate it."
---
# Change the OS disk used by an Azure VM using the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

If you have an existing VM, but you want to swap the disk for a backup disk or another OS disk, you can use the Azure CLI to swap the OS disks. You don't have to delete and recreate the VM. You can even use a managed disk in another resource group, as long as it isn't already in use.

## OS disk swap requirements

You don't need to stop the VM before you swap its OS disk.

Before you swap the OS disk, ensure that the current and replacement disks meet these requirements:

- The VM size supports the storage type of the replacement disk. For example, a replacement disk in Premium Storage requires a VM size that supports Premium Storage, such as a DS-series size.
- The current and replacement OS disks are the same size.
- The VM and replacement OS disk have compatible encryption configurations. Swapping an encrypted OS disk into an unencrypted VM isn't supported.
- If the VM doesn't use Azure Disk Encryption, the replacement OS disk must not use Azure Disk Encryption.
- If the disks use disk encryption sets, both disks belong to the same disk encryption set.

This article requires Azure CLI version 2.0.25 or greater. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI]( /cli/azure/install-azure-cli). 

## Swap the OS disk by using the Azure CLI

Set the VM resource group, replacement OS disk resource group, VM name, and full replacement OS disk resource ID variables. Replace the example values with your own values.

```azurecli-interactive
vmResourceGroup="myVMResourceGroup"
diskResourceGroup="myDiskResourceGroup"
vmName="myVM"
replacementDiskId="/subscriptions/<subscription ID>/resourceGroups/$diskResourceGroup/providers/Microsoft.Compute/disks/myDisk"
```

Use [az disk list](/cli/azure/disk) to get a list of the disks in your resource group.

```azurecli-interactive
az disk list \
   -g $diskResourceGroup \
   --query '[*].{diskId:id}' \
   --output table
```


(Optional) Before swapping the disks, use [az vm stop](/cli/azure/vm#az-vm-stop) to stop the VM.

```azurecli-interactive
az vm stop \
   -n $vmName \
   -g $vmResourceGroup
```


Use [az vm update](/cli/azure/vm#az-vm-update) with the full resource ID of the new disk for the `--os-disk` parameter.

```azurecli-interactive 
az vm update \
   -g $vmResourceGroup \
   -n $vmName \
   --os-disk $replacementDiskId
```

Verify that the VM uses the replacement OS disk. The command output should match the value of `$replacementDiskId`.

```azurecli-interactive
az vm show \
   -g $vmResourceGroup \
   -n $vmName \
   --query storageProfile.osDisk.managedDisk.id \
   --output tsv
```

If you stopped the VM before swapping the disks, restart it by using [az vm start](/cli/azure/vm).

```azurecli-interactive
az vm start \
   -n $vmName \
   -g $vmResourceGroup
```

## Next steps

To create a copy of a disk, see [Snapshot a disk](snapshot-copy-managed-disk.md).
