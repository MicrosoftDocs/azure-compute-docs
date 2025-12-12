---
title: Run scripts in a Windows VM in Azure using managed Run Commands
description: This topic describes how to run scripts within an Azure Windows virtual machine by using the updated Run Command feature.
services: automation
ms.service: azure-virtual-machines
ms.collection: windows
author: GabstaMSFT
ms.author: cynthn
ms.date: 08/18/2025
ms.topic: how-to
ms.custom: devx-track-azurepowershell, devx-track-azurecli
# Customer intent: As an IT administrator, I want to execute scripts on Azure Windows VMs using managed Run Commands, so that I can efficiently manage and troubleshoot machine and application issues.
---
# Run scripts in your Windows VM by using managed Run Commands

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

The Run Command feature uses the virtual machine (VM) agent to execute scripts within an Azure Windows VM. You can use these scripts for general machine or application management. They can help you quickly diagnose and remediate VM access and network issues and get the VM back to a good state.

The *updated* managed Run Command uses the same VM agent channel to execute scripts and provides the following enhancements over the [original action orientated Run Command](run-command.md): 
- Support for updated Run Command through ARM deployment template 
- Parallel execution of multiple scripts 
- Sequential execution of scripts   
- User specified script time-out 
- Support for long running (hours/days) scripts 
- Passing secrets (parameters, passwords) in a secure manner

> [!IMPORTANT]
> **Managed Run Command**  is currently available in Azure CLI, PowerShell, and API at this time.

## Prerequisites

### **Windows Operating Systems Supported**
| OS Version | x64 | ARM64 |
|:-----|:-----:|:-----:|
| Windows 10 | Supported | Supported |
| Windows 11 | Supported | Supported |
| Windows Server 2016 | Supported | Supported |
| Windows Server 2016 Core | Supported | Supported |
| Windows Server 2019 | Supported | Supported |
| Windows Server 2019 Core | Supported | Supported |
| Windows Server 2022 | Supported | Supported |
| Windows Server 2022 Core | Supported | Supported |
| Windows Server 2022 Azure Edition | Supported | Supported |
| Windows Server 2025 | Supported | Supported |
| Windows Server 2025 Core | Supported | Supported |
| Windows Server 2025 Azure Edition| Supported | Supported |


## Available Commands
| **CommandID** | **Description** | **More Info** |
|:-----|:-----|:-----|
| **RunPowerShellScript** | Runs a PowerShell script ||
| **DisableNLA** | Disables Network Level Authentication (NLA). You must restart the VM after the script completes for the change to take effect. The script itself does not restart the VM. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/DisableNLA) |
| **DisableWindowsUpdate** | Disable Automatic Updates through Windows Update. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/DisableWindowsUpdate) |
EnableWindowsUpdate) |
| **EnableAdminAccount** | Checks if the local Administrator account is disabled, and if so enables it. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/EnableAdminAccount) |
| **EnableEMS** | Enable Emergency Management Services (EMS) to allow for serial console connection in troubleshooting scenarios. | [readme](https://github.com/Azure/
| **EnableRemotePS** | Enable remote PowerShell. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/EnableRemotePS) |
| **EnableWindowsUpdate** | Enable Automatic Updates through Windows Update. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/|
azure-support-scripts/blob/master/RunCommand/Windows/EnableEMS) |
| **IMDSCertCheck** | Checks IMDS Health and then analyzes currently installed certificates that IMDS depends upon. If missing, shows additional details and mitigation steps. | [readme](https://aka.ms/AzVmIMDSValidation) |
| **IPConfig** | Shows detailed information for the IP address, subnet mask and default gateway for each adapter bound to TCP/IP. For usage, refer to [Run scripts](/azure/virtual-machines/windows/run-command-managed#create-or-update-run-command-on-a-vm-using-sourcecommandid) | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/IPConfig) |
| **RDPSettings** | Checks registry settings and domain policy settings. Suggests policy actions if machine is part of a domain or modifies the settings to default values. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/RDPSettings) |
| **ResetRDPCert** | Removes the SSL certificate tied to the RDP listener and restores the RDP listener security to default. Use this script if you see any issues with the certificate. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/ResetRDPCert) |
| **SetRDPPort** | Sets the default or user specified port number for Remote Desktop connections. Enables firewall rule for inbound access to the port. | [readme](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/SetRDPPort) |
| **WindowsActivationValidation** | Checks the current Windows license status (activated or not) and alert is generated if the machine is not property activated. | [readme](https://aka.ms/AzVmWinActivationValidationTool) |
| **WindowsGhostedNicValidationScript** | This script scans the Windows registry for network adapters on PCI and VMBUS buses, compares them with currently active network adapters, and identifies ghosted NICs. Useful for troubleshooting network issues or cleaning up old NICs. | [readme](https://aka.ms/AzVmGhostedNicCleanup) |
| **WindowsUpgradeAssessmentValidation** | This script is designed to assess the readiness of a Windows machine (client or server) for an in-place OS upgrade, with special considerations for Azure Virtual Machines. It evaluates OS version, supported upgrade paths, system disk space, and Azure security features like Trusted Launch, Secure Boot, and vTPM. | [readme](https://aka.ms/AzVmOSUpgradeAssessment) |

> [!Note]
> The above command IDs are applicable to Managed Run Commands for Windows VMs. You can list all commandIds using command  "```Get-AzVMRunCommandDocument -Location {RegionName}```".

## Limiting access to Run Command

Listing the run commands or showing the details of a command requires the `Microsoft.Compute/locations/runCommands/read` permission on Subscription Level. The built-in [Reader](/azure/role-based-access-control/built-in-roles#reader) role and higher levels have this permission.

Running a command requires the `Microsoft.Compute/virtualMachines/runCommand/write` permission. The [Virtual Machine Contributor](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) role and higher levels have this permission.

You can use one of the [built-in roles](/azure/role-based-access-control/built-in-roles) or create a [custom role](/azure/role-based-access-control/custom-roles) to use Run Command.

## Azure CLI 

The following examples use [az vm run-command](/cli/azure/vm/run-command) to run shell script on an Azure Windows VM.

### Execute a script with the VM
This command delivers the script to the VM, execute it, and return the captured output.

```azurecli-interactive
az vm run-command create --name "myRunCommand" --vm-name "myVM" --resource-group "myRG" --script "Write-Host Hello World!"
```

### List all deployed RunCommand resources on a VM 
This command returns a full list of previously deployed Run Commands along with their properties.

```azurecli-interactive
az vm run-command list --vm-name "myVM" --resource-group "myRG"
```

### Get execution status and results 
This command retrieves current execution progress, including latest output, start/end time, exit code, and terminal state of the execution.

```azurecli-interactive
az vm run-command show --name "myRunCommand" --vm-name "myVM" --resource-group "myRG" --expand instanceView
```

> [!Note]
> Output and error fields in `instanceView` is limited to last 4KB.
> If you'd like to access the full output and error, you have the option of forwarding the output and error data to storage append blobs using `-outputBlobUri` and `-errorBlobUri` parameters while executing Run Command using `Set-AzVMRunCommand` or `Set-AzVMssRunCommand`.

### Delete RunCommand resource from the VM
Remove the RunCommand resource previously deployed on the VM. If the script execution is still in progress, execution is terminated. 

```azurecli-interactive
az vm run-command delete --name "myRunCommand" --vm-name "myVM" --resource-group "myRG"
```


## PowerShell 

### Execute a script with the VM
This command delivers the script to the VM, execute it, and return the captured output.

```powershell-interactive
Set-AzVMRunCommand -ResourceGroupName "myRG" -VMName "myVM" -Location "EastUS" -RunCommandName "RunCommandName" –SourceScript "echo Hello World!"
```
### Execute a script on the VM using SourceScriptUri parameter 
`OutputBlobUri` and `ErrorBlobUri` are optional parameters.

```powershell-interactive
Set-AzVMRunCommand -ResourceGroupName "myRg" `
-VMName "myVM" `
-RunCommandName "RunCommandName" `
-SourceScriptUri “<SAS_URI_of_a_storage_blob_with_read_access_or_public_URI>" `
-OutputBlobUri “<SAS_URI_of_a_storage_append_blob_with_read_add_create_write_access>" `
-ErrorBlobUri “<SAS_URI_of_a_storage_append_blob_with_read_add_create_write_access>”
```

### Execute a long running Run Command on a VM
The execution time should run longer than default timeout of 90 minutes. It should run for 100 minutes (refer 6,0000-seconds -TimeoutInSecond parameter) provided script is expected to run that long. After 100 minutes, the script execution should stop.

```powershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG -VMName MyVM -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri> -AsyncExecution -TimeoutInSecond 6000
```

### List all deployed RunCommand resources on a VM 
This command returns a full list of previously deployed Run Commands along with their properties.

```powershell-interactive
Get-AzVMRunCommand -ResourceGroupName "myRG" -VMName "myVM"
```

### Get execution status and results 
This command retrieves current execution progress, including latest output, start/end time, exit code, and terminal state of the execution.

```powershell-interactive
Get-AzVMRunCommand -ResourceGroupName "myRG" -VMName "myVM" -RunCommandName "RunCommandName" -Expand InstanceView
```

### Create or update Run Command on a VM using SourceScriptUri (storage blob SAS URL)
Create or update Run Command on a Windows VM using a SAS URL of a storage blob that contains a PowerShell script. `SourceScriptUri` can be a storage blob’s full SAS URL or public URL.

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVMEE -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri>
```

> [!Note]
> SAS URL must provide read access to the blob. An expiration time of 24 hours is suggested for SAS URL. SAS URLs can be generated on Azure portal using blob's options , or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, your SAS URL = "base blob URL" + "?" + "SAS token from `New-AzStorageBlobSASToken`"

### Get a Run Command Instance View for a VM after Creating or Updating Run Command
Get a Run Command for VM with Instance View. Instance View contains execution state of run command (Succeeded, Failed, etc.), exit code, standard output, and standard error generated by executing the script using Run Command. A nonzero ExitCode indicates an unsuccessful execution.

```azurepowershell-interactive
$x = Get-AzVMRunCommand -ResourceGroupName MyRG -VMName MyVM -RunCommandName MyRunCommand -Expand InstanceView
$x.InstanceView
```
Example output
```azurepowershell-interactive
ExecutionState   : Succeeded
ExecutionMessage :
ExitCode         : 0
Output           :   
output       : uid=0(root) gid=0(root) groups=0(root)
                   HelloWorld

Error            :
StartTime        : 10/27/2022 9:10:52 PM
EndTime          : 10/27/2022 9:10:55 PM
Statuses         :

```

`InstanceView.ExecutionState`: Status of user's Run Command script. Refer this state to know whether your script was successful or not.
`ProvisioningState`: Status of general extension provisioning end to end (whether extension platform was able to trigger Run Command script or not). 


### Create or update Run Command on a VM using ScriptLocalPath (local script file)
Create or update Run Command on a VM using a local script file that is on the client machine where cmdlet is executed.

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVMEE -RunCommandName MyRunCommand -Location EastUS2EUAP -ScriptLocalPath "C:\MyScriptsDir\MyScript.ps1"
```


### Create or update Run Command on a VM using SourceScript (script text)

Create or update Run Command on a VM passing the script content directly to -SourceScript parameter. Use `;` to separate multiple commands

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVML -RunCommandName MyRunCommand2 -Location EastUS2EUAP -SourceScript "id; echo HelloWorld"
```

### Create or update Run Command on a VM using SourceCommandId
Create or update Run Command on a VM using preexisting commandId. Available commandIds can be retrieved using [Get-AzVMRunCommandDocument](/powershell/module/az.compute/get-azvmruncommanddocument).

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVMEE -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceCommandId DisableWindowsUpdate
```
### Create, or update Run Command on a VM using OutputBlobUri, ErrorBlobUri to stream standard output and standard error messages to output, and error Append blobs
Create or update Run Command on a VM and stream standard output and standard error messages to output and error Append blobs.

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVML -RunCommandName MyRunCommand3 -Location EastUS2EUAP -ScriptLocalPath "C:\MyScriptsDir\MyScript.ps1" -OutputBlobUri <OutPutBlobUrI> -ErrorBlobUri "ErrorBlobUri
```

>[!NOTE]
> Output and error blobs must be the AppendBlob type and their SAS URLs must provide read, append, create, write access to the blob. An expiration time of 24 hours is suggested for SAS URL. If output or error blob does not exist, a blob of type AppendBlob is created. SAS URLs can be generated on Azure portal using blob's options , or SAS token from using `New-AzStorageBlobSASToken`.

### Create or update Run Command on a VM as a different user using RunAsUser and RunAsPassword parameters
Create or update Run Command on a VM as a different user using `RunAsUser` and `RunAsPassword` parameters. For RunAs to work properly, contact admin of VM, and make sure user is added on the VM, user has access to resources accessed by the Run Command (Directories, Files, Network etc.), and in case of Windows VM, 'Secondary Logon' service is running on the VM.

```azurepowershell-interactive
Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVMEE -RunCommandName MyRunCommand -Location EastUS2EUAP -ScriptLocalPath "C:\MyScriptsDir\MyScript.ps1" -RunAsUser myusername -RunAsPassword mypassword
```

### Create or update Run Command on a Virtual Machine Scale Sets resource using SourceScriptUri (storage blob SAS URL)
Create or update Run Command on a Windows Virtual Machine Scale Sets resource using a SAS URL of a storage blob that contains a PowerShell script.

```azurepowershell-interactive
Set-AzVmssVMRunCommand -ResourceGroupName MyRG0 -VMScaleSetName MyVMSS -InstanceId 0 -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri>
```

>[!Note]
> SAS URL must provide read access to the blob. An expiry time of 24 hours is suggested for SAS URL. SAS URLs can be generated on Azure portal using blob's options, or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, the SAS URL format is: base blob URL + "?" + the SAS token from `New-AzStorageBlobSASToken`.

### Create or update Run Command on a VM instance using Parameter and ProtectedParameter parameters (Public and Protected Parameters to script)
Use ProtectedParameter to pass any sensitive inputs to script such as passwords, keys etc.

```azurepowershell-interactive
$PublicParametersArray = @([Microsoft.Azure.PowerShell.Cmdlets.Compute.Models.Api20210701.IRunCommandInputParameter]@{name='publicParam1';value='publicParam1value'},
>> [Microsoft.Azure.PowerShell.Cmdlets.Compute.Models.Api20210701.IRunCommandInputParameter]@{name='publicParam2';value='publicParam2value'})

$ProtectedParametersArray = @([Microsoft.Azure.PowerShell.Cmdlets.Compute.Models.Api20210701.IRunCommandInputParameter]@{name='secret1';value='secret1value'},
>> [Microsoft.Azure.PowerShell.Cmdlets.Compute.Models.Api20210701.IRunCommandInputParameter]@{name='secret2';value='secret2value'})

Set-AzVMRunCommand -ResourceGroupName MyRG0 -VMName MyVMEE -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri> -Parameter $PublicParametersArray -ProtectedParameter $ProtectedParametersArray
```
- Windows: Parameters and ProtectedParameters are passed to script as arguments are passed to script and run like this - `myscript.ps1 -publicParam1 publicParam1value -publicParam2 publicParam2value -secret1 secret1value -secret2 secret2value`

- Linux: Named Parameters and its values are set to environment config, which should be accessible within the .sh script. For Nameless arguments, pass an empty string to name input. Nameless arguments are passed to script and run like this - `myscript.sh publicParam1value publicParam2value secret1value secret2value`


### Delete RunCommand resource from the VM
Remove the RunCommand resource previously deployed on the VM. If the script execution is still in progress, execution is terminated. 

```powershell-interactive
Remove-AzVMRunCommand -ResourceGroupName "myRG" -VMName "myVM" -RunCommandName "RunCommandName"
```
 

## REST API 

To deploy a new Run Command, execute a PUT on the VM directly and specify a unique name for the Run Command instance. 

```rest
PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/runcommands/<runCommandName>?api-version=2023-03-01
```

```json
{ 
"location": "<location>", 
"properties": { 
    "source": { 
        "script": "Write-Host Hello World!", 
        "scriptUri": "<SAS URI of a storage blob with read access or public URI>",  
        "commandId": "<Id>"  
        }, 
    "parameters": [ 
        { 
            "name": "param1",
            "value": "value1" 
            }, 
        { 
            "name": "param2", 
            "value": "value2" 
            } 
        ], 
    "protectedParameters": [ 
        { 
            "name": "secret1", 
            "value": "value1" 
            }, 
        { 
            "name": "secret2", 
            "value": "value2" 
            } 
        ], 
    "runAsUser": "userName",
    "runAsPassword": "userPassword", 
    "timeoutInSeconds": 3600,
    "treatFailureAsDeploymentFailure": true,
    "outputBlobUri": "< SAS URI of a storage append blob with read, add, create, write access>", 
    "errorBlobUri": "< SAS URI of a storage append blob with read, add, create, write access >"  
    }
}
```

### Notes
 
- You can provide an inline script, a script URI, or a built-in script [command ID](run-command.md#available-commands) as the input source. Script URI is either storage blob SAS URI with read access or public URI.
- Only one type of source input is supported for one command execution.
- Starting with API version 2023-03-01, you can set the property `treatFailureAsDeploymentFailure` to **true** causing the deployment to fail when there is a failure in the script. If set to **false**, ProvisioningState would only reflect whether the run command was run or not by the extensions platform. It would not indicate whether the script failed in case of script failures. 
- Run Command supports writing output and error to Storage blobs using outputBlobUri and errorBlobUri parameters, which can be used to store large script outputs. Use SAS URI of a storage append blob with read, add, create, write access. The blob should be of type AppendBlob. Writing the script output or error blob would fail otherwise. The blob will be overwritten if it already exists. It will be created if it does not exist.


### List running instances of Run Command on a VM 

```rest
GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/runcommands?api-version=2023-03-01
``` 

### Get output details for a specific Run Command deployment 

```rest
GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/runcommands/<runCommandName>?$expand=instanceView&api-version=2023-03-01
```

### Delete a specific Run Command deployment

You can also delete the instance of Run Command.  

```rest
DELETE /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/runcommands/<runCommandName>?api-version=2023-03-01
```

### Deploy scripts in an ordered sequence 

To deploy scripts sequentially, use a deployment template, specifying a `dependsOn` relationship between sequential scripts. 

```json
{ 
    "type": "Microsoft.Compute/virtualMachines/runCommands", 
    "name": "secondRunCommand", 
    "apiVersion": "2019-12-01", 
    "location": "[parameters('location')]", 
    "dependsOn": <full resourceID of the previous other Run Command>, 
    "properties": { 
        "source": {  
            "script": "Write-Host Hello World!"  
        }, 
        "timeoutInSeconds": 60  
    }
} 
```

### Execute multiple Run Commands sequentially 

By default, if you deploy multiple RunCommand resources using deployment template, they will be executed simultaneously on the VM. If you have a dependency on the scripts and a preferred order of execution, you can use the `dependsOn` property to make them run sequentially. 

In this example, **secondRunCommand** will execute after **firstRunCommand**. 

```json
{
   "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion":"1.0.0.0",
   "resources":[
      {
         "type":"Microsoft.Compute/virtualMachines/runCommands",
         "name":"[concat(parameters('vmName'),'/firstRunCommand')]",
         "apiVersion":"2023-03-01",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
         ],
         "properties":{
            "source":{
               "script":"Write-Host First: Hello World!"
            },
            "parameters":[
               {
                  "name":"param1",
                  "value":"value1"
               },
               {
                  "name":"param2",
                  "value":"value2"
               }
            ],
            "timeoutInSeconds":20
         }
      },
      {
         "type":"Microsoft.Compute/virtualMachines/runCommands",
         "name":"[concat(parameters('vmName'),'/secondRunCommand')]",
         "apiVersion":"2019-12-01",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'),'runcommands/firstRunCommand')]"
         ],
         "properties":{
            "source":{
               "scriptUri":"http://github.com/myscript.ps1"
            },
            "timeoutInSeconds":60
         }
      }
   ]
}
```


## Next steps

To learn about other ways to run scripts and commands remotely in your VM, see [Run scripts in your Windows VM](run-scripts-in-vm.md).
