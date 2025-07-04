---
title: 'Make the D: drive of a VM a data disk '
description: 'Describes how to change drive letters for a Windows VM so that you can use the D: drive as a data drive.'
services: virtual-machines
author: ju-shim
ms.service: azure-virtual-machines
ms.collection: windows
ms.subservice: disks
ms.topic: how-to
ms.date: 01/02/2018
ms.author: jushiman
# Customer intent: "As a system administrator managing a Windows VM, I want to configure the D: drive as a data disk, so that I can ensure my applications have a consistent storage location for their data while avoiding the risks associated with using the temporary disk."
---
# Use the D: drive as a data drive on a Windows VM

**Applies to:** :heavy_check_mark: Windows VMs 


If your application needs to use the D drive to store data, follow these instructions to use a different drive letter for the temporary disk. Never use the temporary disk to store data that you need to keep.

If you resize or **Stop (Deallocate)** a virtual machine, this may trigger placement of the virtual machine to a new hypervisor. A planned or unplanned maintenance event may also trigger this placement. In this scenario, the temporary disk will be reassigned to the first available drive letter. If you have an application that specifically requires the D: drive, you need to follow these steps to temporarily move the pagefile.sys, attach a new data disk and assign it the letter D and then move the pagefile.sys back to the temporary drive. Once complete, Azure will not take back the D: if the VM moves to a different hypervisor.

For more information about how Azure uses the temporary disk, see [Understanding the temporary drive on Microsoft Azure Virtual Machines](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines)

## Attach the data disk
First, you'll need to attach the data disk to the virtual machine. To do this using the portal, see [How to attach a managed data disk in the Azure portal](attach-managed-disk-portal.yml).

## Temporarily move pagefile.sys to C drive
1. Connect to the virtual machine. 
2. Right-click the **Start** menu and select **System**.
3. In the left-hand menu, search for and select **View advanced system settings**.
4. In the **Performance** section, select **Settings**.
5. Select the **Advanced** tab.
6. In the **Virtual memory** section, select **Change**.
7. Select the **C** drive and then click **System managed size** and then click **Set**.
8. Select the **D** drive and then click **No paging file** and then click **Set**.
9. Click Apply. You will get a warning that the computer needs to be restarted for the changes to take affect.
10. Restart the virtual machine.

## Change the drive letters
1. Once the VM restarts, log back on to the VM.
2. Click the **Start** menu and type **diskmgmt.msc** and hit Enter. Disk Management will start.
3. Right-click on **D**, the Temporary Storage drive, and select **Change Drive Letter and Paths**.
4. Under Drive letter, select a new drive such as **T** and then click **OK**. 
5. Right-click on the data disk, and select **Change Drive Letter and Paths**.
6. Under Drive letter, select drive **D** and then click **OK**. 

## Move pagefile.sys back to the temporary storage drive
1. Right-click the **Start** menu and select **System**
2. In the left-hand menu, search for and select **View advanced system settings**.
3. In the **Performance** section, select **Settings**.
4. Select the **Advanced** tab.
5. In the **Virtual memory** section, select **Change**.
6. Select the OS drive **C** and click **No paging file** and then click **Set**.
7. Select the temporary storage drive **T** and then click **System managed size** and then click **Set**.
8. Click **Apply**. You will get a warning that the computer needs to be restarted for the changes to take affect.
9. Restart the virtual machine.

## Next steps
* You can increase the storage available to your virtual machine by [attaching an additional data disk](attach-managed-disk-portal.yml).
