---
title: Enable double encryption at rest for managed disks
description: Enable double encryption at rest for your managed disk data using the Azure portal, Azure PowerShell module, or Azure CLI.
author: roygara

ms.date: 09/26/2024
ms.topic: how-to
ms.author: rogarana
ms.service: azure-disk-storage
ms.custom:  devx-track-azurecli, linux-related-content, devx-track-azurepowershell
# Customer intent: As a cloud administrator, I want to enable double encryption at rest for managed disks using the Azure portal, CLI, or PowerShell, so that I can enhance the security of my virtual machine data.
---

# Enable double encryption at rest for managed disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark:

Azure Disk Storage supports double encryption at rest for managed disks. For conceptual information on double encryption at rest, and other managed disk encryption types, see the [Double encryption at rest](disk-encryption.md#double-encryption-at-rest) section of our disk encryption article.

## Restrictions

Double encryption at rest isn't currently supported with either Ultra Disks or Premium SSD v2 disks.

## Prerequisites

If you're going to use Azure CLI, install the latest [Azure CLI](/cli/azure/install-az-cli2) and sign in to an Azure account with [az login](/cli/azure/reference-index).

If you're going to use the Azure PowerShell module, install the latest [Azure PowerShell version](/powershell/azure/install-azure-powershell), and sign in to an Azure account using [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

## Getting started

# [Azure portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and select **Disk Encryption Sets**.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-encryption-sets-search.png" alt-text="Screenshot of the main Azure portal, disk encryption sets is highlighted in the search bar." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-encryption-sets-search.png":::

1. Select **+ Create**.
1. Select one of the supported regions.
1. For **Encryption type**, select **Double encryption with platform-managed and customer-managed keys**.

    > [!NOTE]
    > Once you create a disk encryption set with a particular encryption type, it cannot be changed. If you want to use a different encryption type, you must create a new disk encryption set.

1. Fill in the remaining info.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-create-disk-encryption-set-blade.png" alt-text="Screenshot of the disk encryption set creation blade, regions and double encryption with platform-managed and customer-managed keys are highlighted." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-create-disk-encryption-set-blade.png":::

1. Select an Azure Key Vault and key, or create a new one if necessary.

    > [!NOTE]
    > If you create a Key Vault instance, you must enable soft delete and purge protection. These settings are mandatory when using a Key Vault for encrypting managed disks, and protect you from losing data due to accidental deletion.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-select-key-vault.png" alt-text="Screenshot of the Key Vault creation blade." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-select-key-vault.png":::

1. Select **Create**.
1. Navigate to the disk encryption set you created, and select the error that is displayed. This will configure your disk encryption set to work.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-set-error.png" alt-text="Screenshot of the disk encryption set displayed error, the error text is: To associate a disk, image, or snapshot with this disk encryption set, you must grant permissions to the key vault." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-set-error.png":::

    A notification should pop up and succeed. Doing this will allow you to use the disk encryption set with your key vault.
    
    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/disk-encryption-notification-success.png" alt-text="Screenshot of successful permission and role assignment for your key vault." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/disk-encryption-notification-success.png":::

1. Navigate to your disk.
1. Select **Encryption**.
1. For **Key management**, select one of the keys under **Platform-managed and customer-managed keys**.
1. select **Save**.
    
    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-enable-disk-blade.png" alt-text="Screenshot of the encryption blade for your managed disk, the aforementioned encryption type is highlighted." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-enable-disk-blade.png":::

You have now enabled double encryption at rest on your managed disk.

# [Azure CLI](#tab/azure-cli)

1. Create an instance of Azure Key Vault and encryption key.

    When creating the Key Vault instance, you must enable soft delete and purge protection. Soft delete ensures that the Key Vault holds a deleted key for a given retention period (90 day default). Purge protection ensures that a deleted key can't be permanently deleted until the retention period lapses. These settings protect you from losing data due to accidental deletion. These settings are mandatory when using a Key Vault for encrypting managed disks.

    
    ```azurecli
    subscriptionId=yourSubscriptionID
    rgName=yourResourceGroupName
    location=westcentralus
    keyVaultName=yourKeyVaultName
    keyName=yourKeyName
    diskEncryptionSetName=yourDiskEncryptionSetName
    diskName=yourDiskName
    
    az account set --subscription $subscriptionId
    
    az keyvault create -n $keyVaultName -g $rgName -l $location --enable-purge-protection true --enable-soft-delete true
    
    az keyvault key create --vault-name $keyVaultName -n $keyName --protection software
    ```
    
1. Get the key URL of the key you created with `az keyvault key show`.
    
    ```azurecli
    az keyvault key show --name $keyName --vault-name $keyVaultName
    ```

1.    Create a DiskEncryptionSet with encryptionType set as EncryptionAtRestWithPlatformAndCustomerKeys. Replace `yourKeyURL` with the URL you received from `az keyvault key show`. 

        ```azurecli
        az disk-encryption-set create --resource-group $rgName --name $diskEncryptionSetName --key-url yourKeyURL --source-vault $keyVaultName --encryption-type EncryptionAtRestWithPlatformAndCustomerKeys
        ```

1.    Grant the DiskEncryptionSet resource access to the key vault. 


> [!NOTE]
> It may take few minutes for Azure to create the identity of your DiskEncryptionSet in your Microsoft Entra ID. If you get an error like "Cannot find the Active Directory object" when running the following command, wait a few minutes and try again.
    
```azurecli
desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)

az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
```

# [Azure PowerShell](#tab/azure-powershell)


1. Create an instance of Azure Key Vault and encryption key.

    When creating the Key Vault instance, you must enable soft delete and purge protection. Soft delete ensures that the Key Vault holds a deleted key for a given retention period (90 day default). Purge protection ensures that a deleted key can't be permanently deleted until the retention period lapses. These settings protect you from losing data due to accidental deletion. These settings are mandatory when using a Key Vault for encrypting managed disks.

    ```powershell
    $ResourceGroupName="yourResourceGroupName"
    $LocationName="westus2"
    $keyVaultName="yourKeyVaultName"
    $keyName="yourKeyName"
    $keyDestination="Software"
    $diskEncryptionSetName="yourDiskEncryptionSetName"
    
    $keyVault = New-AzKeyVault -Name $keyVaultName -ResourceGroupName $ResourceGroupName -Location $LocationName -EnableSoftDelete -EnablePurgeProtection
    
    $key = Add-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName -Destination $keyDestination  
    ```

1. Retrieve the URL for the key you created, you'll need it for subsequent commands. The ID output from `Get-AzKeyVaultKey` is the key URL. 

    ```powershell
    Get-AzKeyVaultKey -VaultName $keyVaultName -KeyName $keyName
    ```

1. Get the resource ID for the Key Vault instance you created, you'll need it for subsequent commands.

    ```powershell
    Get-AzKeyVault -VaultName $keyVaultName
    ```

1.  Create a DiskEncryptionSet with encryptionType set as EncryptionAtRestWithPlatformAndCustomerKeys. Replace `yourKeyURL` and `yourKeyVaultURL` with the URLs you retrieved earlier.

    ```powershell
    $config = New-AzDiskEncryptionSetConfig -Location $locationName -KeyUrl "yourKeyURL" -SourceVaultId 'yourKeyVaultURL' -IdentityType 'SystemAssigned'
    
    $config | New-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName -EncryptionType EncryptionAtRestWithPlatformAndCustomerKeys
    ```

1. Grant the DiskEncryptionSet resource access to the key vault.

    > [!NOTE]
    > It may take few minutes for Azure to create the identity of your DiskEncryptionSet in your Microsoft Entra ID. If you get an error like "Cannot find the Active Directory object" when running the following command, wait a few minutes and try again.

    ```powershell  
    $des=Get-AzDiskEncryptionSet -name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
    ```

---

## Next steps

- [Azure PowerShell - Enable customer-managed keys with server-side encryption - managed disks](./windows/disks-enable-customer-managed-keys-powershell.md)
- [Azure Resource Manager template samples](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Enable customer-managed keys with server-side encryption - Examples](./linux/disks-enable-customer-managed-keys-cli.md#examples)