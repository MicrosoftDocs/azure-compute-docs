---
title: Convert Audit Logs to an Allowlist
description: Learn more about audit logs with Metadata Security Protocol (MSP).
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: As a cloud administrator, I want to convert audit logs into an allowlist, so that I can define access control rules effectively and enhance the security of my virtual machine environment.
---

# Convert audit logs to an allowlist

With Metadata Security Protocol (MSP), you can define a custom role-based access control (RBAC) allowlist to help secure metadata service endpoints. The contents of the allowlist come from audit logs. A new resource type in Azure Compute Gallery, `InVMAccessControlProfile`, enables the allowlist.

To learn more about RBAC and the `InVMAccessControlProfile` resource type, see [Advanced configuration for MSP](../advanced-configuration.md).

## Structure of an allowlist

An allowlist consists of:

- **Identities**: Processes on the machine.
- **Privileges**: Endpoints that the identities access.
- **Roles**: A grouping of privileges.
- **Role assignments**: Roles and the list of identities granted access for those roles.

## Collect audit logs

#### If you enable MSP in `audit` or `enforce` mode, you can collect log data from virtual machine client via [Azure Monitor.](/azure/azure-monitor/vm/data-collection) 
 
-	Windows: [Azure monitoring agent](/azure/azure-monitor/vm/data-collection-windows-events)  collects Windows Events via Custom XPath ```Windows Azure!*[System[Provider[@Name=`GuestProxyAgent`]]]```

- Linux: Collect Syslog events with [Azure Monitor Agent](/azure/azure-monitor/vm/data-collection-syslog) by selecting `Log_DAEMON` and `LOG_DEBUG`

## Query audit logs

Once the audit logs are collected  as explained in the previous section, you can verify the logs:

Windows Kusto Query: [Go to Log Analytics and run query](https://ms.portal.azure.com/#@72f988bf-86f1-41af-91ab-2d7cd011db47/blade/Microsoft_OperationsManagementSuite_Workspace/Logs.ReactView/resourceId/%2fsubscriptions%2fa53f7094-a16c-47af-abe4-b05c05d0d79a%2fresourcegroups%2fmayankdaruka-logs-rg%2fproviders%2fmicrosoft.operationalinsights%2fworkspaces%2flog-workspace1/source/LogsBlade.AnalyticsShareLinkToQuery/q/H4sIAAAAAAAAA3MtS80r4apRKM9ILUpVCMnMTXVPzUstSixJTVGwS0zP1zBM0YRLu4IU%252B%252BSnK9jaKiiFZ%252Bal5JcXKzhWlRalKikk5qUoBOeXFiWngmXdS1OLSwKK8isqHdOBmpS4ABekrZVpAAAA)
```kusto
/// Windows VMs
Event
| where TimeGenerated >ago(1d)
| where EventLog == "Windows Azure" and Source == "GuestProxyAgent"
| where _ResourceId startswith "/subscriptions/<your subscription id>/resourcegroups/<your recrouce group>" 
| where RenderedDescription  has  "processFullPath" and RenderedDescription  has "runAsElevated" and RenderedDescription has "processCmdLine"
| extend json = parse_json(RenderedDescription)
| extend method = json.method, url = json.url, processFullPath = json.processFullPath, username = json.userName, runAsElevated = json.runAsElevated 
| extend userGroups = json.userGroups, ip = json.ip, port = json.port, processCmdLine = json.processCmdLine 
| project TimeGenerated, _ResourceId, url, ip, port, processFullPath, method, username, runAsElevated, processCmdLine
```
Linux Kusto Query: [Go to Log Analytics and run query](https://ms.portal.azure.com/#@72f988bf-86f1-41af-91ab-2d7cd011db47/blade/Microsoft_OperationsManagementSuite_Workspace/Logs.ReactView/resourceId/%2fsubscriptions%2fa53f7094-a16c-47af-abe4-b05c05d0d79a%2fresourcegroups%2fmayankdaruka-logs-rg%2fproviders%2fmicrosoft.operationalinsights%2fworkspaces%2flog-workspace1/source/LogsBlade.AnalyticsShareLinkToQuery/q/H4sIAAAAAAAAA3MtS80r4apRKM9ILUpVCMnMTXVPzUstSixJTVGwS0zP1zBM0YRLu4IU%252B%252BSnK9jaKiiFZ%252Bal5JcXKzhWlRalKikk5qUoBOeXFiWngmXdS1OLSwKK8isqHdOBmpS4ABekrZVpAAAA)

```kusto
/// Linux VMs
Syslog
| where TimeGenerated >ago(1d)
| where  ProcessName == "azure-proxy-agent"
| where _ResourceId startswith "/subscriptions/<your subscription id>/resourcegroups/<your recrouce group>" 
| where SyslogMessage  has  "processFullPath" and SyslogMessage  has "runAsElevated" and SyslogMessage has "processCmdLine"
| extend message = substring(SyslogMessage, indexof(SyslogMessage, "{"))
| extend json = parse_json(message)
| extend method = json.method, url = json.url, processFullPath = json.processFullPath, username = json.userName, runAsElevated = json.runAsElevated 
| extend userGroups = json.userGroups, ip = json.ip, port = json.port, processCmdLine = json.processCmdLine 
| project TimeGenerated, _ResourceId, url, ip, port, processFullPath, method, username, runAsElevated, processCmdLine
```
### If you own the Azure VM, you can get the file logs inside the Azure VM by following these steps

1. Find the Proxy Agent json config file:

    - Windows VM: `GuestProxyAgent.json` under the folder as `GuestProxyAgent windows service`
    - Linux VM:  `etc/azure/proxy-agent.json` file 
2. Turn on the file log by updating setting `logFolder` in the json config file. Set it to
    - `%SYSTEMDRIVE%\\WindowsAzure\\ProxyAgent\\Logs` for Windows VMs 
    - `/var/log/azure-proxy-agent` for Linux VMs.
1. Restart Service
- `GuestProxyAgent` for Windows VMs
-  `azure-proxy-agent` for Linux VMs4

4. Wait for `ProxyAgent.Connection.log` to populate while customer services is running.
1. Pull the ProxyAgent connection file logs from the Azure VMs
    - Windows: `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log`
    
    - Linux: `/var/log/azure-proxy-agent/ProxyAgent.Connection.log`

## Convert logs to rules

To create an allowlist, you can use an automated method or a manual method.

### Generate an allowlist automatically

You can use an allowlist generator tool to generate the access control rules. The tool helps parse the audit logs and provides a UI to generate the rules.

1. Download and run the allowlist generator tool. On the [latest release page](https://github.com/Azure/GuestProxyAgent/releases/latest), under **Assets**, select `allowListTool.exe`.

   The tool parses the `ProxyAgentConnection` logs and displays the current privileges and identities on the VM.

1. Create roles and role assignments:

   - To create a role, select a grouping of privileges and give the role a descriptive name.
   - To create a role assignment, select a role and a grouping of identities. These identities can access the privileges grouped in that role. Give the role assignment a descriptive name.

### Manually create an allowlist

After you enable a VM with MSP in `Audit` or `Enforce` mode, the proxy agent captures all the requests being made to the host endpoints.

In the connection logs, you can analyze the applications that are making the requests to the Azure Instance Metadata Service or WireServer endpoints.

[![Screenshot of connection logs.](../images/create-shared-image-gallery/status-log.png)](../images/create-shared-image-gallery/status-log.png#lightbox)

The following example shows the format of the captured JSON.

[![Screenshot of audit logs with captured JSON.](../images/create-shared-image-gallery/parse-json-from-logs.png)](../images/create-shared-image-gallery/parse-json-from-logs.png#lightbox)

In the log file, you can identify the endpoints that you want to secure. These endpoints appear in `privileges` in the final `InVMAccessControlProfile` instance. You can also identify the identities (`identities`) that should have access.

A simple rules schema might look like the following example.

[![Screenshot of a simple rules schema.](../images/create-shared-image-gallery/example-access-control-rules.png)](../images/create-shared-image-gallery/example-access-control-rules.png#lightbox)

## Create an InVMAccessControlProfile instance by using an ARM template

1. [Create a new private gallery](/azure/virtual-machines/create-gallery) in Azure Compute Gallery.

1. Create an `InVMAccessControlProfile` definition with parameters for:

    - Gallery name to store in (from step 1)
    - Profile name
    - OS type
    - Host endpoint type (WireServer or Instance Metadata Service)

1. Create a specific version.

## Sample InVMAccessControlProfile

Here's a sample `InVMAccessControlProfile` instance:

```
"properties": {
    "mode": "Enforce",
    "defaultAccess": "Allow",
    "rules": {
      "privileges": [
        {
          "name": "GoalState",
          "path": "/machine",
          "queryParameters": {
            "comp": "goalstate"
          }
        }
      ],
      "roles": [
        {
          "name": "Provisioning",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "ManageGuestExtensions",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "MonitoringAndSecret",
          "privileges": [
            "GoalState"
          ]
        }
      ],
      "identities": [
        {
          "name": "WinPA",
          "userName": "SYSTEM",
          "exePath": "C:\\Windows\\System32\\cscript.exe"
        },
        {
          "name": "GuestAgent",
          "userName": "SYSTEM",
          "processName": "WindowsAzureGuestAgent.exe"
        },
        {
          "name": "WaAppAgent",
          "userName": "SYSTEM",
          "processName": "WaAppAgent.exe"
        },
        {
          "name": "CollectGuestLogs",
          "userName": "SYSTEM",
          "processName": "CollectGuestLogs.exe"
        },
        {
          "name": "AzureProfileExtension",
          "userName": "SYSTEM",
          "processName": "AzureProfileExtension.exe"
        },
        {
          "name": "AzurePerfCollectorExtension",
          "userName": "SYSTEM",
          "processName": "AzurePerfCollectorExtension.exe"
        },
        {
          "name": "WaSecAgentProv",
          "userName": "SYSTEM",
          "processName": "WaSecAgentProv.exe"
        }
      ],
      "roleAssignments": [
        {
          "role": "Provisioning",
          "identities": [
            "WinPA"
          ]
        },
        {
          "role": "ManageGuestExtensions",
          "identities": [
            "GuestAgent",
            "WaAppAgent",
            "CollectGuestLogs"
          ]
        },
        {
          "role": "MonitoringAndSecret",
          "identities": [
            "AzureProfileExtension",
            "AzurePerfCollectorExtension",
            "WaSecAgentProv"
          ]
        }
      ]
    },
```



### Using PowerShell

If you're using PowerShell to generate an `InVMAccessControlProfile`, ensure that you have the minimum version of 10.1.0 of [PowerShell](https://www.powershellgallery.com/packages/Az.Compute/10.1.0)

Follow the step-by-step guide below to generate an `InVMAccessControlProfile`:

1. Log in to your Azure Account


```powershell
Connect-AzAccount 
```

2. Create the Resource Group where the private gallery is created. You can skip this step if you already have a Resource Group created.

```powershell
$resourceGroup = "MyResourceGroup4" 
$location = "EastUS2EUAP" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
```
3. Create a private gallery. This gallery is used as a container for the `InVMAccessControlProfile` artifact

```powershell
$galleryName = "MyGallery4" 
New-AzGallery -ResourceGroupName $resourceGroup -GalleryName $galleryName -Location $location -Description "My custom image gallery" 
```

4. Create the `InVMAccessControlProfile` artifact in the private gallery created in the previous step. [Click here](/powershell/module/az.compute/new-azgalleryinvmaccesscontrolprofile?view=azps-14.3.0) to learn more about the various parameters for this artifact. 

```powershell
$InVMAccessControlProfileName= "testInVMAccessControlProfileP"  

New-AzGalleryInVMAccessControlProfile -ResourceGroupName  $resourceGroup  -GalleryName $galleryName   -GalleryInVMAccessControlProfileName $InVMAccessControlProfileName -Location $location -OsType "Windows" -ApplicableHostEndPoint "WireServer" -Description "this test1" 
```
5. Get Gallery `InVMAccessControlProfile`

```powershell
$inVMAccessCP=Get-AzGalleryInVMAccessControlProfile -ResourceGroupName  $resourceGroup  -GalleryName $galleryName   -GalleryInVMAccessControlProfileName $InVMAccessControlProfileName 
```
![Screenshot of the output for Get command for InVMAccessControlProfile.](../images/

6. Update Gallery `InVMAccessControlProfile`
Once the `InVMAccessControlProfile` is created, the only attribute editable is the description. For any other changes, create a new artifact. 

To update the description:

```powershell
Update-AzGalleryInVMAccessControlProfile -ResourceGroupName  $resourceGroup  -GalleryName $galleryName   -GalleryInVMAccessControlProfileName $InVMAccessControlProfileName -Location $location -Description "this test2"
```

![Screenshot of Update InVMAccessControlProfile description. ](../images/create-shared-image-gallery/update-description-invmaccesscontrolprofile.png)

  
7. Create `InVMAccessControlProfileVersion`

To create an InVMAccessControlProfileVersion, a payload is required. Since these payloads can be large, especially due to the rules property, it's not practical to use a single PowerShell command to create the entire resource in one go.
The rules property in any version payload consists of four arrays: privileges, roles, identities, and roleAssignments. These arrays can make the payload large and complex.
To simplify this process, we introduced the GalleryInVMAccessControlProfileVersionConfig PowerShell object. You can learn more about it [here.](/powershell/module/az.compute/new-azgalleryinvmaccesscontrolprofileversionconfig?view=azps-14.3.0)

This object allows you to incrementally build the configuration using various commands to add or remove rule properties.
Once the configuration object is ready, you can use it to create an `InVMAccessControlProfileVersion`,  described in the upcoming sections.

Alternatively, if you already have the rules property as a JSON string and prefer not to use the configuration object, you can skip these steps and create the InVMAccessControlProfileVersion using an ARM template deployment, which is also covered later in the section.

Payload for reference:

```json
{
  "name": "1.0.0",
  "location": "East US 2 EUAP",
  "properties": {
    "mode": "Audit", 
    "defaultAccess": "Deny", 
    "rules": { 
      "privileges": [
        {
          "name": "GoalState",
          "path": "/machine",
          "queryParameters": {
            "comp": "goalstate"
          }
        }
      ],
      "roles": [
        {
          "name": "Provisioning",
          "privileges": [
            "GoalState"
          ]
        }
      ],
      "identities": [
        {
          "name": "WinPA",
          "userName": "SYSTEM",
          "groupName": "Administrators",
          "exePath": "C:\\Windows\\System32\\cscript.exe",
          "processName": "cscript"
        }
      ],
      "roleAssignments": [
        {
          "role": "Provisioning",
          "identities": [
            "WinPA"
          ]
        }
      ]
    },
    "targetLocations": [
      {
        "name": "East US 2 EUAP"
      }
    ],
    "excludeFromLatest": false 
  }
}
```
8. Create `InVMAccessControlProfileVersion` Config

```powershell
$inVMAccessControlProfileVersionName= "1.0.0"
$targetRegions= @("EastUS2EUAP", "CentralUSEUAP")
$inVMAccessConrolProfileVersion = New-AzGalleryInVMAccessControlProfileVersionConfig `
 -Name $inVMAccessControlProfileVersionName  `
-Location $location  `
-Mode "Audit"  `
-DefaultAccess "Deny" -TargetLocation $targetRegions  -ExcludeFromLatest
```

![Screenshot of create profile version config.](../images/create-shared-image-gallery/create-profileversion-config.png)

Run this command to add each privilege:

```powershell
Add-AzGalleryInVMAccessControlProfileVersionRulesPrivilege `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -PrivilegeName "GoalState" `
  -Path "/machine" `
  -QueryParameter @{ comp = "goalstate" }
 ```

 To remove a privilege:

 ```powershell
 Remove-AzGalleryInVMAccessControlProfileVersionRulesPrivilege `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -PrivilegeName "GoalState2"
 ```

Run this command to add each role:

 ```powershell
Add-AzGalleryInVMAccessControlProfileVersionRulesRole `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -RoleName "Provisioning" `
  -Privilege @("GoalState")
 ```

 Run this command to remove roles:

 ```powershell
 Remove-AzGalleryInVMAccessControlProfileVersionRulesRole `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -RoleName "Provisioning2" 
```
Add `RulesIdentity`:

```powershell
Add-AzGalleryInVMAccessControlProfileVersionRulesIdentity `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -IdentityName "WinPA" `
  -UserName "SYSTEM" `
  -GroupName "Administrators" `
  -ExePath "C:\Windows\System32\cscript.exe" `
  -ProcessName "cscript"
```

Remove `RulesIdentity`:

```powershell
Remove-AzGalleryInVMAccessControlProfileVersionRulesIdentity `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -IdentityName "WinPA2" 
```

Run this command to add each Role Assignment:

```powershell
Add-AzGalleryInVMAccessControlProfileVersionRulesRoleAssignment `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -Role "Provisioning" `
  -Identity @("WinPA")
```

Remove Role Assignment:

```powershell
Remove-AzGalleryInVMAccessControlProfileVersionRulesRoleAssignment `
  -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion `
  -Role "Provisioning2" 
 ```

9. Create Gallery `InVMAccessControlProfileVersion`

 ```powershell
 New-AzGalleryInVMAccessControlProfileVersion -ResourceGroupName $resourceGroup -GalleryName $galleryName -GalleryInVMAccessControlProfileName   $InVMAccessControlProfileName   -GalleryInVmAccessControlProfileVersion $inVMAccessConrolProfileVersion 
 ```
 ![Screenshot of create InVMAccessControlProfileVersion.](../images/create-shared-image-gallery/create-gallery-invmaccesscontrolprofileversion.png)

10. Get `InVMAccessControlProfileVersion`

```powershell
$ver = Get-AzGalleryInVMAccessControlProfileVersion -ResourceGroupName $resourceGroup -GalleryName $galleryName -GalleryInVMAccessControlProfileName   $InVMAccessControlProfileName `
 -GalleryInVMAccessControlProfileVersionName  $inVMAccessControlProfileVersionName
 $ver | ConvertTo-Json -Depth 10
```
11. Update `InVMAccessControlProfileVersion`

It's recommended to create a new `InVMAccessControlProfileVersion` since most parameters can't be updated. Here's an example:

```powershell
$targetRegions= @("EastUS2EUAP")
 
$ver = Get-AzGalleryInVMAccessControlProfileVersion -ResourceGroupName $resourceGroup -GalleryName $galleryName -GalleryInVMAccessControlProfileName   $InVMAccessControlProfileName `
 -GalleryInVMAccessControlProfileVersionName  $inVMAccessControlProfileVersionName
 
 
Update-AzGalleryInVMAccessControlProfileVersion `
  -GalleryInVmAccessControlProfileVersion $ver `
-TargetLocation $targetRegions -ExcludeFromLatest $true
```
![Screenshot of update InVMAccessControlProfileVersion screenshot.](../images/create-shared-image-gallery/update-invmaccesscontrolprofileversion.png)

12. Delete `InVMAccessControlProfileVersion`

```powershell
Remove-AzGalleryInVMAccessControlProfileVersion -ResourceGroupName $resourceGroup -GalleryName $galleryName -GalleryInVMAccessControlProfileName   $InVMAccessControlProfileName `
 -GalleryInVMAccessControlProfileVersionName  $inVMAccessControlProfileVersionName
```

![Screenshot of delete InVMAccessControlProfileVersion example.](../images/create-shared-image-gallery/delete-invmaccesscontrolprofileversion.png)

13. List all gallery `InVMAccessControlProfile`

```powershell
Get-AzGalleryInVMAccessControlProfile -ResourceGroupName "myResourceGroup" -GalleryName "myGallery"
```

![Screenshot of List all InVMAccessControlProfiles.](../images/create-shared-image-gallery/get-list-invmaccesscontrolprofile.png)

### Using CLI

Follow the step-by-step guide below to generate an `InVMAccessControlProfile`:

1. Log in to your Azure Account


```cli
az login
```

2. Create a resource group in your desired location

```cli
az group create  --resource-group ResourceGroupForINVM  --location eastus
```

3. Create a gallery that will house the `InVMAccessControlProfile` resource

```cli
 az sig create --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --location eastus 
 ```

3. Create an `InVMAccessControlProfile` under the gallery created in the prior step
For additional details of commands: [az sig in-vm-access-control-profile | Microsoft Learn](/cli/azure/sig/in-vm-access-control-profile?view=azure-cli-latest)

```cli
 az sig in-vm-access-control-profile create --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --name myInVMAccessControlProfileName --location eastus --os-type Linux  --applicable-host-endpoint WireServer 
```

Run this command to see additional details about the commands and properties

```cli
az sig in-vm-access-control-profile create --help 
```

![Screenshot of getting details about InVMAccessControlProfile properties](../images/create-shared-image-gallery/details-cli.png)

4. Update the `InVMAccessControlProfile`  
You can only update the description of the `InVMAccessControlProfile`. If additional changes needed, delete the current `InVMAccessControlProfile` and create a new one

```cli
az sig in-vm-access-control-profile update --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --name myInVMAccessControlProfileName  --description test 
```
![Screenshot of updating description of InVMAccessControlProfile](../images/create-shared-image-gallery/update-cli.png)

5. Get operation for `InVMAccessControlProfile`

```cli
az sig in-vm-access-control-profile show --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --name myInVMAccessControlProfileName 
```
![Screenshot of get operation for InVMAccessControlProfile](../images/create-shared-image-gallery/get-cli.png)

6. List all `InVMAccessControlProfile` artifacts

This command shows the list of all `InVMAccessControlProfile` under a specific gallery

```cli
az sig in-vm-access-control-profile show --resource-group ResourceGroupForINVM --gallery-name MyGallery67 
```
![Screenshot of list command to show all `InVMAccessControlProfile` artifacts under a specific gallery](../images/create-shared-image-gallery/list-cli.png)

7. Create `InVMAccessControlProfileVersion`

You must provide a payload to create an `InVMAccessControlProfileVersion`. This payload can be large, especially because of the rule’s property, which may contain extensive configurations.
Instead of passing individual parts of the rules property directly, we’ve introduced a rules parameter that accepts a JSON file as an input. This approach simplifies the process and keeps the command clean and manageable.
Here is an example rules.json file:

```
 1. { 
 2.   "privileges": [
 3. 	{
 4. 	  "name": "GoalState",
 5. 	  "path": "/machine",
 6. 	  "queryParameters": {
 7. 		"comp": "goalstate"
 8. 	  }
 9. 	}
10.   ],
11.   "roles": [
12. 	{
13. 	  "name": "Provisioning",
14. 	  "privileges": [
15. 		"GoalState"
16. 	  ]
17. 	}
18.   ],
19.   "identities": [
20. 	{
21. 	  "name": "WinPA",
22. 	  "userName": "SYSTEM",
23. 	  "groupName": "Administrators",
24. 	  "exePath": "C:\\Windows\\System32\\cscript.exe",
25. 	  "processName": "cscript"
26. 	}
27.   ],
28.   "roleAssignments": [
29. 	{
30. 	  "role": "Provisioning",
31. 	  "identities": [
32. 		"WinPA"
33. 	  ]
34. 	}
35.   ]
36. } 
```

See addtional command details here: [az sig in-vm-access-control-profile-version | Microsoft Learn](/cli/azure/sig/in-vm-access-control-profile?view=azure-cli-latest)

Once you have create your version of rules.json file, use the following command to create the `InVMAccessControlProfileVersion`

```cli
 az sig in-vm-access-control-profile-version create \
   --resource-group ResourceGroupForINVM  \
   --gallery-name MyGallery67  \
   --profile-name myInVMAccessControlProfileName \
   --version-name 1.0.0 \
   --mode Audit \
   --default-access Deny  \
   --target-regions EastUS2EUAP  \
   --exclude-from-latest true \
   --rules @rules.json
```
8. Get `InVMAccessControlProfileVersion`

```cli
az sig in-vm-access-control-profile-version show --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --profile-name myInVMAccessControlProfileName --profile-version 1.0.0
```
![Screenshot of get command for profile version](../images/create-shared-image-gallery/version-get.png)

9. List all `InVMAccessControlProfileVersion` artifacts

```cli
az sig in-vm-access-control-profile-version list --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --profile-name myInVMAccessControlProfileName
```

10. Delete `InVMAccessControlProfileVersion`

Before you delete any `InVMAccessControlProfileVersion` ensure it is not being used by any `InVMAccessControlProfile` on any Virtual Machine or Vitual Machine Scale Sets.
```cli
az sig in-vm-access-control-profile-version delete --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --profile-name myInVMAccessControlProfileName --profile-version 1.0.0
```

11. Delete `InVMAccessControlProfile`

```cli
az sig in-vm-access-control-profile delete --resource-group ResourceGroupForINVM --gallery-name MyGallery67 --name myInVMAccessControlProfileName 
```

