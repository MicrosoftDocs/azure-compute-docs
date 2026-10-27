---
author: cynthnan
ms.service: azure-virtual-machines
ms.custom: linux-related-content
ms.topic: include
ms.date: 09/21/2026
ms.author: wwilliams
# Customer intent: "As a Linux VM administrator, I want to ensure that a disk exists at LUN 0 when adding data disks, so that I can avoid errors and ensure all disks are accessible within the VM."
---
When you add data disks to a Linux VM, you might encounter errors if a disk doesn't exist at LUN 0. If you manually add a disk by using [`az vm disk attach`](/cli/azure/vm/disk#az-vm-disk-attach) with `--new` and specify a LUN (`--lun`) rather than allowing Azure to determine the appropriate LUN, ensure that a disk exists at LUN 0.

Consider the following example showing a snippet of the output from `lsscsi`:

```bash
[5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc 
[5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd 
```

The two data disks exist at LUN 0 and LUN 1 (the first column in the `lsscsi` output details `[host:channel:target:lun]`). Both disks should be accessible from within the VM. If you had manually specified the first disk to be added at LUN 1 and the second disk at LUN 2, you may not see the disks correctly from within your VM.

> [!NOTE]
> The Azure `host` value is 5 in these examples, but this may vary depending on the type of storage you select.
> 
> 

This disk behavior is not an Azure problem, but the way in which the Linux kernel follows the SCSI specifications. When the Linux kernel scans the SCSI bus for attached devices, a device must be found at LUN 0 in order for the system to continue scanning for additional devices. As such:

* Review the output of `lsscsi` after adding a data disk to verify that you have a disk at LUN 0.
* If your disk does not show up correctly within your VM, verify a disk exists at LUN 0.
