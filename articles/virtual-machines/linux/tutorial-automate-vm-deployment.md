---
title: Tutorial - Customize a Linux VM with cloud-init in Azure
description: In this tutorial, you learn how to use cloud-init and Key Vault to customize Linux VMs the first time they boot in Azure
author: ju-shim
ms.service: azure-virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.date: 04/06/2025
ms.author: jushiman
ms.reviewer: mattmcinnes
ms.custom: mvc, devx-track-azurecli, linux-related-content
# Customer intent: "As a system administrator, I want to automate the customization of Linux VMs on their first boot using cloud-init, so that I can ensure consistent and efficient deployment of applications like NGINX and Node.js without manual intervention."
---

# Tutorial - How to use cloud-init to customize a Linux virtual machine in Azure on first boot

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

In a previous tutorial, you learned how to SSH to a virtual machine (VM) and manually install NGINX. To create VMs in a quick and consistent manner, some form of automation is typically desired. A common approach to customize a VM on first boot is to use [cloud-init](https://cloudinit.readthedocs.io). In this tutorial you learn how to:

> [!div class="checklist"]
> * Create a cloud-init config file
> * Create a VM that uses a cloud-init file
> * View a running Node.js app after the VM is created
> * Use Key Vault to securely store certificates
> * Automate secure deployments of NGINX with cloud-init

If you choose to install and use the CLI locally, this tutorial requires that you are running the Azure CLI version 2.0.30 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI]( /cli/azure/install-azure-cli).

## Cloud-init overview

[Cloud-init](https://cloudinit.readthedocs.io) is a widely used approach to customize a Linux VM as it boots for the first time. You can use cloud-init to install packages and write files, or to configure users and security. As cloud-init runs during the initial boot process, there are no additional steps or required agents to apply your configuration.

Cloud-init also works across distributions. For example, you don't use **apt-get install** or **yum install** to install a package. Instead you can define a list of packages to install. Cloud-init automatically uses the native package management tool for the distro you select.

We are working with our partners to get cloud-init included and working in the images that they provide to Azure. For detailed information cloud-init support for each distribution, see [Cloud-init support for VMs in Azure](using-cloud-init.md).

## Create cloud-init config file

To see cloud-init in action, create a VM that installs NGINX and runs a simple 'Hello World' Node.js app. The following cloud-init configuration installs the required packages, creates a Node.js app, then initializes and starts the app.

At your bash prompt or in the Cloud Shell, create a file named *cloud-init.txt* and paste the following configuration. For example, type `sensible-editor cloud-init.txt` to create the file and see a list of available editors. Make sure that the whole cloud-init file is copied correctly, especially the first line:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    defer: true
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    defer: true
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

For more information about cloud-init configuration options, see [cloud-init config examples](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).

## Create virtual machine

Before you can create a VM, create a resource group with [az group create](/cli/azure/group#az-group-create). The following example creates a resource group. In these commands, a random suffix is appended to the resource group and VM names to prevent name collisions during repeated deployments.

```bash
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export RESOURCE_GROUP="myResourceGroupAutomate$RANDOM_SUFFIX"
export REGION="eastus2"
az group create --name $RESOURCE_GROUP --location $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx-xxxxx-xxxxx-xxxxx/resourceGroups/myResourceGroupAutomatexxx",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroupAutomatexxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

Now create a VM with [az vm create](/cli/azure/vm#az-vm-create). Use the `--custom-data` parameter to pass in your cloud-init config file. Provide the full path to the *cloud-init.txt* config if you saved the file outside of your present working directory. The following example creates a VM; note that the VM name is also appended with the random suffix.

```bash
export VM_NAME="myAutomatedVM$RANDOM_SUFFIX"
az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "fqdns": "",
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupAutomatexxx/providers/Microsoft.Compute/virtualMachines/myAutomatedVMxxx",
  "location": "eastus",
  "name": "myAutomatedVMxxx",
  "powerState": "VM running",
  "publicIpAddress": "x.x.x.x",
  "resourceGroup": "myResourceGroupAutomatexxx",
  "zones": ""
}
```

It takes a few minutes for the VM to be created, the packages to install, and the app to start. There are background tasks that continue to run after the Azure CLI returns you to the prompt. It may be another couple of minutes before you can access the app. When the VM has been created, take note of the `publicIpAddress` displayed by the Azure CLI. This address is used to access the Node.js app via a web browser.

To allow web traffic to reach your VM, open port 80 from the Internet with [az vm open-port](/cli/azure/vm#az-vm-open-port):

```bash
az vm open-port --port 80 --resource-group $RESOURCE_GROUP --name $VM_NAME
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "endpoints": [
    {
      "name": "80",
      "protocol": "tcp",
      "publicPort": 80,
      "privatePort": 80
    }
  ],
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupAutomatexxx/providers/Microsoft.Compute/virtualMachines/myAutomatedVMxxx",
  "location": "eastus",
  "name": "myAutomatedVMxxx"
}
```

## Test web app

Now you can open a web browser and enter the public IP address in the address bar. Provide your own public IP address from the VM create process. Your Node.js app is displayed as shown in the following example:

![Screenshot that shows the running NGINX site.](./media/tutorial-automate-vm-deployment/nginx.png)

## Next steps

In this tutorial, you configured VMs on first boot with cloud-init. You learned how to:

> [!div class="checklist"]
> * Create a cloud-init config file
> * Create a VM that uses a cloud-init file
> * View a running Node.js app after the VM is created
> * Use Key Vault to securely store certificates
> * Automate secure deployments of NGINX with cloud-init

Advance to the next tutorial to learn how to create custom VM images.

> [!div class="nextstepaction"]
> [Create custom VM images](./tutorial-custom-images.md)