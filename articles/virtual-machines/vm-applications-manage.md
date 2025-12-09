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

This article talks about how to monitor, update, and delete the published VM Application and the deployed VM application resource on Azure Virtual Machine (VM) or Virtual Machine Scale Sets.

## View the published VM Applications
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

## Monitor the deployed VM Applications

#### [Portal](#tab/portal2)
To show the VM application status, go to the **Extensions + applications** tab/settings and check the status of the VMAppExtension:

:::image type="content" source="media/vmapps/select-app-status.png" alt-text="Screenshot showing VM application status.":::

To show the VM application status for a scale set, go to the Azure portal Virtual Machine Scale Sets page. In the Instances section, select one of the scales sets listed, then go to **VMAppExtension**:

:::image type="content" source="media/vmapps/select-apps-status-vmss-portal.png" alt-text="Screenshot showing virtual machine scale sets application status.":::

#### [REST](#tab/rest2)

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

To get the status for the application on the Virtual Machine Scale Set:

```rest
GET
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/ virtualMachineScaleSets/{VMSSName}/virtualMachines/{instanceId}/instanceView?api-version=2019-03-01
```
The output is similar to the VM example earlier.

#### [CLI](#tab/cli2)

To verify application deployment status on a VM, use ['az vm get-instance-view'](/cli/azure/vm/#az-vm-get-instance-view):

```azurecli-interactive
az vm get-instance-view -g myResourceGroup -n myVM --query "instanceView.extensions[?name == 'VMAppExtension']"
```
To verify application deployment status on Virtual Machine Scale Set, use ['az vmss get-instance-view'](/cli/azure/vmss/#az-vmss-get-instance-view):

```azurecli-interactive
az vmss get-instance-view --ids (az vmss list-instances -g myResourceGroup -n myVmss --query "[*].id" -o tsv) --query "[*].extensions[?name == 'VMAppExtension']"
```
> [!NOTE]
> The previous Virtual Machine Scale Sets deployment status command doesn't list the instance ID with the result. To show the instance ID with the status of the extension in each instance, some more scripting is required. Refer to the following CLI example that contains PowerShell syntax:

```azurepowershell-interactive
$ids = az vmss list-instances -g myResourceGroup -n myVmss --query "[*].{id: id, instanceId: instanceId}" | ConvertFrom-Json
$ids | Foreach-Object {
    $iid = $_.instanceId
    Write-Output "instanceId: $iid" 
    az vmss get-instance-view --ids $_.id --query "extensions[?name == 'VMAppExtension']" 
}
```

#### [PowerShell](#tab/powershell2)

Verify the application succeeded:

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$result = Get-AzVM -ResourceGroupName $rgName -VMName $vmName -Status
$result.Extensions | Where-Object {$_.Name -eq "VMAppExtension"} | ConvertTo-Json
```
To verify on your Virtual Machine Scale Set:
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

## Remove the VM Application from Azure VM or VMSS

#### [Portal](#tab/portal3)
1. Open the Azure portal and go to the target virtual machine (VM) or Virtual Machine Scale Set.
2. In Settings, select **Extensions + applications**, then select the **VM Applications** tab.
3. Click uninstall button on the VM Application and Save.
4. Track progress in Notifications or check Instance view for the VMAppExtension status.

:::image type="content" source="media/vmapps/vm-applications-delete-from-vm-portal.png" alt-text="Screenshot showing how to Uninstall VM application from a VM.":::

#### [REST](#tab/rest3)
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

Remove from a VM scale set (model):
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

Apply the change to existing VMSS instances (required when upgradePolicy.mode is Manual):
```rest
POST
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmssName}/updateInstances?api-version=2024-03-03

Body
{
    "instanceIds": ["0", "1"]
}
```

#### [CLI](#tab/cli3)
Remove from a single VM:
```azurecli-interactive
az vm update -g myResourceGroup -n myVM --set "properties.applicationProfile.galleryApplications=[]"
```

Remove from a VM scale set (model):
```azurecli-interactive
az vmss update -g myResourceGroup -n myVMss --set "virtualMachineProfile.applicationProfile.galleryApplications=[]"
```

Apply the change to existing VMSS instances (Manual upgrade policy):
```azurecli-interactive
az vmss update-instances -g myResourceGroup -n myVMss --instance-ids "*"
```

#### [PowerShell](#tab/powershell3)
Remove from a single VM:
```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"

$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
$vm.ApplicationProfile.GalleryApplications = @()
Update-AzVM -ResourceGroupName $rgName -VM $vm
```

Remove from a VM scale set (model) and apply to instances:
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

## Delete the VM Application from Azure Compute Gallery
To delete the VM Application resource, you need to first delete all its versions. Deleting the application version causes deletion of the application version resource from Azure Compute Gallery and all its replicas. The application blob in Storage Account used to create the application version is unaffected. 

> [!WARNING]
> - Deleting the application version causes subsequent PUT operations on VMs using that version to fail. Use `latest` keyword as the version number in the `applicationProfile` instead of hard coding the version number to address this failure.  
>
> - Deleting the VM application that is referenced by any VM or VMSS causes subsequent PUT operations on those resources to fail (for example, update, scale, or reimage). Before deleting, ensure all VMs/VMSS instances stop using the application by removing it from their applicationProfile. 
>
>- To prevent accidental deletion,  set `safetyProfile/allowDeletionOfReplicatedLocations` to `false` while publishing the version and apply an Azure Resource Manager lock (CanNotDelete or ReadOnly) on the VM application resource.

#### [Portal](#tab/portal4)
1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for Azure Compute Gallery and open the target gallery.
3. Select the VM application you want to remove.
4. Select one or more versions, which you want to delete.
5. To delete the VM application, first delete all the versions. Then click delete (on top of the blade).
6. Monitor Notifications for completion. If deletion is blocked, remove any locks and ensure no VM or scale set references the application.

:::image type="content" source="media/vmapps/vm-applications-delete-from-gallery-portal.png" alt-text="Screenshot showing deletion of a VM application and its versions in the Azure portal.":::

#### [REST](#tab/rest4)
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

#### [CLI](#tab/cli4)
Delete the VM Application version:
```azurecli-interactive
az sig gallery-application version delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name --version-name $version-name
```

Delete the VM Application after all its versions are deleted:
```azurecli-interactive
az sig gallery-application delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name
```

#### [PowerShell](#tab/powershell4)
Delete the VM Application version: 
```azurepowershell-interactive
Remove-AzGalleryApplicationVersion -ResourceGroupName $rgNmae -GalleryName $galleryName -GalleryApplicationName $galleryApplicationName -Name $name
```

Delete the VM Application after all its versions are deleted:
```azurepowershell-interactive
Remove-AzGalleryApplication -ResourceGroupName $rgNmae -GalleryName $galleryName -Name $name
```
---

## Next steps
Learn more about [Azure VM Applications](vm-applications.md).

Learn to [create Azure VM applications](vm-applications-how-to.md).