---
title: Quickstart - Create a Windows VM in the Azure portal
description: In this quickstart, you learn how to use the Azure portal to create a Windows virtual machine
author: cynthn
ms.service: azure-virtual-machines
ms.collection: windows
ms.topic: quickstart
ms.date: 03/25/2026
ms.update-cycle: 180-days
ms.author: cynthn
ms.reviewer: williew
ms.custom:
  - mvc
  - mode-ui
  - sfi-image-nochange
  - portal
# Customer intent: As a cloud user, I want to create a Windows virtual machine through the portal, so that I can deploy and test applications in a controlled environment.
---

# Quickstart: Create a Windows virtual machine in the Azure portal

**Applies to:** :heavy_check_mark: Windows VMs 

Azure virtual machines (VMs) can be created through the Azure portal. This method provides a browser-based user interface to create VMs and their associated resources. This quickstart shows you how to use the Azure portal to deploy a virtual machine (VM) in Azure that runs Windows Server 2022 Datacenter. To see your VM in action, you then RDP to the VM and install the IIS web server.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

> [!IMPORTANT]
> The steps outlined in this quickstart are solely for education purposes and aren't intended for deployments to a production environment.

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Create virtual machine

1. Enter *virtual machines* in the search.
1. Under **Services**, select **Virtual machines**.
1. In the **Virtual machines** page, select **Create** and then **Virtual machine**. The **Create a virtual machine** page opens.
1. Under **Instance details**, enter *myVM* for the **Virtual machine name** and choose *Windows Server 2022 Datacenter: Azure Edition - x64 Gen 2* for the **Image**. The page will fill in the other fields with defaults and use the VM name to create default names for the other resources that are needed.

    :::image type="content" source="media/quick-create-portal/instance-details.png" alt-text="Screenshot of the Instance details section where you provide a name for the virtual machine and select its region, image and size." lightbox="media/quick-create-portal/instance-details.png":::

    > [!NOTE]
    > Some users will now see the option to create VMs in multiple zones. To learn more about this new capability, see [Create virtual machines in an availability zone](../create-portal-availability-zone.md).
    > :::image type="content" source="../media/create-portal-availability-zone/preview.png" alt-text="Screenshot showing that you have the option to create virtual machines in multiple availability zones.":::

1. Under **Administrator account**,  provide a username, such as *azureuser* and a password. The password must be at least 12 characters long and meet the [defined complexity requirements](faq.yml#what-are-the-password-requirements-when-creating-a-vm-).

    :::image type="content" source="media/quick-create-portal/administrator-account.png" alt-text="Screenshot of the Administrator account section where you provide the administrator username and password":::

1. Under **Inbound port rules**, choose **Allow selected ports** and then select **RDP (3389)** and **HTTP (80)** from the drop-down.

    :::image type="content" source="media/quick-create-portal/inbound-port-rules.png" alt-text="Screenshot of the inbound port rules section where you select what ports inbound connections are allowed on":::

1. At the top of the page, select the **Management** tab.
1. In the **Auto-shutdown** section, select the **Enable auto-shutdown** option.
1. Select a **Shutdown time** and the **Time zone** to use (UTC is the default). You can also choose to be notified by e-mail before the shutdown.For more information see [Auto-shutdown](/azure/virtual-machines/auto-shutdown-vm).

1. Leave the remaining defaults and then select the **Review + create** button at the bottom of the page.

    :::image type="content" source="media/quick-create-portal/review-create.png" alt-text="Screenshot showing the Review + create button at the bottom of the page.":::

1. After validation runs, select the **Create** button at the bottom of the page.
    :::image type="content" source="media/quick-create-portal/validation.png" alt-text="Screenshot showing that validation has passed. Select the Create button to create the VM." lightbox="media/quick-create-portal/validation.png":::

1. After deployment is complete, select **Go to resource**.

     :::image type="content" source="media/quick-create-portal/next-steps.png" alt-text="Screenshot showing the next step of going to the resource.":::


## Connect to virtual machine

Create a remote desktop connection to the virtual machine. These directions tell you how to connect to your VM from a Windows computer. On a Mac, you need an RDP client such as this [Remote Desktop Client](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) from the Mac App Store.

1. On the overview page for your virtual machine, select **Connect** and then select **Connect** again from the drop-down. 

    :::image type="content" source="media/quick-create-portal/portal-quick-start-9.png" alt-text="Screenshot of the virtual machine overview page showing the location of the connect button.":::

2. In the **Native RDP** page, select the **Check access** button to make sure port 3389 is available.
1. Select **Download RDP file** to download the connection file to your computer. 

7. Open the downloaded RDP file and click **Connect** when prompted.

9. You may receive a certificate warning during the sign-in process. Click **Yes** or **Continue** to create the connection.

## Install web server

To see your VM in action, install the IIS web server. Open a PowerShell prompt on the VM and run the following command:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

When done, close the RDP connection to the VM.


## View the IIS welcome page

In the portal, select the VM and in the overview of the VM, hover over the IP address to show **Copy to clipboard**. Copy the IP address and paste it into a browser tab. The default IIS welcome page will open, and should look like this:

![Screenshot of the IIS default site in a browser](./media/quick-create-powershell/default-iis-website.png)

## Clean up resources

### Delete resources
When no longer needed, you can delete the resource group, virtual machine, and all related resources.

1. On the Overview page for the VM, select the **Resource group** link.
1. At the top of the page for the resource group, select **Delete resource group**. 
1. A page will open warning you that you are about to delete resources. Type the name of the resource group and select **Delete** to finish deleting the resources and the resource group.

## Next steps

In this quickstart, you deployed a simple virtual machine, opened a network port for web traffic, and installed a basic web server. To learn more about Azure virtual machines, continue to the tutorial for Windows VMs.

> [!div class="nextstepaction"]
> [Azure Windows virtual machine tutorials](./tutorial-manage-vm.md)
