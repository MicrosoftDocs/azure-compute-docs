---
title: Create and deploy VM Application packages
description: Learn how to create and deploy Virtual Machine (VM) Applications using an Azure Compute Gallery.
author: gabstamsft
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/08/2023
ms.author: gabsta
ms.reviewer: jushiman
ms.custom: devx-track-azurepowershell, devx-track-azurecli
---

# Create and deploy VM Application

VM Application are a resource type in Azure Compute Gallery that simplifies management, sharing, and global distribution of applications for your virtual machines. [Learn more about VM Application](./vm-applications.md)


## Prerequisites

1. Create [Azure Compute Gallery for storing and sharing application resources](create-gallery.md).
1. Upload your application to a container in an [Azure storage account](/azure/storage/common/storage-account-create). Your application can be stored in a block or page blob. If you choose to use a page blob, you need to byte align the files before you upload them. Use the following sample to byte align your file.

### [PowerShell](#tab/powershell)
```azurepowershell-interactive
$inputFile = <the file you want to pad>

$fileInfo = Get-Item -Path $inputFile

$remainder = $fileInfo.Length % 512

if ($remainder -ne 0){

    $difference = 512 - $remainder

    $bytesToPad = [System.Byte[]]::CreateInstance([System.Byte], $difference)

    Add-Content -Path $inputFile -Value $bytesToPad -Encoding Byte
    }
```
### [CLI](#tab/cli)
```azurecli-interactive
inputFile="<the file you want to pad>"

# Get the file size
fileSize=$(stat -c %s "$inputFile")

# Calculate the remainder when divided by 512
remainder=$((fileSize % 512))

if [ "$remainder" -ne 0 ]; then
    # Calculate how many bytes to pad
    difference=$((512 - remainder))
    
    # Create padding (empty bytes)
    dd if=/dev/zero bs=1 count=$difference >> "$inputFile"
fi
```
----

Ensure the storage account has public level access or use an SAS URI with read privilege, as other restriction levels fail deployments. You can use [Storage Explorer](/azure/vs-azure-tools-storage-explorer-blobs) to quickly create a SAS URI if you don't already have one.

If you're using PowerShell, you need to be using version 3.11.0 of the Az.Storage module.

To learn more about the installation mechanism, see the [command interpreter.](vm-applications.md#command-interpreter)

## Create the VM Application

### [Portal](#tab/portal1)

1. Go to the [Azure portal](https://portal.azure.com), then search for and select **Azure Compute Gallery**.
1. Select the gallery you want to use from the list.
1. On the page for your gallery, select **Add** from the top of the page and then select **VM application definition** from the drop-down. The **Create a VM application definition** page opens.
1. In the **Basics** tab, enter a name for your application and choose whether the application is for VMs running Linux or Windows.
1. Select the **Publishing options** tab if you want to specify any of the following optional settings for your VM Application definition:
    - A description of the VM Application definition.
    - End of life date
    - Link to an End User License Agreement (EULA)
    - URI of a privacy statement
    - URI for release notes
1. When you're done, select **Review + create**.
1. When validation completes, select **Create** to have the definition deployed.
1. Once the deployment is complete, select **Go to resource**.
1. On the page for the application, select **Create a VM application version**. The **Create a VM Application Version** page opens.
1. Enter a version number like 1.0.0.
1. Select the region where your application packages are uploaded.
1. Under **Source application package**, select **Browse**. Select the storage account, then the container where your package is located. Select the package from the list and then select **Select** when you're done. Alternatively, you can paste the SAS URI in this field if preferred.
1. Provide the '**Install script**'. You can also provide the '**Uninstall script**' and the '**Update script**'. See the [Overview](vm-applications.md#command-interpreter) for information on how to create the scripts.
1. If you have a default configuration file uploaded to a storage account, you can select it in **Default configuration**.
1. Select **Exclude from latest** if you don't want this version to appear as the latest version when you create a VM.
1. For **End of life date**, choose a date in the future to track when this version should be retired. It isn't deleted or removed automatically, it's only for your own tracking.
1. To replicate this version to other regions, select the **Replication** tab, add more regions, and make changes to the number of replicas per region. The original region where your version was created must be in the list and can't be removed.
1. When you're done making changes, select **Review + create** at the bottom of the page.
1. When validation shows as passed, select **Create** to deploy your VM application version.


### [PowerShell](#tab/powershell1)

Create the VM Application definition using [`New-AzGalleryApplication`](/powershell/module/az.compute/new-azgalleryapplication). In this example, we're creating a Linux app named *myApp* in the *myGallery* Azure Compute Gallery and in the *myGallery* resource group. Replace the values for variables as needed.

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$description = "Backend Linux application for finance."
New-AzGalleryApplication `
  -ResourceGroupName $rgName `
  -GalleryName $galleryName `
  -Location "East US" `
  -Name $applicationName `
  -SupportedOSType Linux `
  -Description $description
```

Create a version of your VM Application using [`New-AzGalleryApplicationVersion`](/powershell/module/az.compute/new-azgalleryapplicationversion). Allowed characters for version are numbers and periods. Numbers must be within the range of a 32-bit integer. Format: *MajorVersion*.*MinorVersion*.*Patch*.

In this example, we're creating version number *1.0.0*. Replace the values of the variables as needed.

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$version = "1.0.0"
New-AzGalleryApplicationVersion `
   -ResourceGroupName $rgName `
   -GalleryName $galleryName `
   -GalleryApplicationName $applicationName `
   -Name $version `
   -PackageFileLink "https://<storage account name>.blob.core.windows.net/<container name>/<filename>" `
   -DefaultConfigFileLink "https://<storage account name>.blob.core.windows.net/<container name>/<filename>" `
   -Location "East US" `
   -Install "mv myApp .\myApp\myApp" `
   -Remove "rm .\myApp\myApp" `
```

### [CLI](#tab/cli1)

VM applications require [Azure CLI](/cli/azure/install-azure-cli) version 2.30.0 or later.

Create the VM application definition using ['az sig gallery-application create'](/cli/azure/sig/gallery-application#az_sig_gallery_application_create). In this example, we're creating a VM application definition named *myApp* for Linux-based VMs.


```azurecli-interactive
az sig gallery-application create \
    --application-name myApp \
    --gallery-name myGallery \
    --resource-group myResourceGroup \
    --os-type Linux \
    --location "East US"
```

Create a VM application version using ['az sig gallery-application version create'](/cli/azure/sig/gallery-application/version#az-sig-gallery-application-version-create). Allowed characters for version are numbers and periods. Numbers must be within the range of a 32-bit integer. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Replace the values of the parameters with your own.

```azurecli-interactive
az sig gallery-application version create \
   --version-name 1.0.0 \
   --application-name myApp \
   --gallery-name myGallery \
   --location "East US" \
   --resource-group myResourceGroup \
   --package-file-link "https://<storage account name>.blob.core.windows.net/<container name>/<filename>" \
   --install-command "mv myApp .\myApp\myApp" \
   --remove-command "rm .\myApp\myApp" \
   --update-command  "mv myApp .\myApp\myApp" \
   --default-configuration-file-link "https://<storage account name>.blob.core.windows.net/<container name>/<filename>"\
```

### [REST](#tab/rest1)

Create the VM Application definition using the ['create gallery application API'](/rest/api/compute/gallery-applications)


```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}?api-version=2024-03-03

{
    "location": "West US",
    "name": "myApp",
    "properties": {
        "supportedOSType": "Windows | Linux",
        "endOfLifeDate": "2020-01-01",
	"description": "Description of the App",
	"eula": "Link to End-User License Agreement (EULA)",
	"privacyStatementUri": "Link to privacy statement for the application",
	"releaseNoteUri": "Link to release notes for the application"
    }
}

```

| Field Name | Description | Limitations |
|--|--|--|
| name | A unique name for the VM Application within the gallery. | Max length of 117 characters. Allowed characters are uppercase or lowercase letters, digits, hyphen(-), period (.), underscore (_). Names not allowed to end with period(.). |
| supportedOSType | Define the supported OS Type. | "Windows" or "Linux" |
| endOfLifeDate | A future end of life date for the application. The date is for reference only and isn't enforced. | Valid future date |
| description | Optional. Description of the Application. |
| eula | Optional. Reference to End-User License Agreement (EULA). |
| privacyStatementUri | Optional. Reference to privacy statement for the application. |
| releaseNoteUri | Optional. Reference to release notes for the application. |


Create a VM application version using the ['create gallery application version API'](/rest/api/compute/gallery-applications).

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}/versions/{versionName}?api-version=2024-03-03

{
  "location": "$location",
  "properties": {
    "publishingProfile": {
      "source": {
        "mediaLink": "$mediaLink",
        "defaultConfigurationLink": "$configLink"
      },
      "manageActions": {
        "install": "echo installed",
        "remove": "echo removed",
        "update": "echo update"
      },
      "targetRegions": [
        {
          "name": "West US",
          "regionalReplicaCount": 1
        },
	{
	  "name": "East US"
	}
      ]
      "endofLifeDate": "datetime",
      "replicaCount": 1,
      "excludeFromLatest": false,
      "storageAccountType": "PremiumV2_LRS | Premium_LRS | Standard_LRS | Standard_ZRS"
      "safetyProfile": {
	"allowDeletionOfReplicatedLocations": false
      }
      "settings": {
	"scriptBehaviorAfterReboot": "None | Rerun",
	"configFileName": "$appConfigFileName",
	"packageFileName": "$appPackageFileName"
      }
   }
}

```

| Field Name | Description | Limitations |
|--|--|--|
| location | Source location for the VM Application version. | Valid Azure region |
| mediaLink | The url containing the application version package. | Valid and existing storage url |
| defaultConfigurationLink | Optional. The url containing the default configuration, which may be overridden at deployment time. | Valid and existing storage url |
| Install | The command to install the application. | Valid command for the given OS |
| Remove | The command to remove the application. | Valid command for the given OS |
| Update | Optional. The command to update the application. If not specified and an update is required, the old version is removed and the new one installed. | Valid command for the given OS |
| targetRegions/name | The name of a region to which to replicate. | Validate Azure region |
| targetRegions/regionalReplicaCount | Optional. The number of replicas in the region to create. Defaults to 1. | Integer between 1 and 3 inclusive |
| replicaCount | Optional. Defines the number of replicas across each region. Takes effect if regionalReplicaCount isn't defined. | Integer between 1 and 3 inclusive |
| endOfLifeDate | A future end of life date for the application version. This property is for customer reference only and isn't enforced. | Valid future date |
| storageAccountType | Optional. Type of storage account to use in each region for storing application package. Defaults to Standard_LRS. | This property is nonupdatable. |
| allowDeletionOfReplicatedLocations | Optional. Indicates whether or not removing this Gallery Image Version from replicated regions is allowed. | |
| settings/scriptBehaviorAfterReboot | Optional. The action to be taken for installing, updating, or removing gallery application after the VM is rebooted. | | 
| settings/configFileName | Optional. The name to assign the downloaded package file on the VM. If not specified, the package file is named the same as the Gallery Application name. | This is limited to 4,096 characters. |
| settings/packageFileName | Optional. The name to assign the downloaded config file on the VM. If not specified, the config file is named as the Gallery Application name appended with '_config'. | This is limited to 4,096 characters. |

----

## Deploy the VM Apps

### [Portal](#tab/portal2)

Now you can create a VM and deploy the VM application to it using the portal. Just create the VM as usual, and under the **Advanced** tab, choose **Select a VM application to install**.

:::image type="content" source="media/vmapps/advanced-tab.png" alt-text="Screenshot of the Advanced tab where you can choose to install a VM application.":::

Select the VM application from the list and then select **Save** at the bottom of the page.

:::image type="content" source="media/vmapps/select-app.png" alt-text="Screenshot showing selecting a VM application to install on the VM.":::

If you have more than one VM application to install, you can set the install order for each VM application back on the **Advanced tab**.

You can also deploy the VM application to currently running VMs. Select the **Extensions + applications** option under **Settings** in the left menu when viewing the VM details in the portal.

Choose **VM applications** and then select **Add application** to add your VM application.

:::image type="content" source="media/vmapps/select-extension-app.png" alt-text="Screenshot showing selecting a VM application to install on a currently running VM.":::

Select the VM application from the list and then select **Save** at the bottom of the page.

:::image type="content" source="media/vmapps/select-app.png" alt-text="Screenshot showing selecting a VM application to install on the VM.":::

### [CLI](#tab/cli2)
Set a VM application to an existing VM using ['az vm application set'](/cli/azure/vm/application#az-vm-application-set) and replace the values of the parameters with your own.

```azurecli-interactive
az vm application set \
	--resource-group myResourceGroup \
	--name myVM \
  	--app-version-ids /subscriptions/{subID}/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 \
  	--treat-deployment-as-failure true
```
For setting multiple applications on a VM:

```azurecli-interactive
az vm application set \
	--resource-group myResourceGroup \
	--name myVM \
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp2/versions/1.0.1 \
	--treat-deployment-as-failure true true
```
To add an application to a Virtual Machine Scale Set, use ['az vmss application set'](/cli/azure/vmss/application#az-vmss-application-set):

```azurecli-interactive
az vmss application set \
	--resource-group myResourceGroup \
	--name myVmss \
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 \
	--treat-deployment-as-failure true
```
To add multiple applications to a Virtual Machine Scale Set:
```azurecli-interactive
az vmss application set \
	--resource-group myResourceGroup \
	--name myVmss
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp2/versions/1.0.0 \
	--treat-deployment-as-failure true
```

### [PowerShell](#tab/powershell2)

To add the application to an existing VM, get the application version and use that to get the VM application version ID. Use the ID to add the application to the VM configuration.

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$version = "1.0.0"
$vmName = "myVM"
$vm = Get-AzVM -ResourceGroupName $rgname -Name $vmName
$appVersion = Get-AzGalleryApplicationVersion `
   -GalleryApplicationName $applicationName `
   -GalleryName $galleryName `
   -Name $version `
   -ResourceGroupName $rgName
$packageId = $appVersion.Id
$app = New-AzVmGalleryApplication -PackageReferenceId $packageId
Add-AzVmGalleryApplication -VM $vm -GalleryApplication $app -TreatFailureAsDeploymentFailure true
Update-AzVM -ResourceGroupName $rgName -VM $vm
```
To add the application to a Virtual Machine Scale Set:
```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName $rgname -Name $vmssName
$appVersion = Get-AzGalleryApplicationVersion `
   -GalleryApplicationName $applicationName `
   -GalleryName $galleryName `
   -Name $version `
   -ResourceGroupName $rgName
$packageId = $appVersion.Id
$app = New-AzVmssGalleryApplication -PackageReferenceId $packageId
Add-AzVmssGalleryApplication -VirtualMachineScaleSetVM $vmss.VirtualMachineProfile -GalleryApplication $app
Update-AzVmss -ResourceGroupName $rgName -VirtualMachineScaleSet $vmss -VMScaleSetName $vmssName
```
### [REST](#tab/rest2)


To add a VM application version to a VM, perform a PUT on the VM.

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VMName}?api-version=2024-03-03

{
  "properties": {
    "applicationProfile": {
      "galleryApplications": [
        {
          "order": 1,
          "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/<resource group>/providers/Microsoft.Compute/galleries/{gallery name}/applications/{application name}/versions/{version | latest}",
          "configurationReference": "{path to configuration storage blob}",
          "treatFailureAsDeploymentFailure": false
        }
      ]
    }
  },
  "name": "{vm name}",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resource group}/providers/Microsoft.Compute/virtualMachines/{vm name}",
  "location": "{vm location}"
}
```


To apply the VM application to a uniform scale set:

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMSSName}?api-version=2024-03-03

{
  "properties": {
    "virtualMachineProfile": {
      "applicationProfile": {
        "galleryApplications": [
          {
            "order": 1,
            "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/<resource group>/providers/Microsoft.Compute/galleries/{gallery name}/applications/{application name}/versions/{version | latest}",
            "configurationReference": "{path to configuration storage blob}",
            "treatFailureAsDeploymentFailure": false
          }
        ]
      }
    }
  },
  "name": "{vm name}",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resource group}/providers/Microsoft.Compute/virtualMachines/{vm name}",
  "location": "{vm location}"
}
```


| Field Name | Description | Limitations |
|--|--|--|
| order | Optional. The order in which the applications should be deployed. | Validate integer |
| packageReferenceId | A reference to the gallery application version. Use "latest" keyword for version to automatically install the latest available version. | Valid application version reference |
| configurationReference | Optional. The full url of a storage blob containing the configuration for this deployment. This overrides any value provided for defaultConfiguration earlier. | Valid storage blob reference |
| treatFailureAsDeploymentFailure | Optional. When enabled, app deployment failure causes VM provisioning status to report failed status. | True or False

The order field may be used to specify dependencies between applications. The rules for order are as follows:

| Case | Install Meaning | Failure Meaning |
|--|--|--|
| No order specified | Unordered applications are installed after ordered applications. There's no guarantee of installation order among the unordered applications. | Installation failures of other applications, be it ordered or unordered doesn’t affect the installation of unordered applications. |
| Duplicate order values | Application is installed in any order compared to other applications with the same order. All applications of the same order are installed after the ones with lower orders and before the ones with higher orders. | If a previous application with a lower order failed to install, no applications with this order install. If any application with this order fails to install, no applications with a higher order install. |
| Increasing orders | Application will be installed after the ones with lower orders and before the ones with higher orders. | If a previous application with a lower order failed to install, this application doesn't install. If this application fails to install, no application with a higher order installs. |

The response includes the full VM model. The following are the
relevant parts.

```rest
{
  "name": "{vm name}",
  "id": "{vm id}",
  "type": "Microsoft.Compute/virtualMachines",
  "location": "{vm location}",
  "properties": {
    "applicationProfile": {
      "galleryApplications": ""
    },
    "provisioningState": "Updating"
  },
  "resources": [
    {
      "name": "VMAppExtension",
      "id": "{extension id}",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "centraluseuap",
      "properties": "@{autoUpgradeMinorVersion=True; forceUpdateTag=7c4223fc-f4ea-4179-ada8-c8a85a1399f5; provisioningState=Creating; publisher=Microsoft.CPlat.Core; type=VMApplicationManagerLinux; typeHandlerVersion=1.0; settings=}"
    }
  ]
}

```
----


## Monitor the deployed VM Applications
### [Portal](#tab/portal3)
To show the VM application status, go to the Extensions + applications tab/settings and check the status of the VMAppExtension:

:::image type="content" source="media/vmapps/select-app-status.png" alt-text="Screenshot showing VM application status.":::

To show the VM application status for scale set, go to the Azure portal Virtual Machine Scale Sets page, then the Instances section, select one of the scales sets listed, then go to **VMAppExtension**:

:::image type="content" source="media/vmapps/select-apps-status-vmss-portal.png" alt-text="Screenshot showing virtual machine scale sets application status.":::

### [CLI](#tab/cli3)

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

### [PowerShell](#tab/powershell3)

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

### [REST](#tab/rest3)

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

---

## Delete the VM Application
To delete the VM Application resource, you need to first delete all its versions. Deleting the application version causes deletion of the application version resource from Azure Compute Gallery and all its replicas. The application blob in Storage Account used to create the application version is unaffected. After deleting the application version, if any VM is using that version, then reimage operation on those VMs will fail. Use 'latest' keyword as the version number in the 'applicationProfile' instead of hard coding the version number to address this failure.  
However if the application is deleted, then VM fails during reimage operation since there are no versions available for Azure to install. The VM profile needs to be updated to not use the VM Application. 

### [PowerShell](#tab/powershell4)
Delete the VM Application version: 
```azurepowershell-interactive
Remove-AzGalleryApplicationVersion -ResourceGroupName $rgNmae -GalleryName $galleryName -GalleryApplicationName $galleryApplicationName -Name $name
```

Delete the VM Application after all its versions are deleted:
```azurepowershell-interactive
Remove-AzGalleryApplication -ResourceGroupName $rgNmae -GalleryName $galleryName -Name $name
```

### [CLI](#tab/cli4)
Delete the VM Application version:
```azurecli-interactive
az sig gallery-application version delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name --version-name $version-name
```

Delete the VM Application after all its versions are deleted:
```azurecli-interactive
az sig gallery-application delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name
```

### [REST](#tab/rest4)
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

## Next steps
Learn more about [Azure VM Applications](vm-applications.md).
