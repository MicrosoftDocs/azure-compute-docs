---
title: Copy snapshot of managed disk to subscription (Windows) - PowerShell
description: Azure PowerShell Script Sample -  Copy (move) snapshot of a managed disk to same or different subscription
author: roygara
ms.custom: devx-track-azurepowershell
ms.service: azure-disk-storage
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.date: 03/01/2023
ms.author: rogarana
# Customer intent: As a cloud administrator, I want to copy snapshots of managed disks between subscriptions, so that I can optimize storage costs and enhance data retention strategies.
---

# Copy snapshot of a managed disk in same subscription or different subscription with PowerShell (Windows)

This script copies a snapshot of a managed disk to same or different subscription. Use this script for the following scenarios:

1. Migrate a snapshot in Premium storage (Premium_LRS) to Standard storage (Standard_LRS or Standard_ZRS) to reduce your cost.
1. Migrate a snapshot from locally redundant storage (Premium_LRS, Standard_LRS) to zone redundant storage (Standard_ZRS) to benefit from the higher reliability of ZRS storage.
1. Move a snapshot to different subscription in the same region for longer retention.

[!INCLUDE [sample-powershell-install](../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

 

## Sample script

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-snapshot-to-same-or-different-subscription/copy-snapshot-to-same-or-different-subscription.ps1 "Copy snapshot")]

## Script explanation

This script uses following commands to create a snapshot in the target subscription using the Id of the source snapshot. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [New-AzSnapshotConfig](/powershell/module/az.compute/new-azsnapshotconfig) | Creates snapshot configuration that is used for snapshot creation. It includes the resource Id of the parent snapshot and location that is same as the parent snapshot.  |
| [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) | Creates a snapshot using snapshot configuration, snapshot name, and resource group name passed as parameters. |

## Next steps

[Create a virtual machine from a snapshot](./virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

For more information on the Azure PowerShell module, see [Azure PowerShell documentation](/powershell/azure/).

Additional virtual machine PowerShell script samples can be found in the [Azure Windows VM documentation](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
