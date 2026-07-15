---
title: Migrate from Azure Disk Encryption to encryption at host
description: Learn how to migrate Azure virtual machines from Azure Disk Encryption to encryption at host by creating new disks and VMs.
author: msmbaldwin
ms.date: 07/14/2026
ms.topic: how-to
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: references_regions
ai-usage: ai-assisted
# Customer intent: As a security administrator, I need to migrate my VMs from Azure Disk Encryption to encryption at host
---

# Migrate from Azure Disk Encryption to encryption at host

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

This article provides step-by-step guidance for migrating your virtual machines from Azure Disk Encryption (ADE) to encryption at host. The migration process requires creating new disks and VMs because in-place conversion isn't supported.

## Migration overview

Azure Disk Encryption (ADE) encrypts data within the VM by using BitLocker (Windows) or dm-crypt (Linux), while encryption at host encrypts data at the VM host level without consuming VM CPU resources. Encryption at host enhances Azure's default server-side encryption (SSE) by providing end-to-end encryption for all VM data, including temp disks, caches, and data flows between compute and storage.

For more information, see [Overview of managed disk encryption options](/azure/virtual-machines/disk-encryption-overview) and [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

### Migration limitations and considerations

Before you start the migration process, review these important limitations and considerations that affect your migration strategy:

- **No in-place migration**: You can't directly convert ADE-encrypted disks to encryption at host. Migration requires creating new disks and VMs.

- **Linux OS disk limitation**: Azure doesn't support disabling ADE on Linux OS disks. For Linux VMs with ADE-encrypted OS disks, create a new VM with a new OS disk.

- **Windows ADE encryption patterns**: On Windows VMs, Azure Disk Encryption can encrypt only the OS disk alone or all disks (OS + data disks). You can't encrypt only data disks on Windows VMs.

- **UDE flag persistence**: Disks encrypted with Azure Disk Encryption have a Unified Data Encryption (UDE) flag that persists even after decryption. Both snapshots and disk copies that use the Copy option retain this UDE flag. The migration requires creating new managed disks by using the Upload method and copying the VHD blob data, which creates a new disk object without any metadata from the source disk.

- **Downtime required**: The migration process requires VM downtime for disk operations and VM recreation.

- **Domain-joined VMs**: If your VMs are part of an Active Directory domain, more steps are required:
    - Remove the original VM from the domain before deletion.
    - After creating the new VM, rejoin it to the domain.
    - For Linux VMs, join the VM to a managed domain manually.

  For more information, see [What is Microsoft Entra Domain Services?](/entra/identity/domain-services/overview)

### Prerequisites

Before starting the migration:

1. **Back up your data**: Create backups of all critical data before beginning the migration process.

1. **Test the process**: If possible, test the migration process on a nonproduction VM first.

1. **Prepare encryption resources**: Ensure that your VM size supports encryption at host. Most current VM sizes support this feature. For more information about VM size requirements, see [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

1. **Document configuration**: Record your current VM configuration, including network settings, extensions, and attached resources.

## Migration steps

The following migration steps work for most scenarios, with specific differences noted for each operating system.

> [!IMPORTANT]
> You can't decrypt Linux VMs with encrypted OS disks in-place. For these VMs, create a new VM with a new OS disk and migrate your data. See the [Migrating Linux VMs with encrypted OS disks](#migrating-linux-vms-with-encrypted-os-disks) section after reviewing the general process below.

### Disable Azure Disk Encryption

First, disable the existing Azure Disk Encryption when possible:

- **Windows**: Follow the instructions in [Disable encryption and remove the encryption extension on Windows](windows/disk-encryption-windows.md#disable-encryption-and-remove-the-encryption-extension).
- **Linux**: If **only data disks** are encrypted, follow [Disable encryption and remove the encryption extension on Linux](linux/disk-encryption-linux.md#disable-encryption-and-remove-the-encryption-extension). If **the OS disk is encrypted**, see the [Migrating Linux VMs with encrypted OS disks](#migrating-linux-vms-with-encrypted-os-disks).

After running the ADE disable command, the VM encryption status in the Azure portal changes to "SSE + PMK" immediately. However, the actual decryption process at the OS level takes time and depends on the amount of data encrypted. You must verify that OS-level decryption is completed before proceeding to the next step.

**For Windows VMs:**

- Open Command Prompt as an administrator and run: `manage-bde -status`.
- Verify that all volumes show the "Fully Decrypted" status.
- Confirm that the decryption percentage shows 100% for all encrypted volumes.

**For Linux VMs (data disks only):**

- Run: `sudo cryptsetup status /dev/mapper/<device-name>`.
- Verify that encrypted devices are no longer active.
- Run `lsblk` to confirm that no encrypted mappings remain.

Wait for complete decryption before continuing with disk migration to ensure data integrity.

### Create new managed disks

Create new disks that don't carry over the ADE encryption metadata. This process works for both Windows and Linux VMs, with specific considerations for Linux OS disks.

# [CLI](#tab/CLI)

> [!IMPORTANT]
> Add an offset of 512 bytes when you copy a managed disk from Azure. Azure omits the footer when reporting the disk size. The copy fails if you don't add this offset. The following script adds this offset for you.
>
> If you create an OS disk, add `--hyper-v-generation <yourGeneration>` to `az disk create`.

```azurecli
# Set variables
sourceDiskName="MySourceDisk"
sourceRG="MyResourceGroup"
targetDiskName="MyTargetDisk"
targetRG="MyResourceGroup"
targetLocation="eastus"
# For OS disks, specify either "Windows" or "Linux"
# For data disks, omit the targetOS variable and --os-type parameter
targetOS="Windows"

# Get source disk size in bytes
sourceDiskSizeBytes=$(az disk show -g $sourceRG -n $sourceDiskName --query '[diskSizeBytes]' -o tsv)

# Create a new empty target disk with upload capability
az disk create -g $targetRG -n $targetDiskName -l $targetLocation --os-type $targetOS --for-upload --upload-size-bytes $(($sourceDiskSizeBytes+512)) --sku standard_lrs

# Generate SAS URIs for both disks
targetSASURI=$(az disk grant-access -n $targetDiskName -g $targetRG --access-level Write --duration-in-seconds 86400 --query [accessSas] -o tsv)

sourceSASURI=$(az disk grant-access -n $sourceDiskName -g $sourceRG --access-level Read --duration-in-seconds 86400 --query [accessSas] -o tsv)

# Copy the disk data by using AzCopy
azcopy copy $sourceSASURI $targetSASURI --blob-type PageBlob

# Revoke SAS access when complete
az disk revoke-access -n $sourceDiskName -g $sourceRG

az disk revoke-access -n $targetDiskName -g $targetRG
```

This method creates new disks without the Azure Disk Encryption metadata (UDE flag), which is essential for a clean migration.

# [Azure PowerShell](#tab/azurepowershell)

```azurepowershell
# Get source disk information
$sourceDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MySourceDisk"

# Create a new empty target disk
# For Windows OS disks
$diskConfig = New-AzDiskConfig -Location $sourceDisk.Location -CreateOption Upload `
  -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512) -OsType Windows `
  -HyperVGeneration "V2"

# For Linux OS disks (if not ADE-encrypted)
# $diskConfig = New-AzDiskConfig -Location $sourceDisk.Location -CreateOption Upload `
#   -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512) -OsType Linux `
#   -HyperVGeneration "V2"

# For data disks (no OS type needed)
# $diskConfig = New-AzDiskConfig -Location $sourceDisk.Location -CreateOption Upload `
#   -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512)

$targetDisk = New-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyTargetDisk" -Disk $diskConfig

# Generate SAS URIs and copy the data
# Get SAS URIs for both disks
$sourceSAS = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $sourceDisk.Name `
  -Access Read -DurationInSecond 7200
$targetSAS = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $targetDisk.Name `
  -Access Write -DurationInSecond 7200

# Copy the disk data by using AzCopy
azcopy copy $sourceSAS.AccessSAS $targetSAS.AccessSAS --blob-type PageBlob

# Revoke SAS access when complete
Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $sourceDisk.Name
Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $targetDisk.Name
```

---

### Create a new VM with encryption

Create a new VM by using the newly created disks with your chosen encryption method.

You can choose from several encryption options, depending on your security requirements. This article provides steps for creating a new VM with encryption at host, which is the most common migration path. For other encryption options, see [Overview of managed disk encryption options](/azure/virtual-machines/disk-encryption-overview).

#### Create a new VM with encryption at host

Encryption at host provides the closest equivalent to Azure Disk Encryption's coverage. This section covers encryption at host.

# [CLI](#tab/CLI2)

**For OS disks:**

```azurecli
# For Windows OS disks
az vm create
  --resource-group "MyResourceGroup"
  --name "MyVM-New"
  --os-type "Windows"
  --attach-os-disk "MyTargetDisk"
  --encryption-at-host true

# For Linux OS disks
# az vm create
#   --resource-group "MyResourceGroup"
#   --name "MyVM-New"
#   --os-type "Linux"
#   --attach-os-disk "MyTargetDisk"
#   --encryption-at-host true
```

**For data disks:**

```azurecli
# Enable encryption at host on the VM
az vm update
  --resource-group "MyResourceGroup"
  --name "MyVM-New"
  --encryption-at-host true

# Attach the newly created data disk
az vm disk attach
  --resource-group "MyResourceGroup"
  --vm-name "MyVM-New"
  --name "MyTargetDisk"
```

# [Azure PowerShell](#tab/azurepowershell2)

**For OS disks:**

```azurepowershell
# Define VM configuration
$vmConfig = New-AzVMConfig -VMName "MyVM-New" -VMSize "Standard_D2s_v3"

# Add the OS disk (Windows example)
$vmConfig = Set-AzVMOSDisk -VM $vmConfig -ManagedDiskId $targetDisk.Id -CreateOption Attach -Windows

# For Linux OS disk, use -Linux instead of -Windows

# Enable encryption at host
$vmConfig = Set-AzVMSecurityProfile -VM $vmConfig -EncryptionAtHost $true

# Create the VM with network settings (you'll need to specify your own)
New-AzVM -ResourceGroupName "MyResourceGroup" -Location $targetDisk.Location -VM $vmConfig
```

**For data disks:**

```azurepowershell
# Get the VM
$vm = Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM-New"

# Attach the data disk
$vm = Add-AzVMDataDisk -VM $vm -ManagedDiskId $targetDisk.Id -Lun 0 -CreateOption Attach

# Update the VM
Update-AzVM -ResourceGroupName "MyResourceGroup" -VM $vm
```

---

### Verify and configure the new disks

After creating the new VM with encryption at host, verify and configure the disks properly for your operating system.

# [CLI](#tab/CLI3)

**For Windows VMs:**

- Verify that disk letters are assigned correctly.
- Check that applications can access the disks correctly.
- Update any applications or scripts that reference specific disk IDs.

**For Linux VMs:**

- Update `/etc/fstab` with the new disk UUIDs.
- Mount the data disks to the correct mount points.

```bash
# Get UUIDs of all disks
sudo blkid

# Mount all disks defined in fstab
sudo mount -a
```

# [Azure PowerShell](#tab/azurepowershell3)

**For Windows VMs:**

- Verify that disk letters are assigned correctly by using disk management or PowerShell.
- Check that applications can access the disks correctly.

```powershell
# List all disks and their partitions
Get-Disk | Get-Partition | Format-Table -AutoSize

# Check drive letters
Get-PSDrive -PSProvider FileSystem
```

**For Linux VMs:**

- Connect to the Linux VM by using SSH to perform these tasks. You can use PowerShell to verify that the VM is running:

```powershell
# Verify VM is running
Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM-New" -Status
```

---

Both Windows and Linux might require more configuration steps specific to your applications or workloads.

### Verify encryption and cleanup

Verify that encryption at host is properly configured on both Windows and Linux VMs.

# [CLI](#tab/CLI4)

```azurecli
# Check encryption at host status
az vm show --resource-group "MyResourceGroup" --name "MyVM-New" --query "securityProfile.encryptionAtHost"
```

After confirming that encryption at host is working properly:

1. Test VM functionality to ensure applications work correctly.
1. Verify that data is accessible and intact.
1. Delete the original resources when you're satisfied with the migration:

```azurecli
# Delete the original VM
az vm delete --resource-group "MyResourceGroup" --name "MyVM-Original" --yes

# Delete the original disk
az disk delete --resource-group "MyResourceGroup" --name "MySourceDisk" --yes
```

# [Azure PowerShell](#tab/azurepowershell4)

```azurepowershell
# Check encryption at host status
Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM-New" | `
Select-Object -ExpandProperty SecurityProfile | Select-Object EncryptionAtHost

# Verify disk encryption status
Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyTargetDisk" | Select-Object Name, Encryption
```

After confirming that encryption at host is working properly:

1. Test VM functionality to ensure applications work correctly.
1. Verify that data is accessible and intact.
1. Delete the original resources when you're satisfied with the migration:

```azurepowershell
# Delete the original VM
Remove-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM-Original" -Force

# Delete the original disk
Remove-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MySourceDisk" -Force
```

---

## Migrating Linux VMs with encrypted OS disks

Because you can't disable encryption on Linux OS disks, the process is different from Windows.

# [CLI](#tab/CLI5)

1. Create a new VM with encryption at host enabled.

   ```azurecli
   az vm create \
     --resource-group "MyResourceGroup" \
     --name "MyVM-New" \
     --image "Ubuntu2204" \
     --encryption-at-host true \
     --admin-username "azureuser" \
     --generate-ssh-keys
   ```

1. For data migration options:
    - **Application data**: Use SCP, rsync, or other file transfer methods to copy data.
    - **Configuration**: Replicate important configuration files and settings.
    - **Complex applications**: Use backup and restore procedures appropriate for your applications.

   ```azurecli
   # Example of using SCP to copy files from source to new VM
   az vm run-command invoke -g MyResourceGroup -n MyVM-Original --command-id RunShellScript \
     --scripts "scp -r /path/to/data azureuser@new-vm-ip:/path/to/destination"
   ```

# [Azure PowerShell](#tab/azurepowershell5)

1. Create a new VM with encryption at host enabled.

   ```azurepowershell
   # Create a new VM with encryption at host
   $vmConfig = New-AzVMConfig -VMName "MyVM-New" -VMSize "Standard_D2s_v3" |
     Set-AzVMOperatingSystem -Linux -ComputerName "MyVM-New" -Credential (Get-Credential) |
     Set-AzVMSourceImage -PublisherName "Canonical" -Offer "UbuntuServer" -Skus "18.04-LTS" -Version "latest" |
     Set-AzVMSecurityProfile -EncryptionAtHost $true

   # Add networking and create the VM
   New-AzVM -ResourceGroupName "MyResourceGroup" -Location "EastUS" -VM $vmConfig
   ```

1. For data migration options:
    - **Application data**: Use PowerShell remoting or scripts to copy data.
    - **Configuration**: Replicate important configuration files and settings.
    - **Complex applications**: Use backup and restore procedures appropriate for your applications.

---

After creating the new VM:

1. Configure the new VM to match the original environment.
    - Set up the same network configurations.
    - Install the same applications and services.
    - Apply the same security settings.

1. Test thoroughly before decommissioning the original VM.

This approach works for both Windows and Linux VMs, but is especially important for Linux VMs with encrypted OS disks that you can't decrypt in-place.

For guidance on data migration, see [Upload a VHD to Azure](/azure/virtual-machines/linux/disks-upload-vhd-to-managed-disk-cli) and [Copy files to a Linux VM by using SCP](/azure/virtual-machines/linux/copy-files-to-linux-vm-using-scp).

## Recommended approach for AVD host pools

For Azure Virtual Desktop (AVD) environments, the recommended approach is to **redeploy session hosts** rather than attempting disk-level migration.

For both pooled and personal host pools, the supported approach is to replace Azure Disk Encryption (ADE)-enabled session hosts with new virtual machines that have encryption at host enabled.

### Steps

1. **Create a new golden image**.
    - Ensure that Azure Disk Encryption isn't enabled.
    - Validate applications and configurations.

1. **Deploy new session hosts**.
    - Use Azure Compute Gallery or a custom image.
    - Enable encryption at host at VM creation time. For more information, see [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

1. **Add new session hosts to the host pool**.
    - Ensure that session hosts are healthy and accepting connections.

1. **Validate workloads**.
    - Confirm user profile access, such as FSLogix.
    - Validate applications and policies.

1. **Drain existing ADE-enabled session hosts**.
    - Enable drain mode to block new sessions.
    - Wait for existing sessions to end, or manually sign off users.

1. **Remove and decommission old session hosts**.
    - Remove session hosts from the host pool.
    - Delete associated virtual machines and disks.

## Domain-joined VM considerations

If your VMs are members of an Active Directory domain, more steps are required during the migration process:

### Premigration domain steps

1. **Document domain membership**: Record the current domain, organizational unit (OU), and any special group memberships.
1. **Note computer account**: Manage the computer account in Active Directory.
1. **Back up domain-specific configurations**: Save any domain-specific settings, group policies, or certificates.

### Domain removal process

1. **Remove from domain**: Before deleting the original VM, remove it from the domain by using one of these methods.
    - Use the `Remove-Computer` PowerShell cmdlet on Windows.
    - Use the System Properties dialog to change to a workgroup.
    - Manually delete the computer account from Active Directory Users and Computers.

1. **Clean up Active Directory**: Remove any orphaned computer accounts or DNS entries.

### Post-migration domain rejoining

1. **Join new VM to domain**: After creating the new VM with encryption at host, join it to the domain.
    - **Windows**: Use the `Add-Computer` PowerShell cmdlet or System Properties.
    - **Linux**: Use manual configuration for the domain service that hosts your domain.

1. **Restore domain settings**: Reapply any domain-specific configurations, group policies, or certificates.

1. **Verify domain functionality**: Test domain authentication, group policy application, and network resource access.

### Linux domain joining

For Linux VMs that use Microsoft Entra Domain Services, join the VM to the managed domain by using manual Linux domain-join steps. For more information, see [Join an Ubuntu Linux virtual machine to a Microsoft Entra Domain Services managed domain](/azure/active-directory-domain-services/join-ubuntu-linux-vm).

### Important domain considerations

- The new VM has a different computer SID, which might affect some applications.
- Refresh Kerberos tickets and cached credentials.
- Some domain-integrated applications might require reconfiguration.
- Plan for potential temporary loss of domain services during migration.

## Post-migration verification

After completing the migration, verify that encryption at host is working correctly:

1. **Check encryption at host status**: Verify that encryption at host is enabled.

   ```azurecli
   az vm show --resource-group "MyResourceGroup" --name "MyVM-New" --query "securityProfile.encryptionAtHost"
   ```

1. **Test VM functionality**: Ensure your applications and services are working correctly.

1. **Verify disk encryption**: Confirm that disks are properly encrypted:

   ```azurepowershell
   Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" | Select-Object Name, DiskState
   ```

1. **Monitor performance**: Compare performance before and after migration to confirm the expected improvements.

For more information about encryption verification, see [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

## Cleanup

After successful migration and verification:

1. **Delete old VM**: Remove the original ADE-encrypted VM.
1. **Delete old disks**: Remove the original encrypted disks.
1. **Update Key Vault access policies**: Other disk encryption solutions use standard Key Vault authorization mechanisms. If you no longer need the Key Vault for Azure Disk Encryption, update its access policies to disable the special disk encryption setting:

   # [CLI](#tab/CLI-cleanup)

   ```azurecli
   az keyvault update --name "YourKeyVaultName" --resource-group "YourResourceGroup" --enabled-for-disk-encryption false
   ```

   # [Azure PowerShell](#tab/azurepowershell-cleanup)

   ```azurepowershell
   Update-AzKeyVault -VaultName "YourKeyVaultName" -ResourceGroupName "YourResourceGroup" -EnabledForDiskEncryption $false
   ```

   ---

1. **Clean up resources**: Remove any temporary resources created during migration.
1. **Update documentation**: Update your infrastructure documentation to reflect the migration to encryption at host.

## Common problems and solutions

### VM size doesn't support encryption at host

**Solution**: Check the [list of supported VM sizes](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) and resize your VM if necessary.

### VM fails to start after migration

**Solution**: Check that all disks are properly attached and that the OS disk is set as the boot disk.

### Encryption at host not enabled

**Solution**: Verify that the VM was created with the `--encryption-at-host true` parameter and that your subscription supports this feature.

### Performance problems persist

**Solution**: Verify that encryption at host is properly enabled and that the VM size supports the expected performance.

## Next steps

- [Overview of managed disk encryption options](/azure/virtual-machines/disk-encryption-overview)
- [Enable end-to-end encryption using encryption at host - Azure portal](/azure/virtual-machines/disks-enable-host-based-encryption-portal)
- [Enable end-to-end encryption using encryption at host - Azure PowerShell](/azure/virtual-machines/windows/disks-enable-host-based-encryption-powershell)
- [Enable end-to-end encryption using encryption at host - Azure CLI](/azure/virtual-machines/linux/disks-enable-host-based-encryption-cli)
- [Server-side encryption of Azure Disk Storage](/azure/virtual-machines/disk-encryption)
- [Azure Disk Encryption for Windows VMs](/azure/virtual-machines/windows/disk-encryption-overview)
- [Azure Disk Encryption for Linux VMs](/azure/virtual-machines/linux/disk-encryption-overview)
- [Azure Disk Encryption FAQ](/azure/virtual-machines/linux/disk-encryption-faq)
- [Upload a VHD to Azure or copy a managed disk to another region](/azure/virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell)
