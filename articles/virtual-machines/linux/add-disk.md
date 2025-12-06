---
title: Add a data disk to Linux VM using the Azure CLI
description: Learn to add a persistent data disk to your Linux VM with the Azure CLI
author: roygara
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 09/03/2025
ms.author: rogarana
# Customer intent: As a system administrator managing Linux VMs, I want to attach and configure a persistent data disk using the command line, so that I can ensure data retention and improve performance for my applications.
---

# Add a disk to a Linux VM

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

This article shows you how to attach a persistent disk to your virtual machine (VM) to preserve your data, even if your VM is reprovisioned due to maintenance or resizing.

## Attach a new disk to a VM

If you want to add a new, empty data disk to your VM, use the [az vm disk attach](/cli/azure/vm/disk) command with the `--new` parameter. If your VM is in an Availability Zone, the disk is automatically created in the same zone as the VM. For more information, see [Overview of Availability Zones](/azure/reliability/availability-zones-overview). The following example creates a disk named *myDataDisk* that is 50 GB in size:

```azurecli
az vm disk attach \
   -g myResourceGroup \
   --vm-name myVM \
   --name myDataDisk \
   --new \
   --size-gb 50
```

### Lower latency

In select regions, disk attach latency is reduced. In those regions, there's an improvement of up to 15%. This improvement is useful if you have planned or unplanned failovers between VMs, you're scaling your workload, or you're running a high-scale stateful workload such as Azure Kubernetes Service. However, this improvement is limited to the explicit disk attach command, `az vm disk attach`. You won't see the performance improvement if you call a command that might implicitly perform an attach, like `az vm update`. You don't need to take any action other than calling the explicit attach command to see this improvement.

[!INCLUDE [virtual-machines-disks-fast-attach-detach-regions](../includes/virtual-machines-disks-fast-attach-detach-regions.md)]

## Attach an existing disk

To attach an existing disk, find the disk ID and pass the ID to the [az vm disk attach](/cli/azure/vm/disk) command. The following example queries for a disk named *myDataDisk* in *myResourceGroup*, then attaches it to the VM named *myVM*:

```azurecli
diskId=$(az disk show -g myResourceGroup -n myDataDisk --query 'id' -o tsv)

az vm disk attach -g myResourceGroup --vm-name myVM --name $diskId
```

## Identifying disks

Azure Linux VMs use different disk interfaces depending on the VM size and generation:
- VM sizes v6 and newer: Use NVMe interface for improved performance
- VM sizes v5 and older: Use SCSI interface for disk management


For details about SCSI vs NVMe differences, see [SCSI to NVMe conversion](/azure/virtual-machines/nvme-linux#scsi-vs-nvme).

### Connect to the virtual machine

To identify disks associated with your Linux VM, connect to the VM by using SSH. For details, see [How to use SSH with Linux on Azure](/azure/virtual-machines/linux/mac-create-ssh-keys). The following example connects to a VM with the public IP address of 10.123.123.25 with the username azureuser:

```bash
ssh azureuser@10.123.123.25
```

> [!NOTE]
> Before identifying specific disks, determine whether your VM uses SCSI, NVMe, or a combination of both interfaces.

### [Azure-VM-Utils](#tab/azure-vm-utils)

The [azure-vm-utils](azure-virtual-machine-utilities.md) package provides utilities to optimize the Linux experience on Azure VMs, making disk identification more reliable across different VM configurations.

Use the following commands to list disks on the VM:

```bash
# List all disks
sudo azure-disk-list

# List NVMe disks with detailed information
sudo azure-nvme-id
```

The output from `azure-nvme-id` is similar to:
```
/dev/nvme0n1: type=os
/dev/nvme0n2: type=data, lun=0
/dev/nvme1n1: type=local, index=1, name=nvme-50G-1
```

### [lsblk](#tab/lsblk)

The `lsblk` command is a standard Linux utility that can identify both SCSI and NVMe disks. The command options differ slightly based on the disk controller type.

#### SCSI disks

For VMs with SCSI controllers, use the following command:

```bash
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
```

The output is similar to:
```
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0      50G
```

Here, `sdc` is the data disk we added. The last number in the HCTL field (for example, 3:0:0:**0**) is the LUN number that corresponds to what you see in the Azure portal.

You can also check the Azure disk symlinks:

```bash
ls -l /dev/disk/azure/scsi1
```

Output:
```
lrwxrwxrwx 1 root root 12 Mar 28 19:41 lun0 -> ../../../sdc
```

#### NVMe disks

For VMs with NVMe controllers, use the following command:

```bash
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT | grep nvme
```

The output is similar to:
```
nvme0n1     disk    30G
├─nvme0n1p1 part   29.9G /
├─nvme0n1p14 part   4M
└─nvme0n1p15 part  106M  /boot/efi
nvme1n1     disk   50G
nvme0n2     disk   50G
```

For more detailed information about NVMe devices, use the `nvme` command:

```bash
sudo nvme list
```

---

## Next Steps

- Format and mount the disks based on your requirements and use case. Review instructions for formatting and mounting [managed disks](disks-format-mount-data-disks-linux.md) and [temporary disks](disks-format-mount-temp-disks-linux.md).
- [Learn about Azure-VM-Utils](azure-virtual-machine-utilities.md).