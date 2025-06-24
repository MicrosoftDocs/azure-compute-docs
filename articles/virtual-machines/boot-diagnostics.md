---
title: Azure boot diagnostics
description: Overview of Azure boot diagnostics and managed boot diagnostics
services: virtual-machines
ms.service: azure-virtual-machines
ms.custom:
author: mimckitt
ms.author: mimckitt
ms.topic: concept-article
ms.date: 02/25/2023
ms.reviewer: mattmcinnes
---

# Azure boot diagnostics

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Boot diagnostics is a debugging feature for Azure virtual machines (VM) that allows diagnosis of VM boot failures. Boot diagnostics enables a user to observe the state of their VM as it is booting up by collecting serial log information and screenshots.

## Boot diagnostics storage account

When you create a VM in Azure portal, boot diagnostics is enabled by default. The recommended boot diagnostics experience is to use a managed storage account, as it yields significant performance improvements in the time to create an Azure VM. An Azure managed storage account is used, removing the time it takes to create a user storage account to store the boot diagnostics data.

> [!IMPORTANT]
> The boot diagnostics data blobs (which comprise of logs and snapshot images) are stored in a managed storage account. Customers will be charged only on used GiBs by the blobs, not on the disk's provisioned size. The snapshot meters will be used for billing of the managed storage account. Because the managed accounts are created on either Standard LRS or Standard ZRS, customers will be charged at $0.05/GB per month for the size of their diagnostic data blobs only. For more information on this pricing, see [Managed disks pricing](https://azure.microsoft.com/pricing/details/managed-disks/). Customers see this charge tied to their VM resource URI.

An alternative boot diagnostic experience is to use a custom storage account. A user can either create a new storage account or use an existing one. When the storage firewall is enabled on the custom storage account (**Enabled from all networks** option isn't selected), you must:

- Make sure that access through the storage firewall is allowed for the Azure platform to publish the screenshot and serial log. To do this, go to the custom boot diagnostics storage account in the Azure portal and then select **Networking** from the **Security + networking** section. Check if the **Allow Azure services on the trusted services list to access this storage account** checkbox is selected.

- Allow storage firewall for users to view the boot screenshots or serial logs. To do this, add your network or the client/browser's Internet IPs as firewall exclusions. For more information, see [Configure Azure Storage firewalls and virtual networks](/azure/storage/common/storage-network-security).

To configure the storage firewall for Azure Serial Console, see [Use Serial Console with custom boot diagnostics storage account firewall enabled](/troubleshoot/azure/virtual-machines/serial-console-windows#use-serial-console-with-custom-boot-diagnostics-storage-account-firewall-enabled).

> [!NOTE]
> The custom storage account associated with boot diagnostics requires the storage account and the associated virtual machines reside in the same region and subscription.

## Prerequisites for using a custom storage account

When you choose a **custom storage account** for boot diagnostics (instead of the recommended managed storage option), ensure that any user accessing **Boot diagnostics** through the Azure portal has the necessary permissions to retrieve and view the diagnostic data stored as blobs in the specified custom account.

In the Azure portal UI you might see a generic error message like:

![image](https://github.com/user-attachments/assets/c383951a-c7ba-40be-84e9-4ae17475a7dd)

This message alone does not provide the full cause. To identify the actual issue, inspect the REST API calls made by the portal using the **Network** tab in your browser's Developer Tools. This will help you understand the exact problem. For example:

```json
{
  "error": {
    "code": "AuthorizationFailed",
    "message": "The client '<object id>' does not have authorization to perform action 'Microsoft.Storage/storageAccounts/listKeys/action' over scope '/subscriptions/<subID>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>' or the scope is invalid. If access was recently granted, please refresh your credentials."
  }
}
```

Behind the scenes, the Azure portal:

1. Calls **Microsoft.Storage/storageAccounts/listKeys/action** to obtain a key for the storage account.
2. Uses this key to read the screenshot and serial log stored in the associated blob container.

Ensure that users who access Boot diagnostics on the VM blade have permission to list keys for the custom storage account so that the portal can retrieve the data successfully.

To follow the principle of least privilege, assign an appropriate built-in role, such as [Storage Account Key Operator Service Role](/azure/role-based-access-control/built-in-roles/storage#storage-account-key-operator-service-role) instead of broader roles. Alternatively, you can create a [custom role](articles/role-based-access-control/custom-roles-portal) that includes only the **Microsoft.Storage/storageAccounts/listKeys/action** permission.

> [!NOTE]
> Users with broad roles such as **Owner** or **Contributor** typically already have this permission and may not encounter this issue. This requirement is most relevant for users with more restricted or custom roles.

## Boot diagnostics view

Go to the virtual machine blade in the Azure portal, the boot diagnostics option is under the *Help* section in the Azure portal. Selecting boot diagnostics display a screenshot and serial log information. The serial log contains kernel messaging and the screenshot is a snapshot of your VMs current state. Based on if the VM is running Windows or Linux determines what the expected screenshot would look like. For Windows, users see a desktop background and for Linux, users see a login prompt.

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-linux.png" alt-text="Screenshot of Linux boot diagnostics":::
:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-windows.png" alt-text="Screenshot of Windows boot diagnostics":::

## Enable managed boot diagnostics

Managed boot diagnostics can be enabled through the Azure portal, CLI and ARM Templates.

### Enable managed boot diagnostics using the Azure portal

When you create a VM in the Azure portal, the default setting is to have boot diagnostics enabled using a managed storage account. Navigate to the *Management* tab during the VM creation to view it.

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-enable-portal.png" alt-text="Screenshot enabling managed boot diagnostics during VM creation.":::

### Enable managed boot diagnostics using CLI

Boot diagnostics with a managed storage account is supported in Azure CLI 2.12.0 and later. If you don't input a name or URI for a storage account, a managed account is used. For more information and code samples, see the [CLI documentation for boot diagnostics](/cli/azure/vm/boot-diagnostics).

### Enable managed boot diagnostics using PowerShell

Boot diagnostics with a managed storage account is supported in Azure PowerShell 6.6.0 and later. If you don't input a name or URI for a storage account, a managed account is used. For more information and code samples, see the [PowerShell documentation for boot diagnostics](/powershell/module/az.compute/set-azvmbootdiagnostic).

### Enable managed boot diagnostics using Azure Resource Manager (ARM) templates

Everything after API version 2020-06-01 supports managed boot diagnostics. For more information, see [boot diagnostics instance view](/rest/api/compute/virtualmachines/createorupdate#bootdiagnostics).

```ARM Template
            "name": "[parameters('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        }
                    },
                    "imageReference": {
                        "publisher": "publisherName",
                        "offer": "imageOffer",
                        "sku": "imageSKU",
                        "version": "imageVersion"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaceName'))]"
                        }
                    ]
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachineComputerName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "linuxConfiguration": {
                        "disablePasswordAuthentication": true
                    }
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true
                    }
                }
            }
        }
    ],

```

> [!NOTE]
> Replace publisherName, imageOffer, imageSKU and imageVersion accordingly.

## Limitations

- Managed boot diagnostics is only available for Azure Resource Manager VMs.
- Managed boot diagnostics doesn't support VMs using unmanaged OS disks.
- Boot diagnostics doesn't support premium storage accounts or zone redundant storage accounts. If either of these are used for boot diagnostics users receive an `StorageAccountTypeNotSupported` error when starting the VM.
- Managed storage accounts are supported in Resource Manager API version "2020-06-01" and later.
- Portal only supports the use of boot diagnostics with a managed storage account for single instance VMs.
- Users can't configure a retention period for Managed Boot Diagnostics. The logs are overwritten when the total size crosses 1 GB.

## Next steps

Learn more about the [Azure Serial Console](/troubleshoot/azure/virtual-machines/serial-console-overview) and how to use boot diagnostics to [troubleshoot virtual machines in Azure](/troubleshoot/azure/virtual-machines/boot-diagnostics).
