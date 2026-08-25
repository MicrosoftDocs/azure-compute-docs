---
title: Troubleshooting Azure Disk Encryption for Linux VMs
description: This article provides troubleshooting tips for Microsoft Azure Disk Encryption for Linux VMs.
author: msmbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.collection: linux
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 07/14/2026
ai-usage: ai-assisted
ms.custom: linux-related-content
# Customer intent: "As a cloud administrator, I want to troubleshoot Azure Disk Encryption for Linux VMs, so that I can successfully encrypt disk drives while ensuring compliance with security protocols and minimizing downtime."
---
# Azure Disk Encryption for Linux VMs troubleshooting guide

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

This guide is for IT professionals, information security analysts, and cloud administrators whose organizations use Azure Disk Encryption. Use this article to troubleshoot disk-encryption-related problems.

Before you take any of these steps, first ensure that the VMs you're attempting to encrypt are among the [supported VM sizes and operating systems](disk-encryption-overview.md#supported-vms-and-operating-systems), and that you meet all the prerequisites:

- [Additional requirements for VMs](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Networking requirements](disk-encryption-overview.md#networking-requirements)
- [Encryption key storage requirements](disk-encryption-overview.md#encryption-key-storage-requirements)


## Troubleshooting Linux OS disk encryption

Linux operating system (OS) disk encryption must unmount the OS drive before running it through the full disk encryption process. If it can't unmount the drive, an error message of "failed to unmount after …" is likely to occur.

This error might occur when you attempt OS disk encryption on a VM with an environment that changed from the supported stock gallery image. Deviations from the supported image can interfere with the extension’s ability to unmount the OS drive. Examples of deviations include the following items:
- Customized images no longer match a supported file system or partitioning scheme.
- Large applications such as SAP, MongoDB, Apache Cassandra, and Docker aren't supported when they're installed and running in the OS before encryption. Azure Disk Encryption can't shut down these processes safely as required in preparation of the OS drive for disk encryption. If there are still active processes holding open file handles to the OS drive, Azure Disk Encryption can't unmount the OS drive, resulting in a failure to encrypt the OS drive.
- Custom scripts that run in close time proximity to the encryption being enabled, or if any other changes are being made on the VM during the encryption process. This conflict can happen when an Azure Resource Manager template defines multiple extensions to execute simultaneously, or when a custom script extension or other action runs simultaneously to disk encryption. Serializing and isolating such steps might resolve the problem.
- Security Enhanced Linux (SELinux) isn't disabled before you enable encryption, so the unmount step fails. SELinux you can reenable after encryption is complete.
- The OS disk uses a Logical Volume Manager (LVM) scheme. Although limited LVM data disk support is available, an LVM OS disk isn't.
- Minimum memory requirements aren't met (7 GB is suggested for OS disk encryption).
- Data drives are recursively mounted under the /mnt/ directory, or each other (for example, /mnt/data1, /mnt/data2, /data3 + /data3/data4).

## Update the default kernel for Ubuntu 14.04 LTS

The Ubuntu 14.04 LTS image ships with a default kernel version 4.4. This kernel version has a known problem in which the Out of Memory (OOM) Killer improperly terminates the `dd` command during the OS encryption process. This bug is fixed in the most recent Azure tuned Linux kernel. To avoid this error, before enabling encryption on the image, update to the Azure tuned kernel 4.15 or later by using the following commands:

```bash
sudo apt-get update
sudo apt-get install linux-azure
sudo reboot
```

After the VM restarts into the new kernel, check the new kernel version by running:

```bash
uname -a
```

## Update the Azure Virtual Machine Agent and extension versions

Azure Disk Encryption operations might fail on virtual machine images that use unsupported versions of the Azure Virtual Machine Agent. Update Linux images with agent versions earlier than 2.2.38 before you enable encryption. For more information, see [How to update the Azure Linux Agent on a VM](../extensions/update-linux-agent.md) and [Minimum version support for virtual machine agents in Azure](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

The correct version of the Microsoft.Azure.Security.AzureDiskEncryption or Microsoft.Azure.Security.AzureDiskEncryptionForLinux guest agent extension is also required. Extension versions are maintained and updated automatically by the platform when Azure Virtual Machine agent prerequisites are satisfied and a supported version of the virtual machine agent is used.

The Microsoft.OSTCExtensions.AzureDiskEncryptionForLinux extension is deprecated and no longer supported.

## Unable to encrypt Linux disks

In some cases, Linux disk encryption appears to be stuck at **OS disk encryption started** and SSH is disabled. The encryption process can take 3 to 16 hours to finish on a stock gallery image. If you add multiterabyte-sized data disks, the process might take days.

The Linux OS disk encryption sequence unmounts the OS drive temporarily. It then performs block-by-block encryption of the entire OS disk, before it remounts it in its encrypted state. Linux Disk Encryption doesn't allow for concurrent use of the VM while the encryption is in progress. The performance characteristics of the VM can make a significant difference in the time required to complete encryption. These characteristics include the size of the disk and whether the storage account is standard or premium (SSD) storage.

While the OS drive is being encrypted, the VM enters a servicing state and disables SSH to prevent any disruption to the ongoing process.  To check the encryption status, use the Azure PowerShell [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) command, and check the **ProgressMessage** field. **ProgressMessage** will report a series of statuses as the data and OS disks are encrypted:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings :
ProgressMessage            : Transitioning

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for data volumes

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OS disk encryption started
```

The **ProgressMessage** will remain in **OS disk encryption started** for most of the encryption process.  When encryption is complete and successful, **ProgressMessage** will return:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NotMounted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for all volumes
```

After this message is available, the encrypted OS drive is ready for use and you can use the VM again.

If the boot information, the progress message, or an error reports that OS encryption fails in the middle of this process, restore the VM to the snapshot or backup taken immediately before encryption. An example of a message is the "failed to unmount" error described in this guide.

Before reattempting encryption, reevaluate the characteristics of the VM and make sure that all of the prerequisites are satisfied.

## Troubleshooting Azure Disk Encryption behind a firewall

See [Disk Encryption on an isolated network](disk-encryption-isolated-network.md)

## Troubleshooting encryption status

The portal might display a disk as encrypted even after you unencrypt it within the VM. This problem can occur when you use low-level commands to directly unencrypt the disk from within the VM instead of using the higher-level Azure Disk Encryption management commands. The higher-level commands unencrypt the disk from within the VM. Outside of the VM, they also update important platform-level encryption settings and extension settings associated with the VM. If these settings aren't kept in alignment, the platform can't report encryption status or provision the VM properly.

To disable Azure Disk Encryption with PowerShell, use [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) followed by [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). Running Remove-AzVMDiskEncryptionExtension before the encryption is disabled will fail.

To disable Azure Disk Encryption with CLI, use [az vm encryption disable](/cli/azure/vm/encryption).

## Next steps

In this document, you learned more about some common problems in Azure Disk Encryption and how to troubleshoot those problems. For more information about this service and its capabilities, see the following articles:

- [Apply disk encryption in Microsoft Defender for Cloud](/azure/security-center/asset-inventory)
- [Azure data encryption at rest](/azure/security/fundamentals/encryption-atrest)
