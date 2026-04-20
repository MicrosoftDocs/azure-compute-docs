---
title: Manage, Update, and Delete VM Applications on Azure
description: Learn how to manage, update, and delete VM applications on Azure Virtual Machine (VM) and Virtual Machine Scale Sets using Azure Compute Gallery.
author: tanmaygore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 08/14/2025
ms.author: tagore
ms.reviewer: jushiman
ms.custom: devx-track-azurepowershell, devx-track-azurecli
---

# Manage Azure VM Applications

This article talks about how to view, update, and delete published VM Application in Azure Compute Gallery. It then talks about how to view, monitor, update, and delete deployed VM application resource on Azure Virtual Machine (VM) or Virtual Machine Scale Sets.

## Managed VM Application published in Azure Compute Gallery
This section explains about how to view, update, and delete **published VM Applications** in Azure Compute Gallery. 

## View published VM Applications

#### [Portal](#tab/portal1)
To view the properties of a published VM Application in the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Azure Compute Gallery**.
3. Select the gallery that contains your VM Application. 
4. Click the **VM Application Name** you want to view.
5. The **Overview/Properties** blade displays information about the VM Application.
6. The **Overview/Versions** blade displays all published versions and its basic properties like Target Regions, Provisioning state, and Replication state.
7. Select a **specific version** to view all its details.

:::image type="content" source="media/vmapps/vm-applications-details-portal.png" alt-text="Screenshot showing VM Application properties & all versions in the Azure portal.":::

:::image type="content" source="media/vmapps/vm-applications-version-details-portal.png" alt-text="Screenshot showing VM Application version properties in the Azure portal.":::

#### [Rest](#tab/rest1)
View the properties of a published VM Application or a specific version using the REST API:

**Get VM Application details:**
```rest
GET
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}?api-version=2024-03-03
```
Sample response:
```json
{
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}",
    "name": "{galleryApplicationName}",
    "type": "Microsoft.Compute/galleries/applications",
    "location": "eastus",
    "properties": {
        "description": "Sample VM Application",
        "provisioningState": "Succeeded",
        "supportedOSTypes": ["Windows", "Linux"],
        "endOfLifeDate": null,
        "privacyStatementUri": "https://contoso.com/privacy",
        "releaseNoteUri": "https://contoso.com/release-notes"
    }
}
```

**Get VM Application version details:**
```rest
GET
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}/versions/{galleryApplicationVersionName}?api-version=2024-03-03
```
Sample response:
```json
{
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}/versions/{galleryApplicationVersionName}",
    "name": "{galleryApplicationVersionName}",
    "type": "Microsoft.Compute/galleries/applications/versions",
    "location": "eastus",
    "properties": {
        "provisioningState": "Succeeded",
        "publishingProfile": {
            "source": {
                "mediaLink": "https://storageaccount.blob.core.windows.net/vmapps/app.zip"
            },
            "replicaCount": 1,
            "targetRegions": [
                {
                    "name": "eastus",
                    "regionalReplicaCount": 1
                }
            ]
        },
        "storageAccountType": "Standard_LRS"
    }
}
```

The responses include properties such as name, location, provisioning state, description, and other metadata about the application or version.

#### [CLI](#tab/cli1)

Set variables:
```azurecli-interactive
rgName="myResourceGroup"
galleryName="myGallery"
appName="myVmApp"
versionName="1.0.0"
```

List all VM Applications in the gallery:
```azurecli-interactive
az sig gallery-application list \
    --resource-group $rgName \
    --gallery-name $galleryName \
    -o table
```

Show a VM Application’s properties:
```azurecli-interactive
az sig gallery-application show \
    --resource-group $rgName \
    --gallery-name $galleryName \
    --application-name $appName
```

List all versions for a VM Application:
```azurecli-interactive
az sig gallery-application version list \
    --resource-group $rgName \
    --gallery-name $galleryName \
    --application-name $appName \
    --query "[].{version:name, provisioningState:properties.provisioningState}" -o table
```

Show a specific VM Application version’s properties:
```azurecli-interactive
az sig gallery-application version show \
    --resource-group $rgName \
    --gallery-name $galleryName \
    --application-name $appName \
    --version-name $versionName
```

#### [PowerShell](#tab/powershell1)
Use Azure PowerShell to view VM Application and version details in an Azure Compute Gallery.

Set variables:
```azurepowershell-interactive
$rgName      = "myResourceGroup"
$galleryName = "myGallery"
$appName     = "myVmApp"
$versionName = "1.0.0"
```

List all VM Applications in the gallery:
```azurepowershell-interactive
Get-AzGalleryApplication `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName |
    Select-Object Name, Location, ProvisioningState
```

Show a VM Application’s properties:
```azurepowershell-interactive
Get-AzGalleryApplication `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -Name $appName |
    ConvertTo-Json -Depth 5
```

List all versions for a VM Application:
```azurepowershell-interactive
Get-AzGalleryApplicationVersion `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -GalleryApplicationName $appName |
    Select-Object Name, @{n="ProvisioningState";e={$_.ProvisioningState}}
```

Show a specific VM Application version’s properties:
```azurepowershell-interactive
Get-AzGalleryApplicationVersion `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -GalleryApplicationName $appName `
    -Name $versionName |
    ConvertTo-Json -Depth 6
```
---


## View published VM applications using Azure Resource Graph

[Azure resource graph query](/azure/governance/resource-graph/first-query-portal) can be used to view all published VM applications and their properties across all compute galleries. It provides a programmatic way to view application inventory, and their properties at high scale. Use this method for integrating with dashboards and custom reports.


**View list of all compute galleries**
```kusto-interactive
resources
| where type == "microsoft.compute/galleries"
| where subscriptionId == "85236c53-92ad-4e66-97a4-8253a5246b99"
| extend provisioningState = properties["provisioningState"]
| extend permissions = properties["sharingProfile"]["permissions"]
| extend communityGalleryInfo = properties["sharingProfile"]["communityGalleryInfo"]
| project subscriptionId, resourceGroup, location, name, provisioningState, permissions, communityGalleryInfo
```

**View list of all published VM Applications across all galleries**
```kusto-interactive
resources
| where subscriptionId == "85236c53-92ad-4e66-97a4-8253a5246b99"
| where type == "microsoft.compute/galleries/applications"
| extend OSType = properties["supportedOSType"]
| extend description = properties["description"]
| extend endOfLifeDate = properties["endOfLifeDate"]
| parse id with  "/subscriptions/" SubId "/resourceGroups/" rgName "/providers/Microsoft.Compute/galleries/" gallaryName "/applications/" appName  
| project subscriptionId, resourceGroup, location, gallaryName, name, OSType, description
```

**View list of all published VM Application Versions across all applications and galleries**
```kusto-interactive
resources
| where type == "microsoft.compute/galleries/applications/versions"
| project subscriptionId, resourceGroup, id, location, properties
| parse id with  "/subscriptions/" SubId "/resourceGroups/" rgName "/providers/Microsoft.Compute/galleries/" gallaryName "/applications/" appName "/versions/" versionNumber 
| extend provisioningState = properties["provisioningState"]
| extend publishingProfile = properties["publishingProfile"]
| extend storageAccountType = publishingProfile["storageAccountType"]
| extend scriptBehaviorAfterReboot = publishingProfile["settings"]["scriptBehaviorAfterReboot"]
| extend packageFileName = publishingProfile["settings"]["packageFileName"]
| extend configFileName = publishingProfile["settings"]["configFileName"]
| extend mediaLink = publishingProfile["source"]["mediaLink"]
| extend defaultConfigurationLink = publishingProfile["source"]["defaultConfigurationLink"]
| extend excludeFromLatest = publishingProfile["excludeFromLatest"]
| extend targetRegions = publishingProfile["targetRegions"]
| extend replicaCount = publishingProfile["replicaCount"]
| extend publishedDate = publishingProfile["publishedDate"]
| extend installScript = publishingProfile["manageActions"]["install"]
| extend removeScript = publishingProfile["manageActions"]["remove"]
| extend safetyProfile = properties["safetyProfile"]
| extend allowDeletionOfReplicatedLocations = safetyProfile["allowDeletionOfReplicatedLocations"]
| project-away safetyProfile, publishingProfile, SubId, rgName, id, properties
```

---

## Update published VM Application
> [!NOTE]
> Not all properties of a published VM Application or VM Application version can be updated. For a complete list of updatable properties, refer to the [VM Application resource and VM Application version resource schema](vm-applications.md#publish-application-as-azure-vm-application).

#### [REST](#tab/rest2)

**Update VM Application resource:**
```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}?api-version=2024-03-03

Body
{
    "properties": {
        "description": "Updated description",
        "endOfLifeDate": "2026-12-31T00:00:00Z",
        "eula": "Updated EULA text",
        "privacyStatementUri": "https://contoso.com/privacy",
        "releaseNoteUri": "https://contoso.com/release-notes"
    }
}
```

**Update VM Application version resource:**
```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}/versions/{galleryApplicationVersionName}?api-version=2024-03-03

Body
{
    "properties": {
        "publishingProfile": {
            "targetRegions": [
                {
                    "name": "eastus",
                    "regionalReplicaCount": 2,
                    "storageAccountType": "Standard_LRS"
                },
                {
                    "name": "westus",
                    "regionalReplicaCount": 1,
                    "storageAccountType": "Standard_LRS"
                }
            ],
            "excludeFromLatest": false
        }
    }
}
```

#### [CLI](#tab/cli2)

**Update VM Application resource:**
```azurecli-interactive
az sig gallery-application update \
    --resource-group $rgName \
    --gallery-name $galleryName \
    --application-name $appName \
    --description "Updated description for the VM Application"
```

**Update VM Application version resource:**
```azurecli-interactive
az sig gallery-application version update \
    --resource-group $rgName \
    --gallery-name $galleryName \
    --application-name $appName \
    --version-name $versionName \
    --target-regions "eastus=2" "westus=1"
```

#### [PowerShell](#tab/powershell2)

Set variables:
```azurepowershell-interactive
$rgName      = "myResourceGroup"
$galleryName = "myGallery"
$appName     = "myVmApp"
$versionName = "1.0.0"
```

**Update VM Application resource:**
```azurepowershell-interactive
$app = Get-AzGalleryApplication `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -Name $appName

Update-AzGalleryApplication `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -Name $appName `
    -Description "Updated description for the VM Application"
```

**Update VM Application version resource:**
```azurepowershell-interactive
$targetRegions = @(
    @{
        Name = "eastus"
        RegionalReplicaCount = 2
        StorageAccountType = "Standard_LRS"
    },
    @{
        Name = "westus"
        RegionalReplicaCount = 1
        StorageAccountType = "Standard_LRS"
    }
)

Update-AzGalleryApplicationVersion `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -GalleryApplicationName $appName `
    -Name $versionName `
    -PublishingProfileTargetRegion $targetRegions `
    -PublishingProfileExcludeFromLatest:$false
```
---

## Delete published VM Application from Azure Compute Gallery
To delete the VM Application resource, you need to first delete all its versions. Deleting the application version causes deletion of the application version resource from Azure Compute Gallery and all its replicas. The application blob in Storage Account used to create the application version is unaffected. 

> [!WARNING]
> - Deleting the application version causes subsequent PUT operations on VMs using that version to fail. To prevent this failure, use `latest` keyword as the version number in the `applicationProfile` instead of hard coding the version number  .  
>
> - Deleting the VM application that is deployed on any VM or Virtual Machine Scale Sets causes subsequent PUT operations on these resources to fail (for example, update, scale, or reimage). Before deleting, ensure all VMs/Virtual Machine Scale Sets instances stop using the application by removing it from their applicationProfile.
>
>- To prevent accidental deletion,  set `safetyProfile/allowDeletionOfReplicatedLocations` to `false` while publishing the version and apply an Azure Resource Manager lock (CanNotDelete or ReadOnly) on the VM application resource.

#### [Portal](#tab/portal3)
1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for Azure Compute Gallery and open the target gallery.
3. Select the VM application you want to remove.
4. Select one or more versions, which you want to delete.
5. To delete the VM application, first delete all the versions. Then click delete (on top of the blade).
6. Monitor Notifications for completion. If deletion is blocked, remove any locks and ensure no VM or Virtual Machine Scale Sets references the application.

:::image type="content" source="media/vmapps/vm-applications-delete-from-gallery-portal.png" alt-text="Screenshot showing deletion of a VM application and its versions in the Azure portal.":::

#### [REST](#tab/rest3)
Delete the VM Application version:
```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}/versions/{galleryApplicationVersionName}?api-version=2024-03-03
```

Delete the VM Application after all its versions are deleted:
```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}?api-version=2024-03-03
```

#### [CLI](#tab/cli3)
Delete the VM Application version:
```azurecli-interactive
az sig gallery-application version delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name --version-name $version-name
```

Delete the VM Application after all its versions are deleted:
```azurecli-interactive
az sig gallery-application delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name
```

#### [PowerShell](#tab/powershell3)
Delete the VM Application version: 
```azurepowershell-interactive
Remove-AzGalleryApplicationVersion -ResourceGroupName $rgNmae -GalleryName $galleryName -GalleryApplicationName $galleryApplicationName -Name $name
```

Delete the VM Application after all its versions are deleted:
```azurepowershell-interactive
Remove-AzGalleryApplication -ResourceGroupName $rgNmae -GalleryName $galleryName -Name $name
```
---



## Manage deployed VM Application on Azure VM and Virtual Machine Scale Sets
This section explains how to view deployed application details and monitor deployed applications across infrastructure. It also talks about how to update and delete **deployed VM Applications** on Azure VMs and Virtual Machine Scale Sets. 

## View deployed VM Applications & their state
Azure uses VMAppExtension to deploy, monitor, and manage VM Applications on the VM. Therefore, provisioning state of the deployed VM Application is described in the status of the VMAppExtension. 

#### [Portal](#tab/portal4)
To show the VM application status, go to the **Extensions + applications** tab under settings and check the status of the VMAppExtension:

:::image type="content" source="media/vmapps/select-app-status.png" alt-text="Screenshot showing VM application status.":::

To show the VM application status for a Virtual Machine Scale Sets, go to the Azure portal Virtual Machine Scale Sets page. In the Instances section, select one of the instances. Then go to **Extensions + Applications** tab under settings and check the status of the **VMAppExtension**:

:::image type="content" source="media/vmapps/select-apps-status-vmss-portal.png" alt-text="Screenshot showing Virtual Machine Scale Sets application status.":::

#### [REST](#tab/rest4)

If the VM application isn't installed on the VM, the value is empty. 

To get the result of VM instance view:

```rest
GET
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VMName}/instanceView?api-version=2024-03-03
```

The result looks like this:

```rest
{
    ...
    "extensions"  [
    ...
        {
            "name":  "VMAppExtension",
            "type":  "Microsoft.CPlat.Core.VMApplicationManagerLinux",
            "typeHandlerVersion":  "1.0.9",
            "statuses":  [
                            {
                                "code":  "ProvisioningState/succeeded",
                                "level":  "Info",
                                "displayStatus":  "Provisioning succeeded",
                                "message":  "Enable succeeded: {\n \"CurrentState\": [\n  {\n   \"applicationName\": \"doNothingLinux\",\n   \"version\": \"1.0.0\",\n   \"result\": \"Install SUCCESS\"\n  },\n  {
        \n   \"applicationName\": \"badapplinux\",\n   \"version\": \"1.0.0\",\n   \"result\": \"Install FAILED Error executing command \u0027exit 1\u0027: command terminated with exit status=1\"\n  }\n ],\n \"ActionsPerformed\": []\n}
        "
                            }
                        ]
        }
    ...
    ]
}
```

The VM App status is in the status message of the result of the VM App extension in the instance view.

To get the status for the application on the Virtual Machine Scale Sets:

```rest
GET
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/ virtualMachineScaleSets/{VMSSName}/virtualMachines/{instanceId}/instanceView?api-version=2019-03-01
```
The output is similar to the VM example earlier.

#### [CLI](#tab/cli4)

To verify application deployment status on a VM, use ['az vm get-instance-view'](/cli/azure/vm/#az-vm-get-instance-view):

```azurecli-interactive
az vm get-instance-view -g myResourceGroup -n myVM --query "instanceView.extensions[?name == 'VMAppExtension']"
```
To verify application deployment status on Virtual Machine Scale Sets, use ['az vmss get-instance-view'](/cli/azure/vmss/#az-vmss-get-instance-view):

```azurecli-interactive
az vmss get-instance-view --ids (az vmss list-instances -g myResourceGroup -n myVmss --query "[*].id" -o tsv) --query "[*].extensions[?name == 'VMAppExtension']"
```

#### [PowerShell](#tab/powershell4)

To view the provisioning state of deployed VM Application on Azure VMs, use [Get-AzVM](/powershell/module/az.compute/get-azvm) command. 

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$result = Get-AzVM -ResourceGroupName $rgName -VMName $vmName -Status
$result.Extensions | Where-Object {$_.Name -eq "VMAppExtension"} | ConvertTo-Json
```

To view the provisioning state of deployed VM Application on Virtual Machine Scale Sets, use [Get-AzVMSS](/powershell/module/az.compute/get-azvmss) command. 

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmssName = "myVMss"
$result = Get-AzVmssVM -ResourceGroupName $rgName -VMScaleSetName $vmssName -InstanceView
$resultSummary  = New-Object System.Collections.ArrayList
$result | ForEach-Object {
    $res = @{ instanceId = $_.InstanceId; vmappStatus = $_.InstanceView.Extensions | Where-Object {$_.Name -eq "VMAppExtension"}}
    $resultSummary.Add($res) | Out-Null
}
$resultSummary | ConvertTo-Json -Depth 5
```
---


## View logs of application installation using Run command
When Azure VM Applications downloads and installs the application on Azure VM or Virtual Machine Scale Sets, it pipes all stdout results to `stdout` file within the application repository. Customers can [enable verbose logging for the application installation and write custom logs](vm-applications-how-to.md#3-create-the-install-script) using the `installScript`. Customers can then manually check the `stdout` and `stderr` file or use Runcommand to get the file content. 

Use the following PowerShell script in your managed run command. Update the `appName` and `appVersion` variables for your application

#### [Windows](#tab/windows5)

```azure-powershell-interactive
$appName = "vm-application-name"        # VM Application definition name
$appVersion = "1.0.0"                   # VM Application version name

$VMAppManagerVersion = "1.0.16"         # Version of the VMApplicationManagerWindows extension on the VM
$StdoutFilePath = "C:\Packages\Plugins\Microsoft.CPlat.Core.VMApplicationManagerWindows\$VMAppManagerVersion\Downloads\$appName\$appVersion\stdout"
$StderrFilePath = "C:\Packages\Plugins\Microsoft.CPlat.Core.VMApplicationManagerWindows\$VMAppManagerVersion\Downloads\$appName\$appVersion\stderr"

Write-Host "`n=== Contents of stdout ==="
Get-Content -Path $StdoutFilePath

Write-Host "`n=== Contents of stderr ==="
Get-Content -Path $StderrFilePath
```

#### [Linux](#tab/Linux5)

```bash
appName="vm-application-name"       # VM Application definition name
appVersion="1.0.0"                  # VM Application version name

stdoutFilePath="/var/lib/waagent/Microsoft.CPlat.Core.VMApplicationManagerLinux/$appName/$appVersion/stdout"
stderrFilePath="/var/lib/waagent/Microsoft.CPlat.Core.VMApplicationManagerLinux/$appName/$appVersion/stderr"

printf "=== Contents of stdout ===\n%s \n" "$(cat $stdoutFilePath)"
printf '=== Contents of stderr ===\n%s \n' "$(cat $stderrFilePath)"
```
---

Execute the run command using the scripts and get the application install logs. 

#### [Portal](#tab/portal5)

1. Open the [Azure portal](https://portal.azure.com) and navigate to your VM.
2. Under **Operations**, select **Run command**.
3. Choose **RunPowerShellScript** from the list.
4. Enter your PowerShell script in the editor.
5. Select **Run** to execute the script.
6. View the output in the **Output** section. If the script fails, check the **Error** section for details.

For more information, see [Run PowerShell scripts in your Windows VM by using Run Command](/azure/virtual-machines/windows/run-command).

#### [CLI](#tab/cli5)

**Option 1: Using Action run command:**
```azurecli-interactive
rgName="myResourceGroup"
vmName="myVM"
az vm run-command invoke \
  --resource-group myResourceGroup \
  --name myVm \
  --command-id RunShellScript \
  --scripts @./path/to/script.sh
```
Learn more about [Action run command](/azure/virtual-machines/linux/run-command) to run a local script on the VM.

**Option 2: Using managed run command:**
```azurecli-interactive
rgName="myResourceGroup"
vmName="myVM"
subId="mySubscriptionId"
vmLocation="myVMLocation"

az vm run-command create \
    --name view-vmapp-install-logs \
    --vm-name $vmName \
    --resource-group $rgName \
    --subscription $subId \
    --location $vmLocation \
    --script @.\path\to\script.sh \
    --timeout-in-seconds 600

az vm run-command show \
    --name view-vmapp-install-logs \
    --vm-name $vmName \
    --resource-group $rgName \
    --subscription $subId \
    --instance-view
```
Learn more about [managed run command](/azure/virtual-machines/linux/run-command-managed) to run a script on the VM. 

#### [PowerShell](#tab/powershell5)

**Option 1: Using Action run command**
```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$scriptPath = ".\path\to\script.ps1"

$result = Invoke-AzVMRunCommand `
    -ResourceGroupName $rgName `
    -Name $vmName `
    -CommandId "RunPowerShellScript" `
    -ScriptPath $scriptPath

$result.Value[0].Message
```
Learn more about [Action run command](/powershell/module/az.compute/invoke-azvmruncommand) to run a local PowerShell script on the VM.

**Option 2: Using Managed run**
```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$runCommandName = "GetInstallLogsRunCommand"
$vmlocation = "myVMLocation"

Set-AzVMRunCommand `
    -ResourceGroupName $rgName `
    -VMName $vmName `
    -RunCommandName $runCommandName `
    -Location $vmLocation `
    -SourceScript $(Get-Content -Path "./GetInstallLogs.ps1" -Raw) 

$result = Get-AzVMRunCommand `
    -ResourceGroupName $rgName `
    -VMName $vmName `
    -RunCommandName $runCommandName `
    -Expand InstanceView

$result.InstanceView
```
Learn more about [managed run command](/azure/virtual-machines/windows/run-command-managed) to run a local PowerShell script on the VM. 


---

## View all deployed VM applications using Azure Resource Graph

[Azure resource graph query](/azure/governance/resource-graph/first-query-portal) can be used to view all deployed VM applications and their properties across all VMs and Virtual Machine Scale Sets. It provides a programmatic way to view application inventory, state, and deployed versions at high scale. Use this method for integrating with dashboards and custom reports.

```kusto-interactive
resources
| where type == "microsoft.compute/virtualmachines" or type == "microsoft.compute/virtualmachinescalesets"
| where properties has "applicationProfile"
| extend resourceType = iff(type == "microsoft.compute/virtualmachines", "VM", "VMSS")
| extend applications = iff(resourceType == "VM", parse_json(properties["applicationProfile"]["galleryApplications"]), parse_json(properties["virtualMachineProfile"]["applicationProfile"]["galleryApplications"]))
| mv-expand applications
| extend enableAutomaticUpgrade = applications["enableAutomaticUpgrade"]
| extend packageReferenceId = applications["packageReferenceId"]
| extend treatFailureAsDeploymentFailure = applications["treatFailureAsDeploymentFailure"]
| parse packageReferenceId with "/subscriptions/" publisherSubcriptionId "/resourceGroups/" publisherResourceGroup "/providers/Microsoft.Compute/galleries/" galleryName "/applications/" appName "/versions/" version
| project tenantId, subscriptionId, resourceGroup, resourceName = name, type, location, appName, version, enableAutomaticUpgrade, treatFailureAsDeploymentFailure, galleryName, publisherSubcriptionId, publisherResourceGroup, properties
```

## Audit required VM Application using Azure Policy
Azure Policy helps enforce governance by auditing whether required VM applications are deployed across your VMs and Virtual Machine Scale Sets. You can create and assign custom policies to check compliance and ensure that specific applications are present on your infrastructure.

For step-by-step instructions on how to audit VM application deployment using Azure Policy, see [Audit required VM applications using Azure Policy](vm-applications-inject-with-policy.md#set-up-compliance-monitor-to-govern-required-vm-applications).

## Update the deployed VM Application
To update a deployed VM application, modify the `applicationProfile` to reference a newer version or change deployment settings such as `treatFailureAsDeploymentFailure` or `order`.

#### [REST](#tab/rest6)

**Update the VM application version or settings on a single VM:**

```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-03-03

Body
{
    "properties": {
        "applicationProfile": {
            "galleryApplications": [
                {
                    "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}/versions/{newVersion}",
                    "treatFailureAsDeploymentFailure": true,
                    "enableAutomaticUpgrade": true
                }
            ]
        }
    }
}
```

**Update VM application on a Virtual Machine Scale Sets (model):**

```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}?api-version=2024-03-03

Body
{
    "properties": {
        "virtualMachineProfile": {
            "applicationProfile": {
                "galleryApplications": [
                    {
                        "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}/versions/{newVersion | latest}",
                        "treatFailureAsDeploymentFailure": true,
                        "order": 2
                    }
                ]
            }
        }
    }
}
```

Apply the change to existing Virtual Machine Scale Sets instances (required when upgradePolicy.mode is Manual):

```rest
POST
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}/updateInstances?api-version=2024-03-03

Body
{
    "instanceIds": ["*"]
}
```

#### [CLI](#tab/cli6)

**Update VM application on a single VM:**

```azurecli-interactive
az vm application set \
    --resource-group myResourceGroup \
    --name myVM \
    --app-version-ids /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/latest \
    --treat-deployment-as-failure true
```

**Update VM application on a Virtual Machine Scale Sets (model):**

```azurecli-interactive
az vmss application set \
    --resource-group myResourceGroup \
    --name myVMss \
    --app-version-ids /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/latest \
    --treat-deployment-as-failure true
```

Apply the change to existing Virtual Machine Scale Sets instances (If upgradeMode set to manual):

```azurecli-interactive
az vmss update-instances -g myResourceGroup -n myVMss --instance-ids "*"
```

#### [PowerShell](#tab/powershell6)

**Update VM application on a single VM:**

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$galleryName = "myGallery"
$applicationName = "myApp"
$newVersion = "latest"

$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$appVersion = Get-AzGalleryApplicationVersion `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -GalleryApplicationName $applicationName `
    -Name $newVersion
$packageId = $appVersion.Id

$app = New-AzVmGalleryApplication -PackageReferenceId $packageId -TreatFailureAsDeploymentFailure
$vm.ApplicationProfile.GalleryApplications = @($app)
Update-AzVM -ResourceGroupName $rgName -VM $vm
```

**Update VM application on a Virtual Machine Scale Sets (model):**

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmssName = "myVMss"
$galleryName = "myGallery"
$applicationName = "myApp"
$newVersion = "latest"

$vmss = Get-AzVmss -ResourceGroupName $rgName -VMScaleSetName $vmssName
$appVersion = Get-AzGalleryApplicationVersion `
    -ResourceGroupName $rgName `
    -GalleryName $galleryName `
    -GalleryApplicationName $applicationName `
    -Name $newVersion
$packageId = $appVersion.Id

$app = New-AzVmssGalleryApplication -PackageReferenceId $packageId -TreatFailureAsDeploymentFailure
$vmss.VirtualMachineProfile.ApplicationProfile.GalleryApplications = @($app)
Update-AzVmss -ResourceGroupName $rgName -VMScaleSetName $vmssName -VirtualMachineScaleSet $vmss
```

Apply the change to existing VMSS instances (If upgradeMode set to Manual):

```azurepowershell-interactive
$instanceIds = (Get-AzVmssVM -ResourceGroupName $rgName -VMScaleSetName $vmssName).InstanceId
Update-AzVmssInstance -ResourceGroupName $rgName -VMScaleSetName $vmssName -InstanceId $instanceIds
```
---

> [!TIP]
> Use `latest` as the version identifier in `packageReferenceId` to automatically deploy the newest published version without manually updating deployments.

## Remove the VM Application from Azure VM or Virtual Machine Scale Sets

#### [Portal](#tab/portal7)
1. Open the Azure portal and go to the target virtual machine (VM) or Virtual Machine Scale Sets.
2. In Settings, select **Extensions + applications**, then select the **VM Applications** tab.
3. Click uninstall button on the VM Application and Save.
4. Track progress in Notifications or check Instance view for the VMAppExtension status.

:::image type="content" source="media/vmapps/vm-applications-delete-from-vm-portal.png" alt-text="Screenshot showing how to Uninstall VM application from a VM.":::

#### [REST](#tab/rest7)
To remove a VM application, update the `applicationProfile` by clearing or excluding the target application.

Remove from a single VM:
```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2024-03-03

Body
{
    "properties": {
        "applicationProfile": {
            "galleryApplications": []
        }
    }
}
```

Remove from a Virtual Machine Scale Sets (model):
```rest
PATCH
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}?api-version=2024-03-03

Body
{
    "properties": {
        "virtualMachineProfile": {
            "applicationProfile": {
                "galleryApplications": []
            }
        }
    }
}
```

Apply the change to existing Virtual Machine Scale Sets instances (required when upgradePolicy.mode is Manual):
```rest
POST
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}/updateInstances?api-version=2024-03-03

Body
{
    "instanceIds": ["0", "1"]
}
```

#### [CLI](#tab/cli7)
Remove from a single VM:
```azurecli-interactive
az vm update -g myResourceGroup -n myVM --set "properties.applicationProfile.galleryApplications=[]"
```

Remove from a Virtual Machine Scale Sets (model):
```azurecli-interactive
az vmss update -g myResourceGroup -n myVMss --set "virtualMachineProfile.applicationProfile.galleryApplications=[]"
```

Apply the change to existing VMSS instances (Manual upgrade policy):
```azurecli-interactive
az vmss update-instances -g myResourceGroup -n myVMss --instance-ids "*"
```

#### [PowerShell](#tab/powershell7)
Remove from a single VM:
```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"

$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$vm.ApplicationProfile.GalleryApplications = @()
Update-AzVM -ResourceGroupName $rgName -VM $vm
```

Remove from a Virtual Machine Scale Sets (model) and apply to instances:
```azurepowershell-interactive
$rgName   = "myResourceGroup"
$vmssName = "myVMss"

$vmss = Get-AzVmss -ResourceGroupName $rgName -VMScaleSetName $vmssName
$vmss.VirtualMachineProfile.ApplicationProfile.GalleryApplications = @()
Update-AzVmss -ResourceGroupName $rgName -VMScaleSetName $vmssName -VirtualMachineScaleSet $vmss

$instanceIds = (Get-AzVmssVM -ResourceGroupName $rgName -VMScaleSetName $vmssName).InstanceId
Update-AzVmssInstance -ResourceGroupName $rgName -VMScaleSetName $vmssName -InstanceId $instanceIds
```
---


## Next steps
- Learn more about [Azure VM Applications](vm-applications.md).
- Learn to [create Azure VM applications](vm-applications-how-to.md).