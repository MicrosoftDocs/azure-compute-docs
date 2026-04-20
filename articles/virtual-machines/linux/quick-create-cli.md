---
title: 'Quickstart: Use the Azure CLI to create a Linux Virtual Machine'
description: In this quickstart, you learn how to use the Azure CLI to create a Linux virtual machine
author: cynthn
ms.service: azure-virtual-machines
ms.collection: linux
ms.topic: quickstart
ms.date: 03/27/2026
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli, mode-api, innovation-engine, linux-related-content
# Customer intent: "As a cloud administrator, I want to use the command line to create a Linux virtual machine, so that I can efficiently manage resources and automate deployment in my Azure environment."
---

# Quickstart: Create a Linux virtual machine with the Azure CLI on Azure

**Applies to:** :heavy_check_mark: Linux VMs

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://go.microsoft.com/fwlink/?linkid=2285977)

This quickstart shows you how to use the Azure CLI to deploy a Linux virtual machine (VM) in Azure. Use the Azure CLI to create and manage Azure resources from the command line or in scripts.

If you don't have an Azure subscription, [create a free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Launch Azure Cloud Shell

The Azure Cloud Shell is a free interactive shell that you can use to run the steps in this article. It has common Azure tools preinstalled and configured to use with your account. 

To open the Cloud Shell, select **Try it** from the upper right corner of a code block. You can also open Cloud Shell in a separate browser tab by going to [https://shell.azure.com/bash](https://shell.azure.com/bash). Select **Copy** to copy the blocks of code, paste it into the Cloud Shell, and select **Enter** to run it.

If you prefer to install and use the CLI locally, this quickstart requires Azure CLI version 2.0.30 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI]( /cli/azure/install-azure-cli).

## Log in to Azure using the CLI

To run commands in Azure by using the CLI, you need to sign in first. Sign in by using the `az login` command.

## Create a resource group

A resource group is a container for related resources. You must place all resources in a resource group. Use the [az group create](/cli/azure/group) command to create a resource group with the previously defined `$MY_RESOURCE_GROUP_NAME` and `$REGION` parameters.

```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myVMResourceGroup$RANDOM_ID"
export REGION=EastUS
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myVMResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myVMResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create the virtual machine

To create a VM in this resource group, use the `vm create` command. 

The following example creates a VM and adds a user account. The `--generate-ssh-keys` parameter causes the CLI to look for an available SSH key in `~/.ssh`. If it finds one, it uses that key. If not, it generates and stores a key in `~/.ssh`. The `--public-ip-sku Standard` parameter ensures that the machine is accessible through a public IP address. Finally, it deploys the latest `Ubuntu 22.04` image.

Configure all other values by using environment variables.

```bash
export MY_VM_NAME="myVM$RANDOM_ID"
export MY_USERNAME=azureuser
export MY_VM_IMAGE="Canonical:0001-com-ubuntu-minimal-jammy:minimal-22_04-lts-gen2:latest"
az vm create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $MY_VM_NAME \
    --image $MY_VM_IMAGE \
    --admin-username $MY_USERNAME \
    --assign-identity \
    --generate-ssh-keys \
    --public-ip-sku Standard
```

It takes a few minutes to create the VM and supporting resources. The following example output shows the VM create operation was successful.

Results:
<!-- expected_similarity=0.3 -->
```json
{
  "fqdns": "",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-10-4F-70",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.147.208.85",
  "resourceGroup": "myVMResourceGroup",
  "zones": ""
}
```

## Enable Azure AD Authentication for a Linux virtual machine in Azure

The following code example deploys a Linux VM and then installs the extension to enable Azure AD Authentication for a Linux VM. VM extensions are small applications that provide post-deployment configuration and automation tasks on Azure virtual machines.

```bash
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADSSHLoginForLinux \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --vm-name $MY_VM_NAME
```

## Store IP address of VM to use with SSH

Run the following command to store the IP address of the VM as an environment variable:

```bash
export IP_ADDRESS=$(az vm show --show-details --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_VM_NAME --query publicIps --output tsv)
```

## SSH into the VM

<!--## Export the SSH configuration for use with SSH clients that support OpenSSH & SSH into the VM.
Log in to Azure Linux VMs with Azure AD supports exporting the OpenSSH certificate and configuration. That means you can use any SSH clients that support OpenSSH-based certificates to sign in through Azure AD. The following example exports the configuration for all IP addresses assigned to the VM:-->

<!--
```bash
yes | az ssh config --file ~/.ssh/config --name $MY_VM_NAME --resource-group $MY_RESOURCE_GROUP_NAME
```
-->

You can now SSH into the VM by running the output of the following command in your ssh client of choice:

```bash
ssh -o StrictHostKeyChecking=no $MY_USERNAME@$IP_ADDRESS
```

## Related content

* [Learn about virtual machines](../index.yml)
* [Use Cloud-Init to initialize a Linux VM on first boot](tutorial-automate-vm-deployment.md)
* [Create custom VM images](tutorial-custom-images.md)
* [Load Balance VMs](/azure/load-balancer/quickstart-load-balancer-standard-public-cli)
