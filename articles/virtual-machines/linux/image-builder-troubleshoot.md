---
title: Troubleshoot Azure VM Image Builder
description: This article helps you troubleshoot common problems and errors you might encounter when you're using Azure VM Image Builder.
author: kof-f
ms.author: kofiforson
ms.reviewer: jushiman
ms.date: 11/27/2023
ms.topic: troubleshooting
ms.service: azure-virtual-machines
ms.subservice: image-builder
ms.custom: devx-track-azurecli, linux-related-content
---

# Troubleshoot Azure VM Image Builder

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

Use this article to troubleshoot and resolve common problems that you might encounter when you're using Azure VM Image Builder.

VM Image Builder failures can happen in two areas:

- During image template submission
- During image building

## Prerequisites

These prerequisites are for creating a build:

- The VM Image Builder service communicates to the build virtual machine (VM) by using Windows Remote Management (WinRM) or Secure Shell (SSH). *Don't* disable these settings as part of the build.
- VM Image Builder creates resources in the staging resource group as part of the builds. The exact list of resources depends on the [network configuration](./image-builder-json.md#vnetconfig-optional) that the image template specifies. Be sure to verify that Azure Policy doesn't prevent VM Image Builder from creating or using necessary resources.
  - Create an `IT_` resource group.
  - Create a storage account without a firewall.
  - Deploy [Azure Container Instances](../../container-instances/container-instances-overview.md).
  - Deploy [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) resources (and subnets within them).
  - Deploy [Azure Private Endpoint](/azure/private-link/private-endpoint-overview) resources.
  - Deploy [Azure Files](/azure/storage/files/storage-files-introduction).
- Ensure that Azure Policy does not install unintended features on the build VM or other staging resources, such as Azure extensions or tag modifications.
- Ensure that VM Image Builder has the correct permissions to read/write images and to connect to the storage account. For more information, review the permissions documentation for the [Azure CLI](./image-builder-permissions-cli.md) or [Azure PowerShell](./image-builder-permissions-powershell.md).
- VM Image Builder fails the build if the scripts or inline commands fail with errors (nonzero exit codes). Ensure that you tested the custom scripts and verified that they run without error (exit code `0`) or require user input. For more information, see [Create an Azure Virtual Desktop image by using VM Image Builder and PowerShell](../windows/image-builder-virtual-desktop.md#tips-for-building-windows-images).
- Ensure that your subscription has a sufficient [quota](../../container-instances/container-instances-resource-and-quota-limits.md) for Azure Container Instances.
  
  Each image build might deploy up to one temporary Azure Container Instances resource (of four standard cores) in the staging resource group. These resources are required for [Isolated Image Builds](../security-isolated-image-builds-image-builder.md).

> [!NOTE]
> CIS Hardened Images (Linux or Windows) on Azure Marketplace, managed by the Center for Internet Security (CIS), can cause build failures with the VM Image Builder service due to their configurations. For instance:
>
> - CIS Hardened Images for Windows might disrupt WinRM connectivity, a prerequisite for VM Image Builder builds.
> - CIS Hardened Images for Linux can fail due to `chmod +x` permission issues.

## Troubleshoot image template submission errors

Image template submission errors are returned at submission only. No error log exists for image template submission errors. If there's an error during submission, you can return the error by checking the status of the template. Specifically, review `ProvisioningState` and `ProvisioningErrorMessage`/`provisioningError`.

Here's the command for the Azure CLI:

```azurecli-interactive
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

Here's the command for Azure PowerShell. To use Azure PowerShell, you need to install the [VM Image Builder PowerShell modules](../windows/image-builder-powershell.md#prerequisites).

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object ProvisioningState, ProvisioningErrorMessage
```

Here's the error output for version 2020-02-14 and earlier:

```output
{
  "code": "ValidationFailed",
  "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template."
}
```

Here's the error output for version 2021-10-01 and later:

```output
{
  "error": {
    "code": "ValidationFailed",
    "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template."
  }
}
```

> [!IMPORTANT]
> API version 2021-10-01 introduces a change to the error schema that will be part of every future API release. If you have any Azure VM Image Builder automations, be aware of the [new error output](#troubleshoot-image-template-submission-errors) when you switch to API version 2021-10-01 or later.
>
> We recommend, after you switch to the latest API version, that you don't revert to an earlier version. If you revert, you'll have to change your automation again to produce the earlier error schema. We don't expect the error schema to change again in future releases.

The following sections present problem resolution guidance for common image template submission errors.

### Update or upgrade of image templates is currently not supported

#### Error

```output
'Conflict'. Details: Update/Upgrade of image templates is currently not supported
```

#### Cause

The template already exists.

#### Solution

If you submit an image configuration template and the submission fails, a failed template artifact still exists. Delete the failed template.

### Assigned managed identity on an image template can't be used

#### Error

```output
The assigned managed identity cannot be used. Please remove the existing one and re-assign a new identity. For more troubleshooting steps go to https://aka.ms/azvmimagebuilderts.
```

#### Cause

There are cases where a created [managed identity](./image-builder-permissions-cli.md#create-a-user-assigned-managed-identity) assigned to the image template can't be used.

One possible cause is that the VM Image Builder template uses a customer-provided staging resource group, and the managed identity is deleted before the image template is deleted. (This is a [staging resource group](./image-builder-json.md#properties-stagingresourcegroup) scenario.)

#### Solution

Use the Azure CLI to reset the managed identity on the image template. Be sure to [update](/cli/azure/update-azure-cli) Azure CLI to the 2.45.0 version or later.

Confirm the managed identity from the target VM Image Builder template:

```azurecli-interactive
az image builder identity show -g <template resource group> -n <template name> 
```

Remove the managed identity from the target VM Image Builder template:

```azurecli-interactive
az image builder identity remove -g <template resource group> -n <template name> --user-assigned <identity resource id>
```

Assign a new identity to the target VM Image Builder template:

```azurecli-interactive
az image builder identity assign -g <template rg> -n <template name> --user-assigned <identity resource id>
```

For more information about configuring permissions, see [Configure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

### Assigned managed identity isn't authorized to access a resource

#### Error

```output
Not authorized to access the resource: <resource-not-able-to-access>. Please check the user assigned identity has the correct permissions. For more details, go to https://aka.ms/azvmimagebuilderts.
```

#### Cause

The created [managed identity](./image-builder-permissions-cli.md#create-a-user-assigned-managed-identity) assigned to the image template doesn't have all permissions to access the resource shared in the error message.

#### Solution

Confirm the managed identity from the target VM Image Builder template:

```azurecli-interactive
az image builder identity show -g <template resource group> -n <template name> 
```

Review the role assignments for the identity:

```azurecli-interactive
az role assignment list --assignee <identity_client_id_or_principal_id>
```

Assign the required role. If necessary, create your role with the required permissions.

For more information about configuring permissions, see [Configure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

### Resource operation finished with a terminal provisioning state of "Failed"

#### Error

```output
Microsoft.VirtualMachineImages/imageTemplates 'helloImageTemplateforSIG01' failed with message '{
  "status": "Failed",
  "error": {
    "code": "ResourceDeploymentFailure",
    "message": "The resource operation completed with terminal provisioning state 'Failed'.",
    "details": [
      {
        "code": "InternalOperationError",
        "message": "Internal error occurred."
```

#### Cause

In most cases, the error about resource deployment failure occurs because of missing permissions. A conflict with the staging resource group might also cause this error.

#### Solution

Depending on your scenario, VM Image Builder might need permissions to:

- The source image or Azure Compute Gallery (formerly Shared Image Gallery) resource group.
- The distribution image or Azure Compute Gallery resource.
- The storage account, container, or blob that the `File` customizer is accessing.

Also, ensure that the staging resource group name is uniquely specified for each image template.

For more information about configuring permissions, see [Configure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

### Error occurs with getting a managed image

#### Error

```output
Build (Managed Image) step failed: Error getting Managed Image '/subscriptions/.../providers/Microsoft.Compute/images/mymanagedmg1': Error getting managed image (...): compute.
ImagesClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error.
Status=403 Code="AuthorizationFailed" Message="The client '......' with object id '......' doesn't have authorization to perform action 'Microsoft.Compute/images/read' over scope
```

#### Cause

Permissions are missing.

#### Solution

Depending on your scenario, VM Image Builder might need permissions to:

- The source image or Azure Compute Gallery resource group.
- The distribution image or Azure Compute Gallery resource.
- The storage account, container, or blob that the `File` customizer is accessing.

For more information about configuring permissions, see [Configure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

### Build step failed for the image version

#### Error

```output
Build (Shared Image Version) step failed for Image Version '/subscriptions/.../providers/Microsoft.Compute/galleries/.../images/... /versions/0.23768.4001': Error getting Image Version '/subscriptions/.../resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001': Error getting image version '... :0.23768.4001': compute.GalleryImageVersionsClient#Get: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error.
Status=404 Code="ResourceNotFound" Message="The Resource 'Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001' under resource group '<rgName>' was not found."
```

#### Cause

VM Image Builder can't locate the source image.

#### Solution

Ensure that the source image is correct and exists in the location of VM Image Builder.

### Error occurs with downloading an external file to a local file

#### Error

```output
Downloading external file (<myFile>) to local file (xxxxx.0.customizer.fp) [attempt 1 of 10] failed: Error downloading '<myFile>' to 'xxxxx.0.customizer.fp'..
```

#### Cause

The file name or location is incorrect, or the location isn't reachable.

#### Solution

Ensure that the file is reachable. Verify that the name and location are correct.

Certain file repositories might use unsupported cipher suites and cause download errors with VM Image Builder. Store files and scripts in an Azure storage account to help ensure secure cipher suites and accessibility by VM Image Builder. For more information on how to store your files in Azure storage accounts, see [Storage account overview](/azure/storage/common/storage-account-overview).

### Authorization error occurs with creating a disk

The VM Image Builder build fails with an authorization error that looks like the following example.

#### Error

```output
Attempting to deploy created Image template in Azure fails with an 'The client '6df325020-fe22-4e39-bd69-10873965ac04' with object id '6df325020-fe22-4e39-bd69-10873965ac04' does not have authorization to perform action 'Microsoft.Compute/disks/write' over scope '/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/disks/proxyVmDiskWin_<timestamp>' or the scope is invalid. If access was recently granted, please refresh your credentials.'
```

#### Cause

This error happens when you try to specify a preexisting resource group and virtual network to the VM Image Builder service with a Windows source image.

#### Solution

You need to assign the Contributor role to the resource group for the service principal that corresponds to VM Image Builder first-party app. Use the following Azure CLI commands or Azure portal instructions.

First, validate that the service principal is associated with the VM Image Builder first-party app by using the following Azure CLI command:

```azurecli-interactive
az ad sp show --id {servicePrincipalName, or objectId}
```

Then, to implement this solution by using the Azure CLI, use the following command:

```azurecli-interactive
az role assignment create -g {ResourceGroupName} --assignee {AibrpSpOid} --role Contributor
```

To implement this solution in the portal, follow the instructions in [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal):

- For [Step 1: Identify the needed scope](/azure/role-based-access-control/role-assignments-portal#step-1-identify-the-needed-scope), the needed scope is your resource group.

- For [Step 3: Select the appropriate role](/azure/role-based-access-control/role-assignments-portal#step-3-select-the-appropriate-role), the role is Contributor.

- For [Step 4: Select who needs access](/azure/role-based-access-control/role-assignments-portal#step-4-select-who-needs-access), select the **Azure Virtual Machine Image Builder** member.

- In [Step 7: Assign role](/azure/role-based-access-control/role-assignments-portal#step-7-assign-role), you assign the role.

## Troubleshoot build failures

For image build failures, get the error from `lastRunStatus`, and then review the details in the `customization.log` file.

```azurecli-interactive
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object LastRunStatus, LastRunStatusMessage
```

### Customization log

#### Access live logs during image build

To effectively monitor the progress of your image build, you can access the live logs that VM Image Builder generates in Azure Container Instances. These logs provide real-time insights into the build process. They help you identify any problems or confirm that the build is proceeding as expected.

To locate and view the live logs:

1. Start the image build process.

2. Go to the Azure portal and select **Resource Groups**. Filter by the subscription where you initiated the image build.

3. Find and select the staging resource group that's associated with the image build. This resource group contains the build resources for the VM Image Builder service. For more information on the staging resource group, see [Properties: stagingResourceGroup](./image-builder-json.md#properties-stagingresourcegroup).

4. Within this resource group, look for the resource named `vmimagebuilder-build-container-**********`. If it's not visible, wait a few minutes and refresh the page.

5. On the left pane, under **Settings**, select **Containers**.

6. Go to the **Logs** tab to view the live logs during the image build process.

If you don't see any logs, try refreshing the container after a few minutes.

#### Download the customization and/or validation log after image build

After the image build finishes, the customization and validation logs are stored in a container within the storage account in the staging resource group that the VM Image Builder service created. For more information on the staging resource group, see [Properties: stagingResourceGroup](./image-builder-json.md#properties-stagingresourcegroup).

To locate and download the `customization.log` or `validation.log` file:

1. In the Azure portal, go to the relevant storage account by filtering for storage accounts within the staging resource group that the VM Image Builder service created.

2. Under the storage account, go to **Data Storage**.

3. Select the **Container** option, and then choose the `packerlogs` container.

4. Within the `packerlogs` container, multiple folders appear if the image build ran multiple times. These folders are arranged from the oldest build to the most recent. Select the folder that corresponds to the build that you're interested in.

5. Inside the selected folder, select the `customization.log` and/or `validation.log` file, and then select **Download** to download its contents.

### Stages of the customization log

The log is verbose. It covers the image build, including any problems with the image distribution, such as Azure Compute Gallery replication. These problems are surfaced in the error message of the image template status.

The `customization.log` file includes the following stages:

1. *Deploy the build VM and dependencies by using Azure Resource Manager templates to the `IT_` staging resource group*. This stage includes multiple `POST` requests to the VM Image Builder resource provider:

    ```output
    Azure request method="POST" request="https://management.azure.com/subscriptions/<subID>/resourceGroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-cccc-cccc-ccccccc/providers/Microsoft.Storage/storageAccounts
    ..
    PACKER OUT ==> azure-arm: Deploying deployment template ...
    ..
    ```

1. *Get the status of the deployments*. This stage includes the status of each resource deployment:

    ```output
    PACKER ERR 2020/04/30 23:28:50 packer: 2020/04/30 23:28:50 Azure request method="GET" request="https://management.azure.com/subscriptions/<subID>/resourcegroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-4505-ae28-6661e43fac48/providers/Microsoft.Resources/deployments/pkrdp51lc0339jg/operationStatuses/08586133176207523519?[REDACTED]" body=""
    ```

1. *Connect to the build VM*.

    In Windows, VM Image Builder connects by using WinRM:

    ```output
    PACKER ERR 2020/04/30 23:30:50 packer: 2020/04/30 23:30:50 Waiting for WinRM, up to timeout: 10m0s
    ..
    PACKER OUT     azure-arm: WinRM connected.
    ```

    In Linux, VM Image Builder connects by using SSH:

    ```output
    PACKER OUT ==> azure-arm: Waiting for SSH to become available...
    PACKER ERR 2019/12/10 17:20:51 packer: 2020/04/10 17:20:51 [INFO] Waiting for SSH, up to timeout: 20m0s
    PACKER OUT ==> azure-arm: Connected to SSH!
    ```

1. *Run customizations*. When customizations run, you can identify them by reviewing the `customization.log` file. Search for `(telemetry)`:

    ```text
    (telemetry) Starting provisioner windows-update
    (telemetry) ending windows-update
    (telemetry) Starting provisioner powershell
    (telemetry) ending powershell
    (telemetry) Starting provisioner file
    (telemetry) ending file
    (telemetry) Starting provisioner windows-restart
    (telemetry) ending windows-restart

    (telemetry) Finalizing. - This means the build has finished
    ```

1. *Deprovision*. VM Image Builder adds a hidden customizer. This step is responsible for preparing the VM for deprovisioning. In Windows, it runs `Sysprep` (by using `c:\DeprovisioningScript.ps1`). In Linux, it runs `waagent-deprovision` (by using `/tmp/DeprovisioningScript.sh`).

    For example:

    ```text
    PACKER ERR 2020/03/04 23:05:04 [INFO] (telemetry) Starting provisioner powershell
    PACKER ERR 2020/03/04 23:05:04 packer: 2020/03/04 23:05:04 Found command: if( TEST-PATH c:\DeprovisioningScript.ps1 ){cat c:\DeprovisioningScript.ps1} else {echo "Deprovisioning script [c:\DeprovisioningScript.ps1] could not be found. Image build may fail or the VM created from the Image may not boot. Please make sure the deprovisioning script is not accidentally deleted by a Customizer in the Template."}
    ```

1. *Clean up*. After the build finishes, the VM Image Builder resources are deleted:

    ```text
    PACKER ERR ==> azure-arm: Deleting individual resources ...
    ...
    PACKER ERR 2020/02/04 02:04:23 packer: 2020/02/04 02:04:23 Azure request method="DELETE" request="https://management.azure.com/subscriptions/<subId>/resourceGroups/IT_aibDevOpsImg_t_vvvvvvv_yyyyyy-de5f-4f7c-92f2-xxxxxxxx/providers/Microsoft.Network/networkInterfaces/pkrnijamvpo08eo?[REDACTED]" body=""
    ...
    PACKER ERR ==> azure-arm: The resource group was not created by Packer, not deleting ...
    ```

## Troubleshoot script or inline customization

The following tips can help you troubleshoot script or inline customization:

- Test the code before you supply it to VM Image Builder.
- Ensure that Azure Policy and Azure Firewall allow connectivity to remote resources.
- Send output comments to the console by using `Write-Host` or `echo`. Doing so lets you search the `customization.log` file.

## Troubleshoot common build errors

### Template deployment failed because of a policy violation

#### Error

```text
{
  "statusCode": "BadRequest",
  "serviceRequestId": null,
  "statusMessage": "{\"error\":{\"code\":\"InvalidTemplateDeployment\",\"message\":\"The template deployment failed because of policy violation. Please see details for more information.\",\"details\":[{\"code\":\"RequestDisallowedByPolicy\",\"target\":\"<target_name>\",\"message\":\"Resource '<resource_name>' was disallowed by policy. Policy identifiers: '[{\\\"policyAssignment\\\":{\\\"name\\\":\\\"[Initiative] KeyVault (Microsoft.KeyVault)\\\",\\\"id\\\":\\\"/providers/Microsoft.Management/managementGroups/<managementGroup_name>/providers/Microsoft.Authorization/policyAssignments/Microsoft.KeyVault\\\"},\\\"policyDefinition\\\":{\\\"name\\\":\\\"Azure Key Vault should disable public network access\\\",\\\"id\\\":\\\"/providers/Microsoft.Management/managementGroups/<managementGroup_name>/providers/Microsoft.Authorization/policyDefinitions/KeyVault.disablePublicNetworkAccess_deny_deny\\\"},\\\"policySetDefinition\\\":{\\\"name\\\":\\\"[Initiative] KeyVault (Microsoft.KeyVault)\\\",\\\"id\\\":\\\"/providers/Microsoft.Management/managementGroups/<managementGroup_name>/providers/Microsoft.Authorization/policySetDefinitions/Microsoft.KeyVault\\\"}}]'.\",\"additionalInfo\":[{\"type\":\"PolicyViolation\"}]}]}}",
  "eventCategory": "Administrative",
  "entity": "/subscriptions/<subscription_ID>/<resourcegroups>/<resourcegroupname>/providers/Microsoft.Resources/deployments/<deployment_name>",
  "message": "Microsoft.Resources/deployments/validate/action",
  "hierarchy": "<subscription_ID>/<resourcegroupname>/<policy_name>/<managementGroup_name>/<deployment_ID>"
}
```

#### Cause

The preceding policy violation error is a result of using an Azure key vault with public access disabled. At this time, VM Image Builder doesn't support this configuration.

#### Solution

You must create the key vault with public access enabled.

### Packer build command fails

#### Error

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-04-30T23:24:06.756985789Z",
   "endTime": "2020-04-30T23:39:14.268729811Z",
   "runState": "Failed",
   "message": "Failed while waiting for packerizer: Microservice has failed: Failed while processing request: Error when executing packerizer: Packer build command has failed: exit status 1. During the image build, a failure has occurred, please review the build log to identify which build/customization step failed. For more troubleshooting steps go to https://aka.ms/azvmimagebuilderts. Image Build log location: https://xxxxxxxxxx.blob.core.windows.net/packerlogs/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/customization.log. OperationId: xxxxxx-5a8c-4379-xxxx-8d85493bc791. Use this operationId to search packer logs."
```

#### Cause

The failure of a packer build command is a customization failure.

#### Solution

Review the log to locate customizer failures. Search for `(telemetry)`.

For example:

```text
(telemetry) Starting provisioner windows-update
(telemetry) ending windows-update
(telemetry) Starting provisioner powershell
(telemetry) ending powershell
(telemetry) Starting provisioner file
(telemetry) ending file
(telemetry) Starting provisioner windows-restart
(telemetry) ending windows-restart

(telemetry) Finalizing. - This means the build has finished
```

### Deployment fails due to timeout

#### Error

```text
Deployment failed. Correlation ID: xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx. Failed in building/customizing image: Failed while waiting for packerizer: Timeout waiting for microservice to complete: 'context deadline exceeded'
```

#### Cause

The build exceeded the timeout. This error appears in `lastRunStatus`.

#### Solution

1. Review the `customization.log` file. Identify the last customizer to run. Search for `(telemetry)`, starting from the bottom of the log.

1. Check script customizations. The customizations might not be suppressing user interaction for commands, such as `quiet` options. For example, `apt-get install -y` results in the script execution waiting for user interaction.

1. If you're using the `File` customizer to download artifacts greater than 20 MB, see the workarounds section.

1. Review errors and dependencies in the script that might cause the script to wait.

1. If you expect that the customizations need more time, increase the value of [`buildTimeoutInMinutes`](image-builder-json.md). The default is 4 hours.

1. If you have resource-intensive actions, such as downloading gigabytes of files, consider the size of the underlying build VM.

   The service uses a Standard_D1_v2 VM. The VM has one vCPU and 3.5 GB of memory. If you're downloading 50 GB, you'll likely exhaust the VM resources and cause communication failures between VM Image Builder and the build VM. Retry the build with a larger-memory VM by setting [`VM_size`](image-builder-json.md#vmprofile).

### File download time is long

#### Error

```text
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT
myBigFile.zip 826 B / 826000 B  1.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT
myBigFile.zip 1652 B / 826000 B  2.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT
..
hours later...
..
myBigFile.zip 826000 B / 826000 B  100.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT
```

#### Cause

The `File` customizer is downloading a large file.

#### Solution

The `File` customizer is suitable only for small (less than 20 MB) file downloads. For larger file downloads, use a script or inline command. For example, in Linux, you can use `wget` or `curl`. In Windows, you can use `Invoke-WebRequest`.

### Windows restart continually fails with error code 1190

#### Error

```output
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] (telemetry) Starting provisioner windows-restart
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] starting remote command: shutdown /r /f /t 10
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] command 'shutdown /r /f /t 10' exited with code: 0
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER OUT ==> azure-arm: A system shutdown has already been scheduled.(1190)
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] RPC endpoint: Communicator ended with: 0
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] 0 bytes written for 'stdout'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] 0 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] RPC client: Communicator ended with: 0
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] RPC endpoint: Communicator ended with: 0
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] 0 bytes written for 'stdout'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] 0 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] RPC client: Communicator ended with: 0
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: Check if machine is rebooting...
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] starting remote command: shutdown /r /f /t 60 /c "packer restart test"
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] command 'shutdown /r /f /t 60 /c "packer restart test"' exited with code: 1190
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-plugin-azure plugin: 2023/06/13 08:28:58 [INFO] RPC endpoint: Communicator ended with: 1190
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] 52 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] 0 bytes written for 'stdout'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] RPC client: Communicator ended with: 1190
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 [INFO] RPC endpoint: Communicator ended with: 1190
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] 52 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] 0 bytes written for 'stdout'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: [INFO] RPC client: Communicator ended with: 1190
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:28:58 packer-provisioner-windows-restart plugin: Reboot already in progress, waiting...
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:29:08 packer-provisioner-windows-restart plugin: Check if machine is rebooting...
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:29:09 [INFO] 0 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:29:09 packer-provisioner-windows-restart plugin: [INFO] 0 bytes written for 'stderr'
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:29:09 packer-provisioner-windows-restart plugin: Waiting for machine to reboot with timeout: 15m0s
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:29:09 packer-provisioner-windows-restart plugin: Waiting for machine to become available...
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER OUT ==> Some builds didn't complete successfully and had errors:
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:46:26 machine readable: azure-arm,error []string{"Timeout waiting for machine to restart."}
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER OUT --> azure-arm: Timeout waiting for machine to restart.
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR ==> Builds finished but no artifacts were created.
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER OUT
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER ERR 2023/06/13 08:46:26 [INFO] (telemetry) Finalizing.
[864c0337-b300-48ab-8e8e-7894bc695b7c] PACKER OUT ==> Builds finished but no artifacts were created.
```

#### Cause

The Windows Update step is declared prematurely in images based on Windows Server 2016.

#### Solution

Increase `restartTimeout` from 15 minutes to 30 minutes.

### Error occurs with waiting for Azure Compute Gallery

#### Error

```text
Deployment failed. Correlation ID: XXXXXX-XXXX-XXXXXX-XXXX-XXXXXX. Failed in distributing 1 images out of total 1: {[Error 0] [Distribute 0] Error publishing MDI to Azure Compute Gallery:/subscriptions/<subId>/resourceGroups/xxxxxx/providers/Microsoft.Compute/galleries/xxxxx/images/xxxxxx, Location:eastus. Error: Error returned from SIG client while publishing MDI to Azure Compute Gallery for dstImageLocation: eastus, dstSubscription: <subId>, dstResourceGroupName: XXXXXX, dstGalleryName: XXXXXX, dstGalleryImageName: XXXXXX. Error: Error waiting on Azure Compute Gallery future for resource group: XXXXXX, gallery name: XXXXXX, gallery image name: XXXXXX.Error: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded}
```

#### Cause

VM Image Builder timed out while waiting for the image to be added and replicated to Azure Compute Gallery.

If the image is being injected into the gallery, you can assume that the image build was successful. However, the overall process failed because VM Image Builder was waiting for Azure Compute Gallery to complete the replication.

Even though the build failed, the replication continues. You can get the properties of the image version by checking the distribution `runOutput`:

```azurecli-interactive
$runOutputName=<distributionRunOutput>
az resource show \
    --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/$imageTemplateName/runOutputs/$runOutputName"  \
    --api-version=2020-02-14
```

#### Solution

Increase the value of `buildTimeoutInMinutes`.

### Information events show low Windows resources

#### Error

```text
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 1% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 51% cpu; 35% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 36% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 90% cpu; 32% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for the Windows Modules Installer to exit...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'PowerShell -ExecutionPolicy Bypass -OutputFormat Text -File C:/Windows/Temp/packer-windows-update-elevated.ps1' exited with code: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Restarting the machine...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Waiting for machine to become available...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] starting remote command: shutdown.exe -f -r -t 0 -c "packer restart"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'shutdown.exe -f -r -t 0 -c "packer restart"' exited with code: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: A system shutdown is in progress.(1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] starting remote command: shutdown.exe -f -r -t 60 -c "packer restart test"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] command 'shutdown.exe -f -r -t 60 -c "packer restart test"' exited with code: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 Retryable error: Machine not yet available (exit status 1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT Build 'azure-arm' errored: unexpected EOF
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT
```

#### Cause

The cause is resource exhaustion. This problem commonly happens when Windows Update runs with the default build VM size, D1_V2.

#### Solution

Increase the build VM size.

### Build finished but no artifacts were created

#### Warning

```text
[<log_id>] PACKER 2023/09/14 19:01:18 ui: Build 'azure-arm' finished after 3 minutes 13 seconds.
[<log_id>] PACKER 2023/09/14 19:01:18 ui:
[<log_id>] PACKER ==> Wait completed after 3 minutes 13 seconds
[<log_id>] PACKER 2023/09/14 19:01:18 ui:
[<log_id>] PACKER ==> Builds finished but no artifacts were created.
[<log_id>] PACKER 2023/09/14 19:01:18 [INFO] (telemetry) Finalizing.
[<log_id>] PACKER 2023/09/14 19:01:19 waiting for all plugin processes to complete...
[<log_id>] PACKER 2023/09/14 19:01:19 /aib/packerInput/packer-plugin-azure: plugin process exited
[<log_id>] PACKER 2023/09/14 19:01:19 /aib/packerInput/packer: plugin process exited
[<log_id>] PACKER 2023/09/14 19:01:19 /aib/packerInput/packer: plugin process exited
[<log_id>] PACKER 2023/09/14 19:01:19 /aib/packerInput/packer: plugin process exited
[<log_id>] PACKER Done exporting Packer logs to Azure Storage.
```

#### Solution

You can safely ignore the preceding warning.

### Image creation was skipped

#### Warning

```text
[<log_id>] PACKER 2023/09/14 19:00:18 ui: ==> azure-arm:  -> Snapshot ID : '/subscriptions/<subscription_id>/resourceGroups/<resourcegroup_name>/providers/Microsoft.Compute/snapshots/<snapshot_name>'
[<log_id>] PACKER 2023/09/14 19:00:18 ui: ==> azure-arm: Skipping image creation...
[<log_id>] PACKER 2023/09/14 19:00:18 ui: ==> azure-arm:
[<log_id>] PACKER ==> azure-arm: Deleting individual resources ...
[<log_id>] PACKER 2023/09/14 19:00:18 packer-plugin-azure plugin: 202
```

#### Solution

You can safely ignore the preceding warning.

### Resource is not found

#### Error

```output
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-05-01T00:13:52.599326198Z",
   "endTime": "2020-05-01T00:15:13.62366898Z",
   "runState": "Failed",
   "message": "network.InterfacesClient#UpdateTags: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. Status=404 Code=\"ResourceNotFound\" Message=\"The Resource 'Microsoft.Network/networkInterfaces/aibpls7lz2e.nic.4609d697-be0a-4cb0-86af-49b6fe877fe1' under resource group 'IT_aibImageRG200_window2019VnetTemplate01_9988723b-af56-413a-9006-84130af0e9df' was not found.\""
  },
```

#### Cause

Permissions are missing.

#### Solution

Recheck that VM Image Builder has all the permissions that it requires.

For more information about configuring permissions, see [Configure VM Image Builder permissions by using the Azure CLI](image-builder-permissions-cli.md) or [Configure VM Image Builder permissions by using PowerShell](image-builder-permissions-powershell.md).

### There's a Sysprep timing problem

#### Error

```text
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service RdAgent) -and ((Get-Service RdAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprepping VM ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: & $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while($true) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Write-Output $imageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Start-Sleep -s 5
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprep complete ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (RdAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Sysprepping VM ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_COMPLETE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: Get-Service : Cannot find any service with service name 'WindowsAzureGuestAgent'.
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: At C:\DeprovisioningScript.ps1:6 char:9
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: + while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service Window ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: +         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + CategoryInfo          : ObjectNotFound: (WindowsAzureGuestAgent:String) [Get-Service], ServiceCommandException
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 Cancelling builder after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT Cancelling build after receiving terminated
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling provisioning due to context cancellation: context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling hook after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
```

#### Cause

The D1_V2 VM size might be the reason for the timing problem. If customizations are limited and are run in less than three seconds, VM Image Builder runs `Sysprep` commands to deprovision. When VM Image Builder deprovisions, the `Sysprep` command checks for `WindowsAzureGuestAgent`, which might not be fully installed and might cause the timing problem.

#### Solution

To avoid the timing problem, you can increase the VM size or add a 60-second PowerShell sleep customization.

### Azure Container Instances provider is unregistered

#### Error

```text
Azure Container Instances provider not registered for your subscription.
```

#### Cause

Your template subscription doesn't have the Azure Container Instances provider registered.

#### Solution

Register the Azure Container Instances provider for your template subscription, and add one of the following commands:

- Azure CLI: `az provider register -n Microsoft.ContainerInstance`
- Azure PowerShell: `Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance`

### Subscription exceeded the Azure Container Instances quota

#### Error

```text
Azure Container Instances quota exceeded
```

#### Cause

Your subscription doesn't have enough Azure Container Instances quota for VM Image Builder to successfully build an image.

#### Solution

You can take the following actions to make Azure Container Instances quota available for VM Image Builder:

- Look up other usage of Azure Container Instances in your subscription. Remove any unneeded instances to make quota available for VM Image Builder.
- VM Image Builder deploys Azure Container Instances only temporarily during a build. These instances are deleted after the build finishes.

  If too many concurrent image builds are running in your subscription, you can consider delaying some of the image builds. This delay reduces concurrent usage of Azure Container Instances in your subscription.

  If your image templates are set up for automatic image builds via triggers, VM Image Builder automatically retries such failed builds.
- If the current Azure Container Instances limits for your subscription are too low to support your image-building scenarios, you can [request an increase in your Azure Container Instances quota](../../container-instances/container-instances-resource-and-quota-limits.md#next-steps).

> [!NOTE]
> Azure Container Instances resources are required for [Isolated Image Builds](../security-isolated-image-builds-image-builder.md).

### Too many Azure Container Instances resources are deployed within a period of time

#### Error

"Too many Azure Container Instances deployed within a period of time."

#### Cause

Your subscription doesn't have enough Azure Container Instances quota for VM Image Builder to successfully build images concurrently.

#### Solution

You can try these solutions:

- Retry your failed builds in a less concurrent manner.
- If the current Azure Container Instances limits for your subscription are too low to support your image-building scenarios, you can request an increase in your [Azure Container Instances quota](../../container-instances/container-instances-resource-and-quota-limits.md#next-steps).

### Isolated Image Builds feature causes a failure

#### Error

VM Image Builder builds are failing due to Isolated Image Builds.

#### Cause

VM Image Builder builds can fail for reasons listed elsewhere in this article. However, there's a small chance that a build fails due to Isolated Image Builds, depending on your scenario, subscription quotas, or some unforeseen service error. For more information, see [Isolated Image Builds](../security-isolated-image-builds-image-builder.md).

#### Solution

If you determine that a build is failing due to Isolated Image Builds, ensure that:

- No [Azure policy](/azure/governance/policy/overview) is blocking the deployment of resources mentioned in the [Prerequisites](#prerequisites) section of this article (specifically, Azure Container Instances).
- Your subscription has sufficient quota for Azure Container Instances to support all your concurrent image builds. For more information, see the [Subscription exceeded the Azure Container Instances quota](#subscription-exceeded-the-azure-container-instances-quota) section of this article.

VM Image Builder is currently in the process of deploying Isolated Image Builds. Specific image templates are not tied to Isolated Image Builds, and the same image template might or might not use Isolated Image Builds during different builds.

To temporarily run your build without Isolated Image Builds, retry your build. Because image templates aren't tied to the Isolated Image Builds feature, retrying a build has a high probability of rerunning without Isolated Image Builds.

If none of these solutions mitigate failing image builds, you can contact Azure support to temporarily opt your subscription out of Isolated Image Builds. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).

> [!NOTE]
> If the Isolated Image Builds feature is eventually enabled in all regions and templates, the preceding mitigations will be only temporary. It's best to address the underlying cause of build failures.

### Build is canceled after context cancellation

#### Error

```text
PACKER ERR 2020/03/26 22:11:23 Cancelling builder after context cancellation context canceled
PACKER OUT Cancelling build after receiving terminated
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling hook after context cancellation context canceled
..
PACKER ERR 2020/03/26 22:11:23 packer-builder-azure-arm plugin: Cancelling provisioning due to context cancellation: context canceled
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [ERROR] Remote command exited without exit status or exit signal.
PACKER ERR 2020/03/26 22:11:25 packer-builder-azure-arm plugin: [INFO] RPC endpoint: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] 148974 bytes written for 'stdout'
PACKER ERR 2020/03/26 22:11:25 [INFO] 0 bytes written for 'stderr'
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC client: Communicator ended with: 2300218
PACKER ERR 2020/03/26 22:11:25 [INFO] RPC endpoint: Communicator ended with: 2300218
```

#### Cause

VM Image Builder uses port 22 (Linux) or 5986 (Windows) to connect to the build VM. This problem occurs when the service is disconnected from the build VM during an image build. The reasons for the disconnection can vary, but enabling or configuring a firewall in the script can block the previously mentioned ports.

#### Solution

Review your scripts for firewall changes or enablement, or changes to SSH or WinRM. Ensure that any changes allow for constant connectivity between the service and the build VM on the previously mentioned ports. For more information, see [VM Image Builder networking options](./image-builder-networking.md).

### JWT errors appear in the log early in the build

#### Error

Early in the build process, the build fails and the log indicates a JSON Web Token (JWT) error:

```text
PACKER OUT Error: Failed to prepare build: "azure-arm"
PACKER ERR
PACKER OUT
PACKER ERR * client_jwt will expire within 5 minutes, please use a JWT that is valid for at least 5 minutes
PACKER OUT 1 error(s) occurred:
```

#### Cause

The `buildTimeoutInMinutes` value in the template is set to from 1 to 5 minutes.

#### Solution

As described in [Create a VM Image Builder template](./image-builder-json.md), the timeout must be set to 0 minutes to use the default or set to more than 5 minutes to override the default. Change the timeout in your template to 0 minutes to use the default, or change it to a minimum of 6 minutes.

### Resource deletion errors appear

#### Error

Intermediate resources are cleaned up toward the end of the build, and the customization log might show several resource deletion errors:

```text
PACKER OUT ==> azure-arm: Error deleting resource. Will retry.
...
PACKER OUT ==> azure-arm: Error: network.PublicIPAddressesClient#Delete: Failure sending request: StatusCode=0 -- Original Error: Code="PublicIPAddressCannotBeDeleted" Message=...
...
PACKER ERR 2022/03/07 18:43:06 packer-plugin-azure plugin: 2022/03/07 18:43:06 Retryable error: network.SecurityGroupsClient#Delete: Failure sending request: StatusCode=0 -- Original Error: Code="InUseNetworkSecurityGroupCannotBeDeleted"...
```

#### Cause

These error log messages are mostly harmless, because resource deletions are retried several times. Ordinarily, they eventually succeed. You can verify this behavior by continuing to follow the deletion logs until you observe a success message. Alternatively, you can inspect the staging resource group to confirm whether the resource was deleted.

Making these observations is especially important in build failures. These error messages might lead you to conclude that they're the reason for the failures, even when the actual errors might be elsewhere.

#### Error

When images are stuck in template deletion, the customization log might show the following error:

```output
error deleting resource id /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Network/networkInterfaces/<networkInterfacName>: resources.Client#DeleteByID: Failure sending request: StatusCode=400 --
Original Error: Code="NicInUseWithPrivateEndpoint"
Message="Network interface /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Network/networkInterfaces/<networkInterfacName> cannot be deleted because it is currently in use with an private endpoint (/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Network/privateEndpoints/<pIname>)." Details=[]
```

#### Cause

The error occurs because the network interface is currently in use with a private endpoint.

#### Solution

To resolve the problem, delete the following resources one by one in this specific order:

1. Private endpoint connection. You can find this connection in the Azure Private Link service resource by going to the **Private endpoint connections** tab on the page for the Private Link service resource.
1. Private Link service.
1. Network interface and load balancer.
1. Resource group.
1. Image template.

For more assistance, you can [contact Azure support](/azure/azure-portal/supportability/how-to-create-azure-support-request) to resolve the stuck deletion error.

### Distribution target is not found in the update request

#### Error

```text
Validation failed: Distribute target with Runoutput name <runoutputname> not found in the update request. Deleting a distribution target is not allowed.
```

#### Cause

This error occurs when an existing distribution target isn't found in the patch request body.

#### Solution

The distribution array should contain all the distribution targets: new targets (if any), existing targets with no change, and updated targets. If you want to remove an existing distribution target, delete and re-create the image template. Deleting a distribution target is currently not supported through the Patch API.

### Required fields are missing

#### Error

```text
Validation failed: 'ImageTemplate.properties.distribute[<index>]': Missing field <fieldname>. Please review http://aka.ms/azvmimagebuildertmplref for details on fields required in the Image Builder Template.
```

#### Cause

This error occurs when a required field is missing from a distribution target.

#### Solution

When you're creating a request, be sure to provide every required field in a distribution target, even if there's no change.

## Troubleshoot Azure DevOps

An Azure DevOps task fails only if an error occurs during customization. When this error happens, the task reports the failure and leaves the staging resource group, with the logs, so that you can identify the problem.

To locate the log:

1. Find the template name. Go to **Pipeline** > **Failed build**, and then drill down into the Azure DevOps task in VM Image Builder. Note the `template name` value.

   ```text
   start reading task parameters...
   found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
   end reading parameters
   getting storage account details for aibstordot1556933914
   created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
   Source for image:  { type: 'SharedImageVersion',
     imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
   template name:  t_1556938436xxx
   ```

1. Go to the Azure portal, search for the template name in the resource group, and then search for the resource group by entering  **IT_**.
1. Select the storage account name, and then select **blobs** > **containers** > **logs**.

You might occasionally need to investigate successful builds and review their logs. As mentioned earlier, if the image build is successful, the staging resource group that contains the logs is deleted as part of the cleanup. To prevent an automatic cleanup, you can introduce `sleep` after the inline command, and then view the logs as the build is paused:

1. Update the inline command by adding `Write-Host / Echo "Sleep"`. This addition gives you time to search in the log.
1. Add a `sleep` value of at least 10 minutes by using a [`Start-Sleep`](/powershell/module/microsoft.powershell.utility/start-sleep) or `Sleep` Linux command.
1. Use this method to identify the log location, and then keep downloading or checking the log until it gets to `sleep`.

### Operation was canceled

#### Error

```text
2020-05-05T18:28:24.9280196Z ##[section]Starting: Azure VM Image Builder Task
2020-05-05T18:28:24.9609966Z ==============================================================================
2020-05-05T18:28:24.9610739Z Task         : Azure VM Image Builder Test
2020-05-05T18:28:24.9611277Z Description  : Build images using Azure Image Builder resource provider.
2020-05-05T18:28:24.9611608Z Version      : 1.0.18
2020-05-05T18:28:24.9612003Z Author       : Microsoft Corporation
2020-05-05T18:28:24.9612718Z Help         : For documentation, and end to end example, please visit: http://aka.ms/azvmimagebuilderdevops
2020-05-05T18:28:24.9613390Z ==============================================================================
2020-05-05T18:28:26.0651512Z start reading task parameters...
2020-05-05T18:28:26.0673377Z found build at:  d:\a\r1\a\_AppsAndImageBuilder\webApp
2020-05-05T18:28:26.0708785Z end reading parameters
2020-05-05T18:28:26.0745447Z getting storage account details for aibstagstor1565047758
2020-05-05T18:28:29.8812270Z created archive d:\a\_temp\temp_web_package_09737279437949953.zip
2020-05-05T18:28:33.1568013Z Source for image:  { type: 'PlatformImage',
2020-05-05T18:28:33.1584131Z   publisher: 'MicrosoftWindowsServer',
2020-05-05T18:28:33.1585965Z   offer: 'WindowsServer',
2020-05-05T18:28:33.1592768Z   sku: '2016-Datacenter',
2020-05-05T18:28:33.1594191Z   version: '14393.3630.2004101604' }
2020-05-05T18:28:33.1595387Z template name:  t_1588703313152
2020-05-05T18:28:33.1597453Z starting put template...
2020-05-05T18:28:52.9278603Z put template:  Succeeded
2020-05-05T18:28:52.9281282Z starting run template...
2020-05-05T19:33:14.3923479Z ##[error]The operation was canceled.
2020-05-05T19:33:14.3939721Z ##[section]Finishing: Azure VM Image Builder Task
```

#### Cause

If a user didn't cancel the build, the Azure DevOps user agent canceled it. Most likely, the one-hour timeout occurred because of Azure DevOps capabilities. If you're using a private project and agent, you get 60 minutes of build time. If the build exceeds the timeout, Azure DevOps cancels the running task.

For more information about Azure DevOps capabilities and limitations, see [Microsoft-hosted agents](/azure/devops/pipelines/agents/hosted#capabilities-and-limitations).

#### Solution

You can host your own Azure DevOps agents or try to reduce the time of your build. For example, if you're distributing to Azure Compute Gallery, you can replicate them to one region or replicate them asynchronously.

### Windows logon is slow

#### Error

This error might occur when you create a Windows 10 image by using VM Image Builder, create a VM from the image, and then use Remote Desktop Protocol (RDP). You wait several minutes at the first logon screen, and then a blue screen displays the following message:

```text
Please wait for the Windows Modules Installer
```

#### Solution

1. In the image build, ensure that:

   - There are no outstanding restarts required by adding a Windows restart customizer as the last customization.
   - All software installation is complete.

1. Add the [`/mode:vm`](/windows-hardware/manufacture/desktop/sysprep-command-line-options) option to the default `Sysprep` command that VM Image Builder uses. For more information, see [Overriding the commands](#overriding-the-commands) later in this article.

## Troubleshoot VMs created unsuccessfully from VM Image Builder

By default, VM Image Builder runs deprovisioning code at the end of each image customization phase to *generalize* the image. Generalizing an image sets it up for reuse in creating multiple VMs. As part of the process, you can pass in VM settings, such as host name and username.

In Windows, VM Image Builder runs a generic `Sysprep` command. However, this command might not be suitable for every successful Windows generalization. With VM Image Builder, you can customize the `Sysprep` command. VM Image Builder is an image automation tool that's responsible for running `Sysprep` commands successfully. But you might need different `Sysprep` commands to make your image reusable.

In Linux, VM Image Builder runs a generic `waagent -deprovision+user` command. For more information, see the [Microsoft Azure Linux Agent documentation](https://github.com/Azure/WALinuxAgent#command-line-options).

If you're migrating an existing customization and you're using various `Sysprep` or `waagent` commands, you can try the VM Image Builder generic commands. If the VM creation fails, use your previous `Sysprep` or `waagent` commands.

Let's suppose that you used VM Image Builder successfully to create a Windows custom image, but you failed to create a VM successfully from the image. For example, the VM creation fails to finish or it times out. In this event, do either of the following tasks:

- Review the Windows Server `Sysprep` documentation.
- Raise a support request with the Windows Server `Sysprep` customer support team. That team can help troubleshoot your problem and advise you on the correct `Sysprep` command.

### Command locations and file names

In Windows:

```
c:\DeprovisioningScript.ps1
```

In Linux:

```bash
/tmp/DeprovisioningScript.sh
```

### Sysprep command for Windows

```azurepowershell-interactive
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```

### Deprovision command for Linux

```bash
sudo /usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

### Overriding the commands

To override the commands, use the PowerShell or shell script provisioners to create the command files with the exact file name. Put them in the previously listed directories. VM Image Builder reads these commands and writes output to the `customization.log` file.

## Get support

If you used the guidance in this article and are still having problems, you can open a support case.

Use the following code to select the correct product and support topic. Doing so ensures that you're connected with the Azure VM Image Builder support team.

```text
Product Family: Azure
Product: Virtual Machine Running (Window\Linux)
Support Topic: Azure Features
Support Subtopic: Azure Image Builder
```

## Related content

- [Azure VM Image Builder overview](../image-builder-overview.md)
