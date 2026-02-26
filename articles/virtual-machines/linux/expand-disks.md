---
title: Expand Virtual Hard Disks on a Linux VM
description: Learn how to expand virtual hard disks on a Linux VM with the Azure CLI.
author: pagienge
ms.service: azure-disk-storage
ms.collection: linux
ms.topic: how-to
ms.date: 10/28/2024
ms.author: pagienge
ms.custom: references_regions, devx-track-azurecli, linux-related-content
# Customer intent: "As a Linux system administrator, I want to expand virtual hard disks on a Linux VM, so that I can allocate additional storage space without downtime while managing my resources efficiently."
---

# Expand virtual hard disks on a Linux VM

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

This article covers expanding operating system (OS) disks and data disks for a Linux virtual machine (VM). You can [add data disks](add-disk.md) to provide more storage space, and you can also expand an existing data disk. The default virtual hard disk size for the OS is typically 30 GB on a Linux VM in Azure. This article covers expanding either OS disks or data disks.

An OS disk has a maximum capacity of 4,095 GiB. However, many operating systems are partitioned with [master boot record (MBR)](https://wikipedia.org/wiki/Master_boot_record) by default. MBR limits the usable size to 2 TiB. If you need more than 2 TiB, consider attaching data disks for data storage. If you do need to store data on the OS disk and require extra space, convert it to a GUID Partition Table (GPT).

> [!WARNING]
> Always make sure that your filesystem is in a healthy state and your disk partition table type (GPT or MBR) can support the new size. Back up your data before you perform disk expansion operations. For more information, see the [Azure Backup quickstart](/azure/backup/quick-backup-vm-portal).

## <a id="identifyDisk"></a>Identify an Azure data disk object within the operating system ##

When you expand a data disk that has several data disks on the VM, it might be difficult to relate the Azure logical unit numbers (LUNs) to the Linux devices. If the OS disk needs expansion, it's clearly labeled in the Azure portal as the OS disk.

Start by identifying the relationship between disk utilization, mount point, and device, with the ```df``` command.

```bash
df -Th
```

```output
Filesystem                Type      Size  Used Avail Use% Mounted on
/dev/sda1                 xfs        97G  1.8G   95G   2% /
<truncated>
/dev/sdd1                 ext4       32G   30G  727M  98% /opt/db/data
/dev/sde1                 ext4       32G   49M   30G   1% /opt/db/log
```

Here you can see, for example, that the `/opt/db/data` filesystem is nearly full and is located on the `/dev/sdd1` partition. The output of `df` shows the device path whether the disk is mounted by using the device path or the (preferred) UUID in the fstab. Note the Type column, which indicates filesystem format. The format is important later.

Now locate the LUN that correlates to `/dev/sdd` by examining the contents of `/dev/disk/azure/scsi1`. The output of the following `ls` command shows that the device known as `/dev/sdd` within the Linux OS is located at `LUN1` when you look in the Azure portal.

```bash
sudo ls -alF /dev/disk/azure/scsi1/
```

```output
total 0
drwxr-xr-x. 2 root root 140 Sep  9 21:54 ./
drwxr-xr-x. 4 root root  80 Sep  9 21:48 ../
lrwxrwxrwx. 1 root root  12 Sep  9 21:48 lun0 -> ../../../sdc
lrwxrwxrwx. 1 root root  12 Sep  9 21:48 lun1 -> ../../../sdd
lrwxrwxrwx. 1 root root  13 Sep  9 21:48 lun1-part1 -> ../../../sdd1
lrwxrwxrwx. 1 root root  12 Sep  9 21:54 lun2 -> ../../../sde
lrwxrwxrwx. 1 root root  13 Sep  9 21:54 lun2-part1 -> ../../../sde1
```

## Expand an Azure Managed Disk

### Expand without downtime

You can expand your managed disks without deallocating your VM. The host cache setting of your disk doesn't change whether or not you can expand a data disk without deallocating your VM.

This feature has the following limitations.

[!INCLUDE [virtual-machines-disks-expand-without-downtime-restrictions](../includes/virtual-machines-disks-expand-without-downtime-restrictions.md)]

### Expand Azure Managed Disk

Make sure that you have the latest [Azure CLI](/cli/azure/install-az-cli2) installed and are signed in to an Azure account by using [az login](/cli/azure/reference-index#az-login).

This article requires an existing VM in Azure with at least one data disk attached and prepared. If you don't already have a VM that you can use, see [Create and prepare a VM with data disks](tutorial-manage-disks.md#create-and-attach-disks).

In the following samples, replace the placeholder parameter names like *myResourceGroup* and *myVM* with your own values.

> [!IMPORTANT]
> If your disk meets the requirements in [Expand without downtime](#expand-without-downtime), you can skip steps 1 and 3.

Shrinking an existing disk isn't supported and might result in data loss.

After you expand the disks, expand the volume in the OS to take advantage of the larger disk.

1. Operations on virtual hard disks can't be performed with the VM running. Deallocate your VM with [az vm deallocate](/cli/azure/vm#az-vm-deallocate). The following example deallocates the VM named *myVM* in the resource group named *myResourceGroup*:

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

   
    The VM must be deallocated to expand the virtual hard disk. Stopping the VM with `az vm stop` doesn't release the compute resources. To release compute resources, use `az vm deallocate`.

1. View a list of managed disks in a resource group with [az disk list](/cli/azure/disk#az-disk-list). The following example shows a list of managed disks in the resource group named *myResourceGroup*:

    ```azurecli
    az disk list \
        --resource-group myResourceGroup  \
        --query '[*].{Name:name,size:diskSizeGB,Tier:sku.tier}' \
        --output table
    ```

    Expand the required disk with [az disk update](/cli/azure/disk#az-disk-update). The following example expands the managed disk named *myDataDisk* to *200* GB:

    ```azurecli
    az disk update \
        --resource-group myResourceGroup \
        --name myDataDisk \
        --size-gb 200
    ```
    
   When you expand a managed disk, the updated size is rounded up to the nearest managed disk size.

1. Start your VM with [az vm start](/cli/azure/vm#az-vm-start). The following example starts the VM named *myVM* in the resource group named *myResourceGroup*:

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## Expand a disk partition and filesystem

You can use many tools to perform partition resizing. The tools detailed in the remainder of this article are the same ones that certain automated processes use, such as cloud-init. As described here, the `growpart` tool with the `gdisk` package provides universal compatibility with GPT disks because older versions of some tools such as `fdisk` didn't support GPT.

### Detect a changed disk size

If you used the previously mentioned procedure to expand a data disk without downtime, the reported disk size doesn't change until the device is rescanned. Rescanning normally happens only during the boot process. To call this rescan on demand, use the following procedure. When you use the methods in this article, notice that in this example the data disk is currently `/dev/sda` and was resized from 256 GiB to 512 GiB.

1. Identify the currently recognized size on the first line of output from `fdisk -l /dev/sda`:

   ```bash
   sudo fdisk -l /dev/sda
   ```

   ```output
   Disk /dev/sda: 256 GiB, 274877906944 bytes, 536870912 sectors
   Disk model: Virtual Disk
   Units: sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 4096 bytes
   I/O size (minimum/optimal): 4096 bytes / 4096 bytes
   Disklabel type: dos
   Disk identifier: 0x43d10aad

   Device     Boot Start       End   Sectors  Size Id Type
   /dev/sda1        2048 536870878 536868831  256G 83 Linux
   ```

1. Insert a `1` character into the rescan file for this device. Note the reference to `sda` in the example. The disk identifier changes if a different disk device is resized.

   ```bash
   echo 1 | sudo tee /sys/class/block/sda/device/rescan
   ```

1. Verify that the new disk size is now recognized.

   ```bash
   sudo fdisk -l /dev/sda
   ```

   ```output
   Disk /dev/sda: 512 GiB, 549755813888 bytes, 1073741824 sectors
   Disk model: Virtual Disk
   Units: sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 4096 bytes
   I/O size (minimum/optimal): 4096 bytes / 4096 bytes
   Disklabel type: dos
   Disk identifier: 0x43d10aad

   Device     Boot Start       End   Sectors  Size Id Type
   /dev/sda1        2048 536870878 536868831  256G 83 Linux
   ```

The remainder of this article uses the OS disk for the examples of the procedure to increase the size of a volume at the OS level. If the expanded disk is a data disk, use the [previous guidance to identify the data disk device](#identifyDisk). Follow these instructions as a guideline. Substitute the data disk device (for example, `/dev/sda`), partition numbers, volume names, mount points, and filesystem formats, as necessary.

Consider all Linux OS guidance as generic and that it might apply on any distribution, but it generally matches the conventions of the named marketplace publisher. See the Red Hat documentation for the package requirements on any distribution based on Red Hat or that claims Red Hat compatibility.

### Increase the size of the OS disk

The following instructions apply to endorsed Linux distributions.

Before you proceed, make a full backup copy of your VM or, at a minimum, take a snapshot of your OS disk.

# [Ubuntu](#tab/ubuntu)

On Ubuntu 16.x and newer, the root partition of the OS disk and filesystems are automatically expanded to use all free contiguous space on the root disk by cloud-init. A small amount of free space must be available for the resize operation. In this case, the sequence is to:

1. Increase the size of the OS disk as previously described.
1. Restart the VM, and then access the VM by using the **root** user account.
1. Verify that the OS disk now displays an increased filesystem size.

As shown in the following example, the OS disk was resized from the portal to 100 GB. The `/dev/sda1` filesystem mounted on `/` now displays 97 GB.

```bash
df -Th
```

```output
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  314M     0  314M   0% /dev
tmpfs          tmpfs      65M  2.3M   63M   4% /run
/dev/sda1      ext4       97G  1.8G   95G   2% /
tmpfs          tmpfs     324M     0  324M   0% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     324M     0  324M   0% /sys/fs/cgroup
/dev/sda15     vfat      105M  3.6M  101M   4% /boot/efi
/dev/sdb1      ext4       20G   44M   19G   1% /mnt
tmpfs          tmpfs      65M     0   65M   0% /run/user/1000
user@ubuntu:~#
```

# [SUSE](#tab/suse)

To increase the OS disk size in SUSE 12 SP4, SUSE SLES 12 for SAP, SUSE SLES 15, and SUSE SLES 15 for SAP:

1. Follow the procedure previously described to expand the disk in the Azure infrastructure.

1. Access your VM as the **root** user by using the ```sudo``` command after you sign in as another user:

   ```bash
   sudo -i
   ```

1. Use the following command to install the `growpart` package, which is used to resize the partition, if it isn't already present:

   ```bash
   zypper install growpart
   ```

1. Use the `lsblk` command to find the partition mounted on the root of the filesystem (`/`). In this case, partition 4 of device `sda` is mounted on `/`:

   ```bash
   lsblk
   ```

   ```output
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 28.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Resize the required partition by using the `growpart` command and the partition number determined in the preceding step:

   ```bash
   growpart /dev/sda 4
   ```

   ```output
   CHANGED: partition=4 start=3151872 old: size=59762655 end=62914527 new: size=97511391 end=100663263
   ```

1. Run the `lsblk` command again to check whether the partition was increased.

   The following output shows that the `/dev/sda4` partition was resized to 46.5 GB:

   ```bash
   lsblk
   ```

   ```output
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 46.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Identify the type of filesystem on the OS disk by using the `lsblk` command with the `-f` flag:

   ```bash
   lsblk -f
   ```

   ```output
   NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1
   ├─sda2 vfat   EFI   AC67-D22D                            /boot/efi
   ├─sda3 xfs    BOOT  5731a128-db36-4899-b3d2-eb5ae8126188 /boot
   └─sda4 xfs    ROOT  70f83359-c7f2-4409-bba5-37b07534af96 /
   sdb
   └─sdb1 ext4         8c4ca904-cd93-4939-b240-fb45401e2ec6 /mnt/resource
   ```

1. Based on the filesystem type, use the appropriate commands to resize the filesystem.

   For `xfs`, use this command:

   ```bash
   xfs_growfs /
   ```

   Example output:

   ```output
   meta-data=/dev/sda4              isize=512    agcount=4, agsize=1867583 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0 rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=7470331, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3647, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7470331 to 12188923
   ```

   For `ext4`, use this command:

   ```bash
   resize2fs /dev/sda4
   ```

1. Verify the increased filesystem size for `df -Th` by using this command:

   ```bash
   df -Thl
   ```

   Example output:

   ```output
   Filesystem     Type      Size  Used Avail Use% Mounted on
   devtmpfs       devtmpfs  445M  4.0K  445M   1% /dev
   tmpfs          tmpfs     458M     0  458M   0% /dev/shm
   tmpfs          tmpfs     458M   14M  445M   3% /run
   tmpfs          tmpfs     458M     0  458M   0% /sys/fs/cgroup
   /dev/sda4      xfs        47G  2.2G   45G   5% /
   /dev/sda3      xfs      1014M   86M  929M   9% /boot
   /dev/sda2      vfat      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1      ext4      3.9G   16M  3.7G   1% /mnt/resource
   tmpfs          tmpfs      92M     0   92M   0% /run/user/1000
   tmpfs          tmpfs      92M     0   92M   0% /run/user/490
   ```

   In the preceding example, you can see that the filesystem size for the OS disk was increased.

# [Red Hat with LVM](#tab/rhellvm)

1. Follow the procedure previously described to expand the disk in the Azure infrastructure.

1. Access your VM as the **root** user by using the ```sudo``` command after you sign in as another user:

   ```bash
   sudo -i
   ```

1. Use the `lsblk` command to determine which logical volume (LV) is mounted on the root of the filesystem (`/`). In this case, `rootvg-rootlv` is mounted on `/`. If a different filesystem needs resizing, substitute the LV and mount point throughout this section.

   ```bash
   lsblk -f
   ```

   ```output
   NAME                  FSTYPE      LABEL   UUID                                   MOUNTPOINT
   fd0
   sda
   ├─sda1                vfat                C13D-C339                              /boot/efi
   ├─sda2                xfs                 8cc4c23c-fa7b-4a4d-bba8-4108b7ac0135   /boot
   ├─sda3
   └─sda4                LVM2_member         zx0Lio-2YsN-ukmz-BvAY-LCKb-kRU0-ReRBzh
      ├─rootvg-tmplv      xfs                 174c3c3a-9e65-409a-af59-5204a5c00550   /tmp
      ├─rootvg-usrlv      xfs                 a48dbaac-75d4-4cf6-a5e6-dcd3ffed9af1   /usr
      ├─rootvg-optlv      xfs                 85fe8660-9acb-48b8-98aa-bf16f14b9587   /opt
      ├─rootvg-homelv     xfs                 b22432b1-c905-492b-a27f-199c1a6497e7   /home
      ├─rootvg-varlv      xfs                 24ad0b4e-1b6b-45e7-9605-8aca02d20d22   /var
      └─rootvg-rootlv     xfs                 4f3e6f40-61bf-4866-a7ae-5c6a94675193   /
   ```

1. Check whether there's free space in the LVM volume group (VG) that contains the root partition. If free space is available, skip to step 12.

   ```bash
   vgdisplay rootvg
   ```

   ```output
   --- Volume group ---
   VG Name               rootvg
   System ID
   Format                lvm2
   Metadata Areas        1
   Metadata Sequence No  7
   VG Access             read/write
   VG Status             resizable
   MAX LV                0
   Cur LV                6
   Open LV               6
   Max PV                0
   Cur PV                1
   Act PV                1
   VG Size               <63.02 GiB
   PE Size               4.00 MiB
   Total PE              16132
   Alloc PE / Size       6400 / 25.00 GiB
   Free  PE / Size       9732 / <38.02 GiB
   VG UUID               lPUfnV-3aYT-zDJJ-JaPX-L2d7-n8sL-A9AgJb
   ```

   In this example, the line `Free  PE / Size` shows that there's 38.02 GB free in the volume group because the disk was already resized.

1. Install the `cloud-utils-growpart` package to provide the `growpart` command, which is required to increase the size of the OS disk and the `gdisk` handler for GPT disk layouts. This package is preinstalled on most marketplace images.

   ```bash
   dnf install cloud-utils-growpart gdisk
   ```

   In Red Hat versions 7 or earlier, you can use the `yum` command instead of `dnf`.

1. Determine which disk and partition holds the LVM physical volume (PV) or volumes in the volume group named `rootvg` by using the `pvscan` command. Note the size and free space listed between the brackets (`[` and `]`).

   ```bash
   pvscan
   ```

   ```output
   PV /dev/sda4   VG rootvg          lvm2 [<63.02 GiB / <38.02 GiB free]
   ```

1. Verify the size of the partition by using `lsblk`.

   ```bash
   lsblk /dev/sda4
   ```

   ```output
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  63G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Expand the partition that contains this PV by using `growpart`, the device name, and partition number. Doing so expands the specified partition to use all the free contiguous space on the device.

   ```bash
   growpart /dev/sda 4
   ```

   ```output
   CHANGED: partition=4 start=2054144 old: size=132161536 end=134215680 new: size=199272414 end=201326558
   ```

1. Verify that the partition was resized to the expected size by using the `lsblk` command again. Notice that in the example, `sda4` changed from `63G` to `95G`.

   ```bash
   lsblk /dev/sda4
   ```

   ```output
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  95G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Expand the PV to use the rest of the newly expanded partition.

   ```bash
   pvresize /dev/sda4
   ```

   ```output
   Physical volume "/dev/sda4" changed
   1 physical volume(s) resized or updated / 0 physical volume(s) not resized
   ```

1. Verify that the new size of the PV is the expected size when compared to the original `[size / free]` values.

   ```bash
   pvscan
   ```

   ```output
   PV /dev/sda4   VG rootvg          lvm2 [<95.02 GiB / <70.02 GiB free]
   ```

1. Expand the LV by the required amount, which doesn't need to be all the free space in the volume group. In the following example, `/dev/mapper/rootvg-rootlv` is resized from 2 GB to 12 GB (an increase of 10 GB) through the following command. This command also resizes the filesystem on the LV.

   ```bash
   lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   ```

   Example output:

   ```output
   Size of logical volume rootvg/rootlv changed from 2.00 GiB (512 extents) to 12.00 GiB (3072 extents).
   Logical volume rootvg/rootlv successfully resized.
   meta-data=/dev/mapper/rootvg-rootlv isize=512    agcount=4, agsize=131072 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=524288, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 524288 to 3145728
   ```

1. The `lvresize` command automatically calls the appropriate resize command for the filesystem in the LV. Verify whether `/dev/mapper/rootvg-rootlv`, which is mounted on `/`, has an increased filesystem size by using the `df -Th` command:

   ```bash
   df -Th /
   ```
    Example output:

   ```output
   Filesystem                Type  Size  Used Avail Use% Mounted on
   /dev/mapper/rootvg-rootlv xfs    12G   71M   12G   1% /
   ```

To use the same procedure to resize any other logical volume, change the `lv` name in step 12.

# [Red Hat without LVM](#tab/rhelraw)

1. Follow the procedure previously described to expand the disk in the Azure infrastructure.

1. Access your VM as the **root** user by using the ```sudo``` command after you sign in as another user:

   ```bash
   sudo -i
   ```

1. Install the `cloud-utils-growpart` package to provide the `growpart` command, which is required to increase the size of the OS disk and the `gdisk` handler for GPT disk layouts. This package is preinstalled on most marketplace images.

   ```bash
   dnf install cloud-utils-growpart gdisk
   ```

   In Red Hat versions 7 or earlier, you can use the `yum` command instead of `dnf`.

1. Use the `lsblk -f` command to verify the partition and filesystem type that holds the root (`/`) partition.

   ```bash
   lsblk -f
   ```

   ```output
   NAME    FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1  xfs          2a7bb59d-6a71-4841-a3c6-cba23413a5d2 /boot
   ├─sda2  xfs          148be922-e3ec-43b5-8705-69786b522b05 /
   ├─sda14
   └─sda15 vfat         788D-DC65                            /boot/efi
   sdb
   └─sdb1  ext4         923f51ff-acbd-4b91-b01b-c56140920098 /mnt/resource
   ```

1. For verification, start by listing the partition table of the `sda` disk with `gdisk`. In this example, you see a 48.0-GiB disk with partition 2 sized 29.0 GiB. The disk was expanded from 30 GB to 48 GB in the Azure portal.

   ```bash
   gdisk -l /dev/sda
   ```

   ```output
   GPT fdisk (gdisk) version 0.8.10

   Partition table scan:
   MBR: protective
   BSD: not present
   APM: not present
   GPT: present

   Found valid GPT with protective MBR; using GPT.
   Disk /dev/sda: 100663296 sectors, 48.0 GiB
   Logical sector size: 512 bytes
   Disk identifier (GUID): 78CDF84D-9C8E-4B9F-8978-8C496A1BEC83
   Partition table holds up to 128 entries
   First usable sector is 34, last usable sector is 62914526
   Partitions will be aligned on 2048-sector boundaries
   Total free space is 6076 sectors (3.0 MiB)

   Number  Start (sector)    End (sector)  Size       Code  Name
   1         1026048         2050047   500.0 MiB   0700
   2         2050048        62912511   29.0 GiB    0700
   14            2048           10239   4.0 MiB     EF02
   15           10240         1024000   495.0 MiB   EF00  EFI System Partition
   ```

1. Expand the partition for root, in this case `sda2`, by using the `growpart` command. Using this command expands the partition to use all the contiguous space on the disk.

   ```bash
   growpart /dev/sda 2
   ```

   ```output
   CHANGED: partition=2 start=2050048 old: size=60862464 end=62912512 new: size=98613214 end=100663262
   ```

1. Now print the new partition table with `gdisk` again. Notice that partition 2 is now sized 47.0 GiB.

   ```bash
   gdisk -l /dev/sda
   ```

   ```output
   GPT fdisk (gdisk) version 0.8.10

   Partition table scan:
   MBR: protective
   BSD: not present
   APM: not present
   GPT: present

   Found valid GPT with protective MBR; using GPT.
   Disk /dev/sda: 100663296 sectors, 48.0 GiB
   Logical sector size: 512 bytes
   Disk identifier (GUID): 78CDF84D-9C8E-4B9F-8978-8C496A1BEC83
   Partition table holds up to 128 entries
   First usable sector is 34, last usable sector is 100663262
   Partitions will be aligned on 2048-sector boundaries
   Total free space is 4062 sectors (2.0 MiB)

   Number  Start (sector)    End (sector)  Size       Code  Name
      1         1026048         2050047   500.0 MiB   0700
      2         2050048       100663261   47.0 GiB    0700
   14            2048           10239   4.0 MiB     EF02
   15           10240         1024000   495.0 MiB   EF00  EFI System Partition
   ```

1. Expand the filesystem on the partition with `xfs_growfs`, which is appropriate for a standard marketplace-generated Red Hat system:

   ```bash
   xfs_growfs /
   ```

   ```output
   meta-data=/dev/sda2              isize=512    agcount=4, agsize=1901952 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=7607808, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3714, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7607808 to 12326651
   ```

1. Verify that the new size is reflected with the `df` command:

   ```bash
   df -hl
   ```

   ```output
   Filesystem      Size  Used Avail Use% Mounted on
   devtmpfs        452M     0  452M   0% /dev
   tmpfs           464M     0  464M   0% /dev/shm
   tmpfs           464M  6.8M  457M   2% /run
   tmpfs           464M     0  464M   0% /sys/fs/cgroup
   /dev/sda2        48G  2.1G   46G   5% /
   /dev/sda1       494M   65M  430M  13% /boot
   /dev/sda15      495M   12M  484M   3% /boot/efi
   /dev/sdb1       3.9G   16M  3.7G   1% /mnt/resource
   tmpfs            93M     0   93M   0% /run/user/1000
   ```

---

## Expand without downtime classic VM SKU support

If you're using a classic VM SKU, it might not support expanding disks without downtime.

Use the following PowerShell script to determine which VM SKUs it's available with:

```azurepowershell
Connect-AzAccount
$subscriptionId="yourSubID"
$location="desiredRegion"
Set-AzContext -Subscription $subscriptionId
$vmSizes=Get-AzComputeResourceSku -Location $location | where{$_.ResourceType -eq 'virtualMachines'}

foreach($vmSize in $vmSizes){
    foreach($capability in $vmSize.Capabilities)
    {
       if(($capability.Name -eq "EphemeralOSDiskSupported" -and $capability.Value -eq "True") -or ($capability.Name -eq "PremiumIO" -and $capability.Value -eq "True") -or ($capability.Name -eq "HyperVGenerations" -and $capability.Value -match "V2"))
        {
            $vmSize.Name
       }
   }
}
```
