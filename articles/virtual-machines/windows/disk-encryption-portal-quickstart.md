---
title: Create and encrypt a Windows VM by using the Azure portal
description: In this quickstart, you learn how to use the Azure portal to create and encrypt a Windows virtual machine.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: windows
ms.topic: quickstart
ms.date: 07/14/2026
ai-usage: ai-assisted
ms.custom:
   - mode-ui
   - sfi-image-nochange
# Customer intent: As a cloud administrator, I want to create and encrypt a Windows virtual machine using a browser-based portal, so that I can secure sensitive data in compliance with security best practices.
---

# Quickstart: Create and encrypt a Windows virtual machine by using the Azure portal

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

**Applies to:** :heavy_check_mark: Windows VMs

You can create Azure virtual machines (VMs) through the Azure portal. The Azure portal is a browser-based user interface to create VMs and their associated resources. In this quickstart, you use the Azure portal to deploy a Windows virtual machine, create a key vault for the storage of encryption keys, and encrypt the VM.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).


## Create a virtual machine

1. Select **Create a resource** in the upper left corner of the Azure portal.
1. On the **New** page, under **Popular**, select **Windows Server 2016 Datacenter**.
1. On the **Basics** tab, under **Project details**, make sure the correct subscription is selected.
1. For **Resource group**, select **Create new**. Enter *myResourceGroup* as the name and select **OK**.
1. For **Virtual machine name**, enter *MyVM*.
1. For **Region**, select *(US) East US*.
1. Verify that the **Size** is *Standard D2s v3*.
1. Under **Administrator account**, select **Password**. Enter a user name and a password.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-windows-vm-creation.png" alt-text="Screenshot that shows Azure portal Basics tab for Windows VM creation.":::

    > [!WARNING]
    > The "Disks" tab features an "Encryption Type" field under **Disk options**. This field is used to specify encryption options for [managed disks](../managed-disks-overview.md) + CMK, not for Azure Disk Encryption.
    >
    > To avoid confusion, we suggest you skip the *Disks* tab entirely while completing this tutorial.

1. Select the **Management** tab and verify that you have a diagnostics storage account. If you have no storage accounts, select **Create new**, give your new account a name, and select **OK**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-vm-creation-storage.png" alt-text="Screenshot that shows Azure portal Management tab with diagnostics storage account settings.":::

1. Select **Review + create**.
1. On the **Create a virtual machine** page, you can see the details about the VM you're about to create. When you're ready, select **Create**.

It takes a few minutes to deploy your VM. When the deployment is finished, move on to the next section.

## Encrypt the virtual machine

1. When the VM deployment is complete, select **Go to resource**.
1. On the left-hand sidebar, select **Disks**.
1. On the top bar, select **Additional settings**.
1. Under **Encryption settings** > **Disks to encrypt**, select **OS and data disks**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-disks-to-encryption.png" alt-text="Screenshot that shows OS and data disks.":::

1. Under **Encryption settings**, select **Select a key vault and key for encryption**.
1. On the **Select key from Azure Key Vault** screen, select **Create new**.

    :::image type="content" source="../media/disk-encryption/portal-qs-keyvault-create.png" alt-text="Screenshot that shows the Create new option.":::

1. To the left of **Key vault and key**, select **Click to select a key**.
1. On the **Select key from Azure Key Vault**, under the **Key Vault** field, select **Create new**.
1. On the **Create key vault** screen, ensure that the resource group is *myResourceGroup*, and give your key vault a name. Every key vault in Azure must have a unique name.
1. On the **Access policies** tab, check the **Azure Disk Encryption for volume encryption** box.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-keyvault-enable.png" alt-text="Screenshot that shows Azure portal Access policies tab with Azure Disk Encryption for volume encryption selected.":::

1. Select **Review + create**.
1. After the key vault passes validation, select **Create**. You return to the **Select key from Azure Key Vault** screen.
1. Leave the **Key** field blank and choose **Select**.
1. At the top of the encryption screen, select **Save**. A pop-up warns you that the VM reboots. Select **Yes**.

## Clean up resources

When you no longer need these resources, you can delete the resource group, virtual machine, and all related resources. To delete these resources, select the resource group for the virtual machine, select **Delete**, and then confirm the name of the resource group to delete.

## Next steps

In this quickstart, you created a key vault that was enabled for encryption keys, created a virtual machine, and enabled the virtual machine for encryption.

> [!div class="nextstepaction"]
> [Azure Disk Encryption overview](disk-encryption-overview.md)
