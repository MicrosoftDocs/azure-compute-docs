---
title: Run scripts in an Azure Linux VM
description: This topic describes how to run scripts within a virtual machine
services: automation
ms.service: azure-virtual-machines
ms.custom: linux-related-content
ms.collection: linux
author: bobbytreed
ms.author: robreed
ms.date: 05/02/2018
ms.topic: how-to
# Customer intent: "As a system administrator managing Azure Linux VMs, I want to run scripts for configuration and troubleshooting, so that I can automate tasks and resolve issues efficiently, even when network access is limited."
---
# Run scripts in your Linux VM

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

To automate tasks or troubleshoot issues, you may need to run commands in a VM. The following article gives a brief overview of the features that are available to run scripts and commands within your VMs.

## Custom Script Extension

The [Custom Script Extension](../extensions/custom-script-linux.md) is primarily used for post deployment configuration and software installation.

* Download and run scripts in Azure virtual machines.
* Can be run using Azure Resource Manager templates, Azure CLI, REST API, PowerShell, or Azure portal.
* Script files can be downloaded from Azure storage or GitHub, or provided from your PC when run from the Azure portal.
* Run PowerShell script in Windows machines and Bash script in Linux machines.
* Useful for post deployment configuration, software installation, and other configuration or management tasks.

## Run command

The [Run Command](run-command.md) feature enables virtual machine and application management and troubleshooting using scripts, and is available even when the machine is not reachable, for example if the guest firewall doesn't have the RDP or SSH port open.

* Run scripts in Azure virtual machines.
* Can be run using [Azure portal](run-command.md), [REST API](../windows/run-command.md), [Azure CLI](/cli/azure/vm/run-command#az-vm-run-command-invoke), or [PowerShell](/powershell/module/az.compute/invoke-azvmruncommand)
* Quickly run a script and view output and repeat as needed in the Azure portal.
* Script can be typed directly or you can run one of the built-in scripts.
* Run PowerShell script in Windows machines and Bash script in Linux machines.
* Useful for virtual machine and application management and for running scripts in virtual machines that are unreachable.

## Hybrid Runbook Worker

The [Hybrid Runbook Worker](/azure/automation/automation-hybrid-runbook-worker) provides general machine, application, and environment management with user's custom scripts stored in an Automation account.

* Run scripts in Azure and non-Azure machines.
* Can be run using Azure portal, Azure CLI, REST API, PowerShell, webhook.
* Scripts stored and managed in an Automation Account.
* Run PowerShell, PowerShell Workflow, Python, or Graphical runbooks
* No time limit on script run time.
* Multiple scripts can run concurrently.
* Full script output is returned and stored.
* Job history available for 90 days.
* Scripts can run as Local System or with user-supplied credentials.
* Requires [manual installation](/azure/automation/automation-windows-hrw-install)

## Serial console

The [Serial console](/troubleshoot/azure/virtual-machines/serial-console-linux) provides direct access to a VM, similar to having a keyboard connected to the VM.

* Run commands in Azure virtual machines.
* Can be run using a text-based console to the machine in the Azure portal.
* Login to the machine with a local user account.
* Useful when access to the virtual machine is needed regardless of the machine's network or operating system state.

## Next steps

Learn more about the different features that are available to run scripts and commands within your VMs.

* [Custom Script Extension](../extensions/custom-script-linux.md)
* [Run Command](run-command.md)
* [Hybrid Runbook Worker](/azure/automation/automation-hybrid-runbook-worker)
* [Serial console](/troubleshoot/azure/virtual-machines/serial-console-linux)
