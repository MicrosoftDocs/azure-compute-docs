---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 06/23/2020
 ms.author: rogarana
 ms.custom: include file, devx-track-azurepowershell
# Customer intent: As a cloud administrator, I want to retrieve the encryption type of a specified disk in a resource group, so that I can ensure compliance with security standards for our virtual machines.
---
```PowerShell
$ResourceGroupName="yourResourceGroupName"
$DiskName="yourDiskName"

$disk=Get-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName
$disk.Encryption.Type
```
