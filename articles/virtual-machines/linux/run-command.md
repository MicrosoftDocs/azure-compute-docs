---
title: Run scripts in a Linux VM in Azure using action Run Commands
description: This article describes how to run scripts within an Azure Linux virtual machine by using the Run Command feature
services: automation
ms.service: azure-virtual-machines
ms.collection: linux
author: GabstaMSFT
ms.author: jushiman
ms.date: 08/18/2025
ms.topic: how-to
ms.reviewer: jushiman
ms.custom: devx-track-azurecli, linux-related-content
ms.devlang: azurecli
# Customer intent: "As a system administrator managing Azure Linux VMs, I want to run shell scripts using the Run Command feature, so that I can efficiently troubleshoot and manage my virtual machines without needing remote access."
---
# Run scripts in your Linux VM by using action Run Commands

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

The Run Command feature uses the virtual machine (VM) agent to run shell scripts within an Azure Linux VM. You can use these scripts for general machine or application management. They can help you to quickly diagnose and remediate VM access and network issues and get the VM back to a good state.

## Benefits

You can access your virtual machines in multiple ways. Run Command can run scripts on your virtual machines remotely by using the VM agent. You use Run Command through the Azure portal, [REST API](/rest/api/compute/virtual-machine-run-commands), or [Azure CLI](/cli/azure/vm/run-command#az-vm-run-command-invoke) for Linux VMs.

This capability is useful in all scenarios where you want to run a script within a virtual machine. It's one of the only ways to troubleshoot and remediate a virtual machine that doesn't have the RDP or SSH port open because of network or administrative user configuration.

## Prerequisites

### Linux Distro’s Supported
| Publisher | Distribution | x64 | ARM64 |
|:-----|:-----|:-----:|:-----:|
| Alma Linux Community | Alma Linux | 8.x+, 9.x+ | 8.x+, 9.x+ |
| Credativ | Debian | 10+ | 11.x+ |
| Kinvolk | Flatcar Linux | 3374.2.x+ | 3374.2.x+ |
| Microsoft | Azure Linux | 2.x | 2.x |
| openSUSE Project | openSUSE | 12.3+ | *Not supported* |
| Oracle | Oracle Linux | 6.4+, 7.x+, 8.x+ | *Not supported* |
| Red Hat | Red Hat Enterprise Linux | 6.7+, 7.x+,  8.x+, 9.x+, 10.x+ | 8.6+, 9.0+, 10.x+ |
| CIQ | Rocky Linux | 9.x+ | 9.x+ |
| SUSE | SLES | 12.x+, 15.x+ | 15.x SP4+ |
| Canonical | Ubuntu (LTS releases)| 18.04+, 20.04+, 22.04+, 24.04+ | 20.04+, 22.04+, 24.04+ |

## Restrictions

The following restrictions apply when you're using Run Command:

* Output is limited to the last 4,096 bytes.
* The minimum time to run a script is about 20 seconds.
* Scripts run by default as an elevated user on Linux.
* You can run one script at a time.
* Scripts that prompt for information (interactive mode) aren't supported.
* You can't cancel a running script.
* The maximum time a script can run is 90 minutes. After that, the script will time out.
* Outbound connectivity from the VM is required to return the results of the script.

> [!NOTE]
> To function correctly, Run Command requires connectivity (port 443) to Azure public IP addresses. If the extension doesn't have access to these endpoints, the scripts might run successfully but not return the results. If you're blocking traffic on the virtual machine, you can use [service tags](/azure/virtual-network/network-security-groups-overview#service-tags) to allow traffic to Azure public IP addresses by using the `AzureCloud` tag.

## Available commands

This table shows the list of commands available for Linux VMs. You can use the **RunShellScript** command to run any custom script that you want. When you're using the Azure CLI or PowerShell to run a command, the value that you provide for the `--command-id` or `-CommandId` parameter must be one of the following listed values. When you specify a value that isn't an available command, you receive this error:

```error
The entity was not found in this Azure location
```

|**Name**|**Description**|**More Info**|
|---|---|---|
|**RunShellScript**|Runs a Linux shell script.||
|**ifconfig**| Gets the configuration of all network interfaces.|[readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Linux/ifconfig/readme.md)|

## Azure CLI

The following example uses the [az vm run-command](/cli/azure/vm/run-command#az-vm-run-command-invoke) command to run a shell script on an Azure Linux VM.

```azurecli-interactive
az vm run-command invoke -g myResourceGroup -n myVm --command-id RunShellScript --scripts "apt-get update && apt-get install -y nginx"
```

> [!NOTE]
> To run commands as a different user, enter `sudo -u` to specify a user account.

## Azure portal

Go to a VM in the [Azure portal](https://portal.azure.com) and select **Run command** in the left menu, under **Operations**. You see a list of the available commands to run on the VM.

![List of commands](./media/run-command/run-command-list.png)

Choose a command to run. Some of the commands might have optional or required input parameters. For those commands, the parameters are presented as text fields for you to provide the input values. For each command, you can view the script that's being run by expanding **View script**. **RunShellScript** is different from the other commands, because it allows you to provide your own custom script.

> [!NOTE]
> The built-in commands are not editable.

After you choose the command, select **Run** to run the script. After the script finishes, it returns the output and any errors in the output window. The following screenshot shows an example output from running the **ifconfig** command.

![Run command script output](./media/run-command/run-command-script-output.png)

### PowerShell

The following example uses the [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand) cmdlet to run a PowerShell script on an Azure VM. The cmdlet expects the script referenced in the `-ScriptPath` parameter to be local to where the cmdlet is being run.

```powershell-interactive
Invoke-AzVMRunCommand -ResourceGroupName '<myResourceGroup>' -Name '<myVMName>' -CommandId 'RunShellScript' -ScriptPath '<pathToScript>' -Parameter @{"arg1" = "var1";"arg2" = "var2"}
```

## Limiting access to Run Command

Listing the run commands or showing the details of a command requires the `Microsoft.Compute/locations/runCommands/read` permission on Subscription level. The built-in [Reader](/azure/role-based-access-control/built-in-roles#reader) role and higher levels have this permission.

Running a command requires the `Microsoft.Compute/virtualMachines/runCommands/write` permission. The [Virtual Machine Contributor](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) role and higher levels have this permission.

You can use one of the [built-in roles](/azure/role-based-access-control/built-in-roles) or create a [custom role](/azure/role-based-access-control/custom-roles) to use Run Command.

## Action Run Command Linux troubleshooting

When troubleshooting action run command for Linux environments, refer to the *handler* log file typically located in the following directory: `/var/log/azure/run-command-handler/handler.log` for further details.

### Known issues
The Linux action run command logs have a few notable differences compared to the action run command Windows logs:

- The sequence number is reported with each line of the log as 'seq=#'
- There won't be a line that contains `Awaiting completion...` as this will be in action run command Windows only.
- The line `Command existed with code: #` is also only present in action run command Windows logging.

### Action Run Command Removal

If needing to remove your action run command Linux extension, refer to the below steps for Azure PowerShell and CLI:

 Replace *rgname* and *vmname* with your relevant resource group name and virtual machine name in the following removal examples.


```powershell-interactive
 Invoke-AzVMRunCommand -ResourceGroupName 'rgname' -VMName 'vmname' -CommandId 'RemoveRunCommandLinuxExtension'
```

```azurecli-interactive
az vm run-command invoke  --command-id RemoveRunCommandLinuxExtension --name vmname -g rgname
```
> [!NOTE]
> When you apply a Run Command again, the extension will get installed automatically. You can use the extension removal command to troubleshoot any issues related to the extension.

## Next steps

To learn about other ways to run scripts and commands remotely in your VM, see [Run scripts in your Linux VM](run-scripts-in-vm.md).
