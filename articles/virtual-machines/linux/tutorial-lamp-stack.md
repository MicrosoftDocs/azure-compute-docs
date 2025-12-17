---
title: Tutorial - Deploy LAMP and WordPress on a VM
description: In this tutorial, you learn how to install the LAMP stack, and WordPress, on a Linux virtual machine in Azure.
author: cynthn 
ms.collection: linux
ms.service: azure-virtual-machines
ms.devlang: azurecli
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 04/06/2025
ms.author: mattmcinnes
ms.reviewer: cynthn
#Customer intent: As an IT administrator, I want to learn how to install the LAMP stack so that I can quickly prepare a Linux VM to run web applications.
# Customer intent: As an IT administrator, I want to deploy a LAMP stack on a Linux VM, so that I can create a web server environment to host and test applications like WordPress.
---

# Tutorial: Install a LAMP stack on an Azure Linux VM

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321739)

**Applies to:** :heavy_check_mark: Linux VMs

This article walks you through how to deploy an Apache web server, MySQL, and PHP (the LAMP stack) on an Ubuntu VM in Azure. To see the LAMP server in action, you can optionally install and configure a WordPress site. In this tutorial you learn how to:

> [!div class="checklist"]
> * Create an Ubuntu VM
> * Open port 80 for web traffic
> * Install Apache, MySQL, and PHP
> * Verify installation and configuration
> * Install WordPress

This setup is for quick tests or proof of concept. For more on the LAMP stack, including recommendations for a production environment, see the [Ubuntu documentation](https://help.ubuntu.com/community/ApacheMySQLPHP).

This tutorial uses the CLI within the [Azure Cloud Shell](/azure/cloud-shell/overview), which is constantly updated to the latest version. To open the Cloud Shell, select **Try it** from the top of any code block.

If you choose to install and use the CLI locally, this tutorial requires that you're running the Azure CLI version 2.0.30 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI]( /cli/azure/install-azure-cli).

## Create a resource group

Create a resource group with the [az group create](/cli/azure/group) command. An Azure resource group is a logical container into which Azure resources are deployed and managed.

The following example creates a resource group using environment variables and appends a random suffix to ensure uniqueness.

```azurecli-interactive
export REGION="eastus2"
export RANDOM_SUFFIX="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myResourceGroup${RANDOM_SUFFIX}"
az group create --name "${MY_RESOURCE_GROUP_NAME}" --location $REGION
```

Results:

<!-- expected_similarity=0.3 -->

```JSON
{
  "id": "/subscriptions/xxxxx-xxxxx-xxxxx-xxxxx/resourceGroups/myResourceGroupxxxxx",
  "location": "eastus",
  "name": "myResourceGroupxxxxx",
  "properties": {
      "provisioningState": "Succeeded"
  }
}
```

## Create a virtual machine

Create a VM with the [az vm create](/cli/azure/vm) command.

The following example creates a VM using environment variables. It creates a VM named *myVM* and creates SSH keys if they don't already exist in a default key location. To use a specific set of keys, use the `--ssh-key-value` option. The command also sets *azureuser* as an administrator user name. You use this name later to connect to the VM.

```azurecli-interactive
export MY_VM_NAME="myVM${RANDOM_SUFFIX}"
export IMAGE="Ubuntu2204"
export ADMIN_USERNAME="azureuser"
az vm create \
    --resource-group "${MY_RESOURCE_GROUP_NAME}" \
    --name $MY_VM_NAME \
    --image $IMAGE \
    --admin-username $ADMIN_USERNAME \
    --generate-ssh-keys
```

When the VM has been created, the Azure CLI shows information similar to the following example. Take note of the `publicIpAddress`. This address is used to access the VM in later steps.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

## Open port 80 for web traffic

By default, only SSH connections are allowed into Linux VMs deployed in Azure. Because this VM is going to be a web server, you need to open port 80 from the internet. Use the [az vm open-port](/cli/azure/vm) command to open the desired port.

```azurecli-interactive
az vm open-port --port 80 --resource-group "${MY_RESOURCE_GROUP_NAME}" --name $MY_VM_NAME
```

For more information about opening ports to your VM, see [Open ports](nsg-quickstart.md).

## SSH into your VM

If you don't already know the public IP address of your VM, run the [az network public-ip list](/cli/azure/network/public-ip) command. You need this IP address for several later steps.

```azurecli-interactive
export PUBLIC_IP=$(az network public-ip list --resource-group "${MY_RESOURCE_GROUP_NAME}" --query [].ipAddress -o tsv)
```

Use the `ssh` command to create an SSH session with the virtual machine. Substitute the correct public IP address of your virtual machine. 

## Install Apache, MySQL, and PHP

Run the following command to update Ubuntu package sources and install Apache, MySQL, and PHP. Note the caret (^) at the end of the command, which is part of the `lamp-server^` package name.

```bash
ssh -o StrictHostKeyChecking=no azureuser@$PUBLIC_IP "sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt-get -y install lamp-server^"
```

You're prompted to install the packages and other dependencies. This process installs the minimum required PHP extensions needed to use PHP with MySQL.

## Verify Apache

Check the version of Apache with the following command:
```bash
ssh -o StrictHostKeyChecking=no azureuser@$PUBLIC_IP "apache2 -v"
```

With Apache installed, and port 80 open to your VM, the web server can now be accessed from the internet. To view the Apache2 Ubuntu Default Page, open a web browser, and enter the public IP address of the VM. Use the public IP address you used to SSH to the VM:

![Apache default page][3]

## Verify and secure MySQL

Check the version of MySQL with the following command (note the capital `V` parameter):

```bash
ssh -o StrictHostKeyChecking=no azureuser@$PUBLIC_IP "mysql -V"
```

To help secure the installation of MySQL, including setting a root password, you can run the `sudo mysql_secure_installation` command. This command prompts you to answer several questions to help secure your MySQL installation. 

You can optionally set up the Validate Password Plugin (recommended). Then, set a password for the MySQL root user, and configure the remaining security settings for your environment. We recommend that you answer "Y" (yes) to all questions.

If you want to try MySQL features (create a MySQL database, add users, or change configuration settings), login to MySQL. This step isn't required to complete this tutorial. For doing this, you can use the `sudo mysql -u root -p` command in your VM and then enter your root password when prompted. This command connects to your VM via SSH and launches the MySQL command line client as the root user.

When done, exit the mysql prompt by typing `\q`.

## Verify PHP

Check the version of PHP with the following command:

```bash
ssh -o StrictHostKeyChecking=no azureuser@$PUBLIC_IP "php -v"
```

If you want to test further, you can create a quick PHP info page to view in a browser. The following command creates the PHP info page `sudo sh -c 'echo \"<?php phpinfo\(\)\; ?>\" > /var/www/html/info.php`

Now you can check the PHP info page you created. Open a browser and go to `http://yourPublicIPAddress/info.php`. Substitute the public IP address of your VM. It should look similar to this image.

![PHP info page][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../includes/virtual-machines-linux-tutorial-wordpress.md)]

## Next steps

In this tutorial, you deployed a LAMP server in Azure. You learned how to:

> [!div class="checklist"]
> * Create an Ubuntu VM
> * Open port 80 for web traffic
> * Install Apache, MySQL, and PHP
> * Verify installation and configuration
> * Install WordPress on the LAMP server

Advance to the next tutorial to learn how to secure web servers with TLS/SSL certificates.

> [!div class="nextstepaction"]
> [Secure web server with TLS](tutorial-secure-web-server.md)

[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png