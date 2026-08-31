---
title: Check whether a VM is hosted in Azure public cloud from the guest operating system
description: Learn how to use the SMBIOS chassis asset tag to check whether a virtual machine is hosted in Azure public cloud from inside the guest operating system without network access.
author: mattmcinnes
ms.author: mattmcinnes
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 08/14/2026
ai-usage: ai-assisted
# Customer intent: "As a system administrator, I want to check whether a server is hosted in Azure public cloud from inside the guest operating system without relying on network access."
---

# Check whether a VM is hosted in Azure public cloud from the guest operating system

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs

To check whether a virtual machine (VM) is hosted in Azure public cloud, read its System Management BIOS (SMBIOS) chassis asset tag from inside the guest operating system. Azure public cloud VMs expose the following value:

```text
7783-7084-3265-9085-8269-3286-77
```

You can compare the chassis asset tag to this value to determine whether a system reports itself as an Azure public cloud VM. The check reads information that Azure projects into the guest, so it doesn't require network access.

> [!IMPORTANT]
> This value applies only to Azure public cloud. Don't use it to identify Azure Local or Azure sovereign clouds.

## Check the chassis asset tag on Linux

Read the chassis asset tag from the kernel Desktop Management Interface (DMI) sysfs file:

```bash
cat /sys/class/dmi/id/chassis_asset_tag
```

For an Azure public cloud VM, the command returns:

```output
7783-7084-3265-9085-8269-3286-77
```

This method works across Linux distributions without root permissions or extra packages, because the kernel exposes the `chassis_asset_tag` field through sysfs.

Alternatively, if you prefer `dmidecode`, run the following command with root permissions:

```bash
sudo dmidecode -s chassis-asset-tag
```

This command returns the same value. If `dmidecode` isn't installed, install it by using your Linux distribution's package manager.

## Check the chassis asset tag on Windows

Run the following PowerShell command:

```powershell
(Get-CimInstance -ClassName Win32_SystemEnclosure).SMBIOSAssetTag
```

For an Azure public cloud VM, the command returns:

```output
7783-7084-3265-9085-8269-3286-77
```

The `SMBIOSAssetTag` property is part of the `Win32_SystemEnclosure` class.

## Choose between the chassis asset tag and IMDS

Use the chassis asset tag when you only need to detect an Azure public cloud VM and can't depend on network access. The tag doesn't provide details about the VM.

Use the [Azure Instance Metadata Service (IMDS)](instance-metadata-service.md) when you need richer information, such as the VM ID, size, location, or network configuration. IMDS is a REST API available from within the VM and requires guest network access to its endpoint.

IMDS also distinguishes which Azure environment hosts the VM through the `azEnvironment` field. Query it directly to tell an Azure public cloud VM apart from an Azure Stack VM.

On Linux, run:

```bash
curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2025-04-07&format=text"
```

On Windows, run:

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2025-04-07&format=text"
```

For an Azure public cloud VM, the request returns:

```output
AZUREPUBLICCLOUD
```

A VM hosted on Azure Stack returns a different value, such as `AzureStack`. Unlike the chassis asset tag, `azEnvironment` identifies the specific cloud environment rather than only confirming that the VM runs on Azure.

> [!CAUTION]
> The chassis asset tag is readable from the guest operating system. Use it for platform detection only, not as proof of identity, an authentication or authorization credential, or a security boundary.
