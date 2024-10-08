---
title: Prepare a Debian Linux VHD 
description: Learn how to create Debian VHD images for virtual machine deployments in Azure.
author: srijang
ms.service: azure-virtual-machines
ms.custom: linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 06/27/2024
ms.author: maries
ms.reviewer: mattmcinnes
---

# Prepare a Debian VHD for Azure

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

## Prerequisites

This section assumes that you've already installed a Debian Linux operating system from an .iso file downloaded from the [Debian website](https://www.debian.org/distrib/) to a virtual hard disk (VHD). Multiple tools exist to create .vhd files. Hyper-V is only one example. For instructions on using Hyper-V, see [Install the Hyper-V role and configure a virtual machine (VM)](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## Installation notes

* For more tips on preparing Linux for Azure, see [General Linux installation notes](create-upload-generic.md#general-linux-installation-notes).
* The newer VHDX format isn't supported in Azure. You can convert the disk to VHD format by using Hyper-V Manager or the `convert-vhd` cmdlet.
* When you install the Linux system, we recommend that you use standard partitions rather than Logical Volume Manager (LVM), which is often the default for many installations. Using partitions avoids LVM name conflicts with cloned VMs, particularly if an OS disk ever needs to be attached to another VM for troubleshooting. [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) or [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid) can also be used on data disks.
* Don't configure a swap partition on the OS disk. The Azure Linux agent can be configured to create a swap file on the temporary resource disk. More information is available in the following steps.
* All VHDs on Azure must have a virtual size aligned to 1 MB. When you convert from a raw disk to VHD, you must ensure that the raw disk size is a multiple of 1 MB before conversion. For more information, see [Linux installation notes](create-upload-generic.md#general-linux-installation-notes).

## Prepare a Debian image for Azure

You can create the base Azure Debian cloud image with the [fully automatic installation (FAI) cloud image builder](https://salsa.debian.org/cloud-team/debian-cloud-images). To prepare an image without FAI, check out the [generic steps article](./create-upload-generic.md).

The following git clone and apt installation commands were pulled from the Debian cloud images repo. Start by cloning the repo and installing dependencies:

```
$ git clone https://salsa.debian.org/cloud-team/debian-cloud-images.git
$ sudo apt install --no-install-recommends ca-certificates debsums dosfstools \
    fai-server fai-setup-storage make python3 python3-libcloud python3-marshmallow \
    python3-pytest python3-yaml qemu-utils udev
$ cd ./debian-cloud-images
```

Optional: Customize the build by adding scripts (for example, shell scripts) to `./config_space/[release]/scripts/AZURE`.

## Script example to customize the image

```
$ mkdir -p ./config_space/[release]/scripts/AZURE
$ cat > ./config_space/scripts/[release]/AZURE/10-custom <<EOF
#!/bin/bash

\$ROOTCMD bash -c "echo test > /usr/local/share/testing"
EOF
$ sudo chmod 755 ./config_space/[release]/scripts/AZURE/10-custom
```

Prefix any commands you want to have customizing the image with `$ROOTCMD`. It's aliased as `chroot $target`.

## Build the Azure Debian image

```
$ make image_[release]_azure_amd64
```

This command outputs a handful of files in the current directory, most notably the `image_[release]_azure_amd64.raw` image file.

Convert the raw image to VHD for Azure:

```
rawdisk="image_[release]_azure_amd64.raw"
vhddisk="image_[release]_azure_amd64.vhd"

MB=$((1024*1024))
size=$(qemu-img info -f raw --output json "$rawdisk" | \
gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

rounded_size=$(((($size+$MB-1)/$MB)*$MB))
rounded_size_adjusted=$(($rounded_size + 512))

echo "Rounded Size Adjusted = $rounded_size_adjusted"

sudo qemu-img resize "$rawdisk" $rounded_size
qemu-img convert -f raw -o subformat=fixed,force_size -O vpc "$rawdisk" "$vhddisk"
```

This process creates a VHD `image_[release]_azure_amd64.vhd` with a rounded size so that it can be copied successfully to an Azure disk.

>[!NOTE]
> Rather than cloning the salsa repository and building images locally, current stable images can be built and downloaded from [FAI](https://fai-project.org/FAIme/cloud/).

After you create a Debian VHD image and before you upload, verify that the following packages listed as ```install```(first character `i` in line) action with ```installed```(second charactor `i` in line) status in the "image_[release]_azure_amd64.info" file:

* hyperv-daemons
* waagent # *(Optional but recommended for password resets and the use of extensions)*
* cloud-init

Now you should have your Debian VHD that ready to upload to be [uploaded to Azure](./disks-upload-vhd-to-managed-disk-cli.md#option-1-upload-a-vhd).

## Related content

You're now ready to use your Debian Linux VHD to create new VMs in Azure. If this is the first time that you're uploading the .vhd file to Azure, see [Create a Linux VM from a custom disk](./upload-vhd.md#option-1-upload-a-vhd).
