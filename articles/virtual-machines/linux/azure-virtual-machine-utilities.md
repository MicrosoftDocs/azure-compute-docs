---
title: Introduction to Azure virtual machine utilities (azure-vm-utils)
description: Learn about the azure-vm-utils package that provides utilities and udev rules for an optimal Linux experience on Azure VMs
author: vamckms
ms.service: azure-virtual-machines
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 09/03/2025
ms.author: vakavuru
---

# Learn about Azure-VM-Utils

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

The [azure-vm-utils](https://github.com/Azure/azure-vm-utils) package provides essential utilities and udev rules to optimize the Linux experience on Azure Virtual Machines. This package consolidates device management tools for Small Computer System Interface (SCSI), Non-Volatile Memory Express (NVMe), Microsoft Azure Network Adapter ([MANA](/azure/virtual-network/accelerated-networking-mana-overview)), and Mellanox devices, making disk identification and management more reliable and consistent across different VM configurations.

## NVMe udev rules

Newer virtual machines (VMs) SKUs on Azure use the NVMe interface for disk management. VMs with NVMe interfaces interpret and present disks differently from VMs that use SCSI interfaces. For details, see [SCSI to NVMe conversion](/azure/virtual-machines/nvme-linux#scsi-vs-nvme). 

NVMe udev rules in this package consolidate critical tools and udev rules to create stable, predictable symlinks for Azure disks. This package provides an easy and reliable way to identify disks, making automation, troubleshooting, and management simpler.

### Symlinks

WALinuxAgent currently includes udev rules to provide several symlinks for SCSI disks:

- `/dev/disk/azure/resource`
- `/dev/disk/azure/root`
- `/dev/disk/azure/scsi0/lun<lun>`
- `/dev/disk/azure/scsi1/lun<lun>`

The rules found in WALinuxAgent are being extended with azure-vm-utils to add identification support for NVMe devices.

The following symlinks are provided for all instances with NVMe disks:

- `/dev/disk/azure/data/by-lun/<lun>`
- `/dev/disk/azure/local/by-serial/<serial>`
- `/dev/disk/azure/os`

For v6 and newer VM sizes with local NVMe disks that support namespace identifiers, the following links are also available:

- `/dev/disk/azure/local/by-index/<index>`
- `/dev/disk/azure/local/by-name/<name>`

### SCSI compatibility

To ensure backward compatibility for disks using SCSI controllers, azure-vm-utils supports the following links:

- `/dev/disk/azure/os`
- `/dev/disk/azure/resource`

> [!NOTE]
> Some VM sizes come with both a NVMe temporary disk in addition to a SCSI temporary disk.

### Linux distribution support

The following distros and versions currently include az-vm-utils in their official Azure marketplace images:

| Distribution | Version |
|--------|---------|
| Fedora | 42 |
| Kinvolk / Flatcar | 4152.2.3 |
| Azure Linux | 2.0 |
| Canonical / Ubuntu | 22.04, 24.04, 25.04 |

### Installation

If the package isn't present in the default platform image, install it via package managers or from the [GitHub repository](https://github.com/Azure/azure-vm-utils).

### Manual installation

For distributions where azure-vm-utils isn't preinstalled, build and install it manually:

```bash
# Clone the repository
git clone https://github.com/Azure/azure-vm-utils.git
cd azure-vm-utils

# Build the package
cmake .
make

# Install (requires root privileges)
sudo make install
```

## Utilities

### azure-nvme-id

The `azure-nvme-id` utility helps identify Azure NVMe devices and their properties. This utility is useful for troubleshooting and scripting.

To run the utility:

```bash
sudo azure-nvme-id
```

To run in udev mode (typically used by udev rules):

```bash
DEVNAME=/dev/nvme0n1 azure-nvme-id --udev
```

## Using the symlinks

After azure-vm-utils is installed, you can use the predictable symlinks for disk operations instead of relying on device names that might change between reboots.

### Examples

List all Azure disk symlinks:

```bash
find /dev/disk/azure/ -type l
```

Access the OS disk:

```bash
ls -la /dev/disk/azure/os
```

Access data disks by LUN:

```bash
ls -la /dev/disk/azure/data/by-lun/
```

Access local NVMe disks by serial number:

```bash
ls -la /dev/disk/azure/local/by-serial/
```

## Verification

To verify that azure-vm-utils is working correctly on your VM:

1. Check if the package is installed:
   
   ```bash
   # For RPM-based systems
   rpm -qa azure-vm-utils
   
   # For DEB-based systems
   dpkg -l azure-vm-utils
   ```

1. Verify udev rules are in place:
   
   ```bash
   ls -al /usr/lib/udev/rules.d/10-azure-unmanaged-sriov.rules
   ls -al /usr/lib/udev/rules.d/80-azure-disk.rules
   ```

1. Check for Azure disk symlinks:
   
   ```bash
   find /dev/disk/azure -type l
   ```