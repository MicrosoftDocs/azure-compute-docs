---
title: Deprovision or generalize a VM before creating an image
description: Generalized or deprovision VM to remove machine specific information before creating an image.
author: cynthn
ms.author: cynthn
ms.date: 01/28/2026
ms.service: azure-virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.custom:
  - portal
  - sysprep
  - waagent
  - linux
# Customer intent: As a cloud administrator, I want to create an image of a VM that doesn't contain VM specific information that might cause collisions, so I need to deprovision or generalize my VM first. 
---

# Remove machine specific information by deprovisioning or generalizing a VM before creating an image

Generalizing removes machine specific information so the image can be used to create multiple VMs. Once the VM has been generalized or deprovisioned, you need to let the platform know so that the boot sequence can be set correctly. 

Generalizing or deprovisioning a VM isn't necessary for creating an image in an [Azure Compute Gallery](shared-image-galleries.md#generalized-and-specialized-images) unless you want to create an image that has no machine specific information, like user accounts. Azure Compute Gallery supports *specialized* images that retain machine specific information. For more information, see [Generalized and specialized images](shared-image-galleries.md#generalized-and-specialized-images).

Generalizing is still required when creating a legacy  image outside of a gallery.

> [!IMPORTANT]
> Once you mark a VM as `generalized` in Azure, you can't restart the VM.

## Deprovisioning a Linux VM before creating an image

Use the Azure Linux Agent (waagent) to remove machine-specific information from a Linux VM to prepare it for imaging. Waagent deletes the Azure VM agent's provisioning data, machine-specific files, and the last provisioned user account, making the VM ready to be captured as a reusable image template.

### Prerequisites for Linux VMs

Before creating a deprovisioning a VM to create an image, we recommend you follow the distro specific instructions for production workloads. Distribution specific instructions for preparing Linux images for Azure are available here:

- [Generic steps](./linux/create-upload-generic.md)
- [CentOS](./linux/create-upload-centos.md)
- [Debian](./linux/debian-create-upload-vhd.md)
- [Flatcar](./linux/flatcar-create-upload-vhd.md)
- [FreeBSD](./linux/freebsd-intro-on-azure.md)
- [Oracle Linux](./linux/oracle-create-upload-vhd.md)
- [OpenBSD](./linux/create-upload-openbsd.md)
- [Red Hat](./linux/redhat-create-upload-vhd.md)
- [SUSE](./linux/suse-create-upload-vhd.md)
- [Ubuntu](./linux/create-upload-ubuntu.md)


### Deprovision a Linux VM

First you deprovision the VM using the Azure Linux Agent (waagent) to delete machine-specific files and data. Use the `waagent` command with the `-deprovision+user` parameter on your source Linux VM. For more information, see the [Azure Linux Agent user guide](./extensions/agent-linux.md). This process can't be reversed.

1. Connect to your Linux VM with an SSH client.

1. In the SSH window, enter the following command:

   ```bash
   sudo waagent -deprovision+user
   ```

   > [!NOTE]
   > Only run this command on a VM that you'll capture as an image. This command doesn't guarantee that the image is cleared of all sensitive information or is suitable for redistribution. The `+user` parameter also removes the last provisioned user account. To keep user account credentials in the VM, use only `-deprovision`.

1. Type **y** to continue. You can add the `-force` parameter to avoid this confirmation step.

1. After the command completes, type **exit** to close the SSH client.  The VM will still be running at this point.

   Deallocate the VM that you deprovisioned with `az vm deallocate` so that it can be generalized.

   ```azurecli-interactive
   az vm deallocate \
      --resource-group myResourceGroup \
      --name myVM
   ```

   Then mark the VM as generalized on the platform. 

   ```azurecli-interactive
   az vm generalize \
      --resource-group myResourceGroup \
      --name myVM
   ```

## Generalizing a Windows VM before creating an image

Sysprep removes all your personal account and security information, and then prepares the machine to be used as an image. For information about Sysprep, see [Sysprep overview](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

> [!IMPORTANT]
> After you have run Sysprep on a VM, that VM is considered *generalized* and can't be restarted. The process of generalizing a VM isn't reversible. If you need to keep the original VM functioning, you should create a snapshot of the OS disk, create a VM from the snapshot, and then generalize that copy of the VM. 
>
> Custom answer files are not supported in the sysprep step, so you can't use the '/unattend:_answerfile_' switch with your sysprep command.  

### Prerequisites for Windows VMs

- Verify the  CD/DVD-ROM is enabled. If it is disabled, the Windows VM will be stuck at out-of-box experience (OOBE). At a command prompt running as an administrator, run the following command:

   ```cmd
   REM Enable CD/DVD-ROM
   reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\cdrom /v start /t REG_DWORD /d 1 /f
   ```

   The Azure platform mounts an ISO file to the DVD-ROM when a Windows VM is created from a generalized image. For this reason, the **DVD-ROM must be enabled in the OS in the generalized image**. If it is disabled, the Windows VM will be stuck at out-of-box experience (OOBE).

- Verify there are no policies applied restricting removable storage access. For example: Computer configuration\Administrative Templates\System\Removable Storage Access\All Removable Storage classes: Deny all access

- Make sure the server roles running on the machine are supported by Sysprep. For more information, see [Sysprep support for server roles](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles) and [Unsupported scenarios](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview#unsupported-scenarios). 
- 
- Disable encryption. Sysprep requires the drives to be fully decrypted. If encryption is enabled on your VM, disable encryption before you run Sysprep.

- If you plan to run Sysprep on a local machine before uploading your virtual hard disk (VHD) to Azure for the first time, make sure you have [prepared your VM](./windows/prepare-for-upload-vhd-image.md) before starting.  

### Generalize a Windows VM

To generalize your Windows VM, follow these steps:

1. Sign in to your Windows VM.

1. Open a Command Prompt window as an administrator.

1. Delete the panther directory (C:\Windows\Panther) if it exists. This directory contains logs from previous Sysprep operations that can cause Sysprep to fail.

   ```cmd
   rd /s /q C:\Windows\Panther
   ```

1. Change to the %windir%\system32\sysprep directory, and then run:

   ```cmd
   sysprep.exe /generalize /shutdown
   ```

1. The VM will shut down when Sysprep is finished generalizing the VM. Do not restart the VM.

   Once Sysprep has finished, set the status of the virtual machine to **Generalized**.
   
   ```azurepowershell-interactive
   Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized
   ```

## Related content

- [Azure Compute Gallery](shared-image-galleries.md)
