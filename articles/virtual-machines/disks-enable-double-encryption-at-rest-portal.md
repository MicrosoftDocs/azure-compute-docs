---
title: Enable double encryption at rest for managed disks
description: Enable double encryption at rest for your managed disk data using the Azure portal, Azure PowerShell, or Azure CLI.
author: roygara
ms.date: 09/16/2026
ms.topic: how-to
ms.author: rogarana
ms.service: azure-disk-storage
ms.custom:  devx-track-azurecli, linux-related-content, devx-track-azurepowershell, portal
ai-usage: ai-assisted
# Customer intent: As a cloud administrator, I want to enable double encryption at rest for managed disks using the Azure portal, Azure CLI, or Azure PowerShell, so that I can enhance the security of my virtual machine data.
---

# Enable double encryption at rest for managed disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark:

Azure Disk Storage supports double encryption at rest for managed disks. For conceptual information on double encryption at rest, and other managed disk encryption types, see the [Double encryption at rest](disk-encryption.md#double-encryption-at-rest) section of our disk encryption article.

## Restrictions

Double encryption at rest isn't currently supported with either Ultra Disks or Premium SSD v2 disks.

## Prerequisites

If you're going to use Azure CLI, install the latest [Azure CLI](/cli/azure/install-az-cli2) and sign in to an Azure account with [az login](/cli/azure/reference-index).

If you're going to use Azure PowerShell, install the latest [Azure PowerShell version](/powershell/azure/install-azure-powershell), and sign in to an Azure account by using [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

If you create a key vault, enable soft delete and purge protection. Soft delete retains a deleted key for the retention period, which is 90 days by default. Purge protection prevents permanent deletion until that period ends. Both settings are mandatory when you use Azure Key Vault to encrypt managed disks.

## Enable double encryption at rest

# [Azure portal](#tab/portal)

### Enable double encryption in the Azure portal

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and select **Disk Encryption Sets**.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-encryption-sets-search.png" alt-text="Screenshot of the Azure portal search results with Disk Encryption Sets highlighted." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-encryption-sets-search.png":::

1. Select **+ Create**.
1. Select one of the supported regions.
1. For **Encryption type**, select **Double encryption with platform-managed and customer-managed keys**.

    > [!NOTE]
    > Once you create a disk encryption set with a particular encryption type, it cannot be changed. If you want to use a different encryption type, you must create a new disk encryption set.

1. Fill in the remaining info.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-create-disk-encryption-set-blade.png" alt-text="Screenshot of disk encryption set creation with West US 2 selected and Double encryption with platform-managed and customer-managed keys selected." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-create-disk-encryption-set-blade.png":::

1. Select an Azure Key Vault and key, or create a new one if necessary.

    > [!NOTE]
    > If you create a key vault, enable soft delete and purge protection as described in the prerequisites.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-select-key-vault.png" alt-text="Screenshot of the Select key from Azure Key Vault pane with a key vault and key selected." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-select-key-vault.png":::

1. Select **Create**.
1. Navigate to the disk encryption set you created, and then select the alert to grant the required key vault permissions.

    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-set-error.png" alt-text="Screenshot of an alert that requires granting the disk encryption set permission to the selected key vault." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-disk-set-error.png":::

    The notifications confirm that the role was assigned and the key vault permissions were granted.
    
    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/disk-encryption-notification-success.png" alt-text="Screenshot of notifications confirming that the role was assigned and key vault permissions were granted." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/disk-encryption-notification-success.png":::

1. Navigate to your disk.
1. Select **Encryption**.
1. For **Key management**, select one of the keys under **Platform-managed and customer-managed keys**.
1. Select **Save**.
    
    :::image type="content" source="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-enable-disk-blade.png" alt-text="Screenshot of the managed disk Encryption pane with a platform-managed and customer-managed key selected and Save highlighted." lightbox="media/virtual-machines-disks-double-encryption-at-rest-portal/double-encryption-enable-disk-blade.png":::

You have now enabled double encryption at rest on your managed disk.

# [Azure CLI](#tab/azure-cli)

### Enable double encryption with Azure CLI

1. Create a key vault and encryption key.

    Use [az account set](/cli/azure/account#az-account-set) to select the subscription, [az keyvault create](/cli/azure/keyvault#az-keyvault-create) to create a key vault with soft delete and purge protection enabled, and [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create) to create the encryption key.

    
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
    
1. Use [az keyvault key show](/cli/azure/keyvault/key#az-keyvault-key-show) to get the URL of the key you created.
    
    ```azurecli
    az keyvault key show --name $keyName --vault-name $keyVaultName
    ```

1. Use [az disk-encryption-set create](/cli/azure/disk-encryption-set#az-disk-encryption-set-create) to create a disk encryption set with the encryption type set to `EncryptionAtRestWithPlatformAndCustomerKeys`. Replace `yourKeyURL` with the URL returned by `az keyvault key show`.

    ```azurecli
    az disk-encryption-set create --resource-group $rgName --name $diskEncryptionSetName --key-url yourKeyURL --source-vault $keyVaultName --encryption-type EncryptionAtRestWithPlatformAndCustomerKeys
    ```

1. Grant the disk encryption set access to the key vault. Use [az disk-encryption-set show](/cli/azure/disk-encryption-set#az-disk-encryption-set-show) to get its principal ID and [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) to grant the required key permissions.

    > [!NOTE]
    > It might take a few minutes for Azure to create the identity of your disk encryption set in Microsoft Entra ID. If the command returns a "Cannot find the Active Directory object" error, wait a few minutes and try again.

    ```azurecli
    desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)

    az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
    ```

1. Apply the disk encryption set to the managed disk. The disk must not be attached to a running VM. Use [az disk update](/cli/azure/disk#az-disk-update) to configure double encryption, then use [az disk show](/cli/azure/disk#az-disk-show) to verify the encryption type.

    ```azurecli
    diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

    az disk update -n $diskName -g $rgName \
      --encryption-type EncryptionAtRestWithPlatformAndCustomerKeys \
      --disk-encryption-set $diskEncryptionSetId

    az disk show -n $diskName -g $rgName --query encryption.type -o tsv
    ```

    Verify that the command returns `EncryptionAtRestWithPlatformAndCustomerKeys`.

# [Azure PowerShell](#tab/azure-powershell)

### Enable double encryption with Azure PowerShell

1. Create a key vault and encryption key.

    Use [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) to create a key vault with soft delete and purge protection enabled, then use [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) to create the encryption key.

    ```powershell
    $ResourceGroupName="yourResourceGroupName"
    $LocationName="westus2"
    $keyVaultName="yourKeyVaultName"
    $keyName="yourKeyName"
    $keyDestination="Software"
    $diskEncryptionSetName="yourDiskEncryptionSetName"
    $diskName="yourDiskName"
    
    $keyVault = New-AzKeyVault -Name $keyVaultName -ResourceGroupName $ResourceGroupName -Location $LocationName -EnableSoftDelete -EnablePurgeProtection
    
    $key = Add-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName -Destination $keyDestination  
    ```

1. Use [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey) to retrieve the key URL for subsequent commands.

    ```powershell
    Get-AzKeyVaultKey -VaultName $keyVaultName -KeyName $keyName
    ```

1. Use [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) to retrieve the key vault resource ID for subsequent commands.

    ```powershell
    Get-AzKeyVault -VaultName $keyVaultName
    ```

1. Use [New-AzDiskEncryptionSetConfig](/powershell/module/az.compute/new-azdiskencryptionsetconfig) and [New-AzDiskEncryptionSet](/powershell/module/az.compute/new-azdiskencryptionset) to create a disk encryption set with the encryption type set to `EncryptionAtRestWithPlatformAndCustomerKeys`. Replace `yourKeyURL` and `yourKeyVaultURL` with the values retrieved earlier.

    ```powershell
    $config = New-AzDiskEncryptionSetConfig -Location $locationName -KeyUrl "yourKeyURL" -SourceVaultId 'yourKeyVaultURL' -IdentityType 'SystemAssigned'
    
    $config | New-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName -EncryptionType EncryptionAtRestWithPlatformAndCustomerKeys
    ```

1. Grant the disk encryption set access to the key vault. Use [Get-AzDiskEncryptionSet](/powershell/module/az.compute/get-azdiskencryptionset) to get its identity and [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) to grant the required key permissions.

    > [!NOTE]
    > It might take a few minutes for Azure to create the identity of your disk encryption set in Microsoft Entra ID. If the command returns a "Cannot find the Active Directory object" error, wait a few minutes and try again.

    ```powershell  
    $des=Get-AzDiskEncryptionSet -name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
    ```

1. Apply the disk encryption set to the managed disk. The disk must not be attached to a running VM. Use [New-AzDiskUpdateConfig](/powershell/module/az.compute/new-azdiskupdateconfig) and [Update-AzDisk](/powershell/module/az.compute/update-azdisk) to configure double encryption, then use [Get-AzDisk](/powershell/module/az.compute/get-azdisk) to verify the encryption type.

    ```powershell
    $diskEncryptionSet = Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

    New-AzDiskUpdateConfig `
        -EncryptionType "EncryptionAtRestWithPlatformAndCustomerKeys" `
        -DiskEncryptionSetId $diskEncryptionSet.Id | `
        Update-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $diskName

    $disk = Get-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $diskName
    $disk.Encryption.Type
    ```

    Verify that the command returns `EncryptionAtRestWithPlatformAndCustomerKeys`.

---

## Next steps

- [Azure PowerShell - Enable customer-managed keys with server-side encryption - managed disks](./windows/disks-enable-customer-managed-keys-powershell.md)
- [Azure Resource Manager template samples](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Enable customer-managed keys with server-side encryption - Examples](./linux/disks-enable-customer-managed-keys-cli.md#examples)
