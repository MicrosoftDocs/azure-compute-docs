---
title: Quickstart - Create a Linux VM in the Azure portal
description: In this quickstart, you learn how to use the Azure portal to create a Linux virtual machine.
author: ju-shim
ms.service: azure-virtual-machines
ms.collection: linux
ms.topic: quickstart
ms.date: 01/04/2024
ms.author: jushiman
ms.reviewer: jushiman
ms.custom: mvc, mode-ui, linux-related-content
# Customer intent: "As a cloud user, I want to create a Linux virtual machine through a web portal, so that I can quickly deploy and test applications in a cloud environment."
---

# Quickstart: Create a Linux virtual machine in the Azure portal

**Applies to:** :heavy_check_mark: Linux VMs

Azure virtual machines (VMs) can be created through the Azure portal. The Azure portal is a browser-based user interface to create Azure resources. This quickstart shows you how to use the Azure portal to deploy a Linux virtual machine (VM) running Ubuntu Server 22.04 LTS. To see your VM in action, you also SSH to the VM and install the NGINX web server.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Create virtual machine

1. Enter *virtual machines* in the search.
1. Under **Services**, select **Virtual machines**.
1. In the **Virtual machines** page, select **Create** and then **Virtual machine**.  The **Create a virtual machine** page opens.

1. In the **Basics** tab, under **Project details**, make sure the correct subscription is selected and then choose to **Create new** resource group. Enter *myResourceGroup* for the name.*.

	![Screenshot of the Project details section showing where you select the Azure subscription and the resource group for the virtual machine](~/reusable-content/ce-skilling/azure/media/virtual-machines/project-details.png)

1. Under **Instance details**, enter *myVM* for the **Virtual machine name**, and choose *Ubuntu Server 22.04 LTS - Gen2* for your **Image**. Leave the other defaults. The default size and pricing is only shown as an example. Size availability and pricing are dependent on your region and subscription.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/virtual-machines/instance-details.png" alt-text="Screenshot of the Instance details section where you provide a name for the virtual machine and select its region, image, and size.":::

    > [!NOTE]
    > Some users will now see the option to create VMs in multiple zones. To learn more about this new capability, see [Create virtual machines in an availability zone](../create-portal-availability-zone.md).
    > :::image type="content" source="../media/create-portal-availability-zone/preview.png" alt-text="Screenshot showing that you have the option to create virtual machines in multiple availability zones.":::

1. Under **Administrator account**, select **SSH public key**.

1. In **Username** enter *azureuser*.

1. For **SSH public key source**, leave the default of **Generate new key pair**, and then enter *myKey* for the **Key pair name**.

    ![Screenshot of the Administrator account section where you select an authentication type and provide the administrator credentials](~/reusable-content/ce-skilling/azure/media/virtual-machines/administrator-account.png)

1. Under **Inbound port rules** > **Public inbound ports**, choose **Allow selected ports** and then select **SSH (22)** and **HTTP (80)** from the drop-down.

	![Screenshot of the inbound port rules section where you select what ports inbound connections are allowed on](~/reusable-content/ce-skilling/azure/media/virtual-machines/inbound-port-rules.png)

1. Leave the remaining defaults and then select the **Review + create** button at the bottom of the page.

1. On the **Create a virtual machine** page, you can see the details about the VM you are about to create. When you are ready, select **Create**.

1. When the **Generate new key pair** window opens, select **Download private key and create resource**. Your key file will be download as **myKey.pem**. Make sure you know where the `.pem` file was downloaded; you will need the path to it in the next step.

1. When the deployment is finished, select **Go to resource**.

1. On the page for your new VM, select the public IP address and copy it to your clipboard.


	![Screenshot showing how to copy the IP address for the virtual machine](~/reusable-content/ce-skilling/azure/media/virtual-machines/ip-address.png)


## Connect to virtual machine

Create an [SSH connection](/azure/virtual-machines/linux-vm-connect) with the VM.

1. If you are on a Mac or Linux machine, open a Bash prompt and set read-only permission on the .pem file using `chmod 400 ~/Downloads/myKey.pem`. If you are on a Windows machine, open a PowerShell prompt.

1. At your prompt, open an SSH connection to your virtual machine. Replace the IP address with the one from your VM, and replace the path to the `.pem` with the path to where the key file was downloaded.

```console
ssh -i ~/Downloads/myKey.pem azureuser@10.111.12.123
```

> [!TIP]
> The SSH key you created can be used the next time your create a VM in Azure. Just select the **Use a key stored in Azure** for **SSH public key source** the next time you create a VM. You already have the private key on your computer, so you won't need to download anything.

## Install web server

To see your VM in action, install the NGINX web server. From your SSH session, update your package sources and then install the latest NGINX package.

# [Ubuntu](#tab/ubuntu)

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

# [SUSE Linux (SLES)](#tab/SLES)

```bash
sudo zypper --non-interactive update
sudo zypper --non-interactive install nginx
```

# [Red Hat Enterprise Linux (RHEL)](#tab/rhel)

```bash
sudo dnf update
sudo dnf install nginx
```
---
When done, type `exit` to leave the SSH session.


## View the web server in action

Use a web browser of your choice to view the default NGINX welcome page. Type the public IP address of the VM as the web address. The public IP address can be found on the VM overview page or as part of the SSH connection string you used earlier.

![Screenshot showing the NGINX default site in a browser](./media/quick-create-portal/nginx.png)

## Clean up resources

### Delete resources
When no longer needed, you can delete the resource group, virtual machine, and all related resources.

1. On the Overview page for the VM, select the **Resource group** link.
1. At the top of the page for the resource group, select **Delete resource group**.
1. A page will open warning you that you are about to delete resources. Type the name of the resource group and select **Delete** to finish deleting the resources and the resource group.


### Auto-shutdown
If the VM is still needed, Azure provides an Auto-shutdown feature for virtual machines to help manage costs and ensure you are not billed for unused resources.

1. On the **Operations** section for the VM, select the **Auto-shutdown** option.
1. A page will open where you can configure the auto-shutdown time. Select the **On** option to enable and then set a time that works for you.
1. Once you have set the time, select **Save**  at the top to enable your Auto-shutdown configuration.

> [!NOTE]
> Remember to configure the time zone correctly to match your requirements, as (UTC) Coordinated Universal Time is the default setting in the Time zone dropdown.

For more information see [Auto-shutdown](/azure/virtual-machines/auto-shutdown-vm).

## Next steps

In this quickstart, you deployed a virtual machine, created a Network Security Group and rule, and installed a basic web server.

To learn more about Azure virtual machines, continue to the tutorial for Linux VMs.

> [!div class="nextstepaction"]
> [Azure Linux virtual machine tutorials](./tutorial-manage-vm.md)
