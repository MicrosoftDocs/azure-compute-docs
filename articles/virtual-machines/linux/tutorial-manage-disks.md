---
title: Tutorial - Manage Azure disks with the Azure CLI
description: In this tutorial, you learn how to use the Azure CLI to create and manage Azure disks for virtual machines
author: roygara
ms.author: rogarana
ms.service: azure-disk-storage
ms.topic: tutorial
ms.date: 06/02/2026
ms.custom: mvc, devx-track-azurecli, linux-related-content
ai-usage: ai-assisted
# Customer intent: As an IT administrator, I want to use the Azure CLI to manage VM disks, so that I can efficiently create, attach, and configure storage for Azure virtual machines in a cloud environment.
---

# Tutorial - Manage Azure disks with the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

Azure virtual machines (VMs) use disks to store the operating system, applications, and data. When you create a VM, it is important to choose a disk size and configuration appropriate to the expected workload. This tutorial shows you how to deploy and manage VM disks. You learn about:

> [!div class="checklist"]
> * OS disks and temporary disks
> * Data disks
> * Disk types and performance options
> * Disk performance
> * Attaching and preparing data disks
> * Disk snapshots


## Default Azure disks

When an Azure virtual machine is created, two disks are automatically attached to the virtual machine.

**Operating system disk** - The operating system (OS) disk hosts the VM operating system and can be up to 4,095 gibibytes (GiB), although many operating systems are partitioned with [master boot records (MBRs)](/windows/win32/fileio/basic-and-dynamic-disks#master-boot-record) by default. An MBR limits the usable size to 2 TiB. If you need more than 2 TiB, create and attach [data disks](#data-disks) and use them for data storage. Device names vary by VM generation and disk controller type, so don't rely on a fixed path such as */dev/sda*. OS disk caching is optimized for OS performance, so don't store application data on the OS disk. Use data disks for application and data storage.

**Temporary disk** - Temporary storage uses local storage on the same Azure host as the VM. Depending on the VM size and generation, this storage might appear as a temporary disk or local NVMe disk. It's high-performance, but nonpersistent. If the VM is moved to a new host, data on temporary storage is removed. The available size depends on the VM size.

## data disks

To install applications and store data, additional data disks can be added. Data disks should be used in any situation where durable and responsive data storage is desired. The size of the virtual machine determines how many data disks can be attached to a VM.

## VM disk types

Azure managed disks offer five disk types:

- [Ultra Disks](../disks-types.md#ultra-disks)
- [Premium SSD v2](../disks-types.md#premium-ssd-v2)
- [Premium SSDs](../disks-types.md#premium-ssds)
- [Standard SSDs](../disks-types.md#standard-ssds)
- [Standard HDDs](../disks-types.md#standard-hdds)

In general, use Premium SSD v2 or Premium SSD for production workloads, Standard SSD for dev/test and less performance-sensitive workloads, and Ultra Disks for IO-intensive data workloads. Ultra Disks and Premium SSD v2 are data-disk only options and can't be used as OS disks. For detailed limits, region availability, and sizing guidance, see [Azure managed disk types](../disks-types.md).

## Launch Azure Cloud Shell

Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account.

To open Cloud Shell, select **Try it** from the upper-right corner of a code block. You can also launch Cloud Shell in a separate browser tab at <https://shell.azure.com>. Select **Copy** to copy a code block, paste it into Cloud Shell, and press Enter to run it.

## Create and attach disks

Data disks can be created and attached at VM creation time or to an existing VM.

### Attach disk at VM creation

Create a resource group with the [az group create](/cli/azure/group#az-group-create) command.

```azurecli-interactive
az group create --name myResourceGroupDisk --location eastus
```

Create a VM using the [az vm create](/cli/azure/vm#az-vm-create) command. The following example creates a VM named *myVM*, adds a user account named *azureuser*, and generates SSH keys if they don't already exist. The `--data-disk-sizes-gb` argument specifies additional data disks to create and attach. To create and attach more than one disk, use a space-delimited list of disk sizes. In the following example, a VM is created with two data disks, both 128 GB. Because the disk sizes are 128 GB, these disks are both configured as P10s, which provide maximum 500 IOPS per disk.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupDisk \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_DS2_v2 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 128
```

### Attach disk to existing VM

To create and attach a new disk to an existing virtual machine, use the [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) command. The following example creates a premium disk, 128 gigabytes in size, and attaches it to the VM created in the last step.

```azurecli-interactive
az vm disk attach \
    --resource-group myResourceGroupDisk \
    --vm-name myVM \
    --name myDataDisk \
    --size-gb 128 \
    --sku Premium_LRS \
    --new
```

## Prepare data disks

Once a disk has been attached to the virtual machine, the operating system needs to be configured to use the disk. The following example shows how to manually configure a disk. This process can also be automated using cloud-init, which is covered in a [later tutorial](./tutorial-automate-vm-deployment.md).


Create an SSH connection with the virtual machine. Replace the example IP address with the public IP of the virtual machine.

```console
ssh azureuser@10.101.10.10
```

Partition the disk with `parted`.

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
```

Write a file system to the partition by using the `mkfs` command. Use `partprobe` to make the OS aware of the change.

```bash
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Mount the new disk so that it is accessible in the operating system.

```bash
sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive
```

The disk can now be accessed through the `/datadrive` mountpoint, which can be verified by running the `df -h` command.

```bash
df -h | grep -i "sd"
```

The output shows the new drive mounted on `/datadrive`.

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        29G  2.0G   27G   7% /
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        14G   41M   13G   1% /mnt
/dev/sdc1        50G   52M   47G   1% /datadrive
```

To ensure that the drive is remounted after a reboot, it must be added to the */etc/fstab* file. To do so, get the UUID of the disk with the `blkid` utility.

```bash
sudo -i blkid
```

The output displays the UUID of the drive, `/dev/sdc1` in this case.

```bash
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs"
```

> [!NOTE]
> Improperly editing the **/etc/fstab** file could result in an unbootable system. If unsure, refer to the distribution's documentation for information on how to properly edit this file. It is also recommended that a backup of the /etc/fstab file is created before editing.

Open the `/etc/fstab` file in a text editor as follows:

```bash
sudo nano /etc/fstab
```

Add a line similar to the following to the */etc/fstab* file, replacing the UUID value with your own.

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive  xfs    defaults,nofail   1  2
```

When you are done editing the file, use `Ctrl+O` to write the file and `Ctrl+X` to exit the editor.

Now that the disk has been configured, close the SSH session.

```bash
exit
```

## Take a disk snapshot

When you take a disk snapshot, Azure creates a read-only, point-in-time copy of the disk. Azure VM snapshots are useful to quickly save the state of a VM before you make configuration changes. In the event of an issue or error, you can restore a VM by using a snapshot. When a VM has more than one disk, a snapshot is taken of each disk independently. To take application-consistent backups, consider stopping the VM before you take disk snapshots. Alternatively, use [Azure Backup](/azure/backup/), which supports automated backups while the VM is running.

### Create snapshot

Before you create a snapshot, you need the ID or name of the disk. Use [az vm show](/cli/azure/vm#az-vm-show) to show the disk ID. In this example, the disk ID is stored in a variable so that it can be used in a later step.

```azurecli-interactive
osdiskid=$(az vm show \
   -g myResourceGroupDisk \
   -n myVM \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)
```

Now that you have the ID, use [az snapshot create](/cli/azure/snapshot#az-snapshot-create) to create a snapshot of the disk.

```azurecli-interactive
az snapshot create \
    --resource-group myResourceGroupDisk \
    --source "$osdiskid" \
    --name osDisk-backup
```

### Create disk from snapshot

This snapshot can then be converted into a disk using [az disk create](/cli/azure/disk#az-disk-create), which can be used to recreate the virtual machine.

```azurecli-interactive
az disk create \
   --resource-group myResourceGroupDisk \
   --name mySnapshotDisk \
   --source osDisk-backup
```

### Restore virtual machine from snapshot

To demonstrate virtual machine recovery, delete the existing virtual machine using [az vm delete](/cli/azure/vm#az-vm-delete).

```azurecli-interactive
az vm delete \
--resource-group myResourceGroupDisk \
--name myVM
```

Create a new virtual machine from the snapshot disk.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupDisk \
    --name myVM \
    --attach-os-disk mySnapshotDisk \
    --os-type linux
```

### Reattach data disk

All data disks need to be reattached to the virtual machine.

Find the data disk name using the [az disk list](/cli/azure/disk#az-disk-list) command. This example places the name of the disk in a variable named `datadisk`, which is used in the next step.

```azurecli-interactive
datadisk=$(az disk list \
   -g myResourceGroupDisk \
   --query "[?contains(name,'myVM')].[id]" \
   -o tsv)
```

Use the [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) command to attach the disk.

```azurecli-interactive
az vm disk attach \
   --resource-group myResourceGroupDisk \
   --vm-name myVM \
   --name $datadisk
```

## Next steps

In this tutorial, you learned about VM disks topics such as:

> [!div class="checklist"]
> * OS disks and temporary disks
> * Data disks
> * Disk types and performance options
> * Disk performance
> * Attaching and preparing data disks
> * Disk snapshots

Advance to the next tutorial to learn about automating VM configuration.

> [!div class="nextstepaction"]
> [Automate VM configuration](./tutorial-automate-vm-deployment.md)
