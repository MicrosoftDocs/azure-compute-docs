---
title: Create a Linux VM in Azure with multiple NICs
description: Learn how to create a Linux VM with multiple NICs attached to it using the Azure CLI or Resource Manager templates.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.custom: devx-track-azurecli, linux-related-content
ms.date: 04/06/2023
ms.author: mattmcinnes
ms.reviewer: cynthn
# Customer intent: "As a cloud engineer, I want to create a Linux VM with multiple network interface cards, so that I can efficiently manage traffic between different subnets and enhance network performance."
---

# How to create a Linux virtual machine in Azure with multiple network interface cards

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

This article details how to create a VM with multiple NICs with the Azure CLI.

## Create supporting resources
Install the latest [Azure CLI](/cli/azure/install-az-cli2) and log in to an Azure account using [az login](/cli/azure/reference-index).

In the following examples, replace example parameter names with your own values. Example parameter names included *myResourceGroup*, *mystorageaccount*, and *myVM*.

First, create a resource group with [az group create](/cli/azure/group). The following example creates a resource group named *myResourceGroup* in the *eastus* location. In these examples, we declare environment variables as they are used and add a random suffix to unique resource names.

```azurecli
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export MY_RESOURCE_GROUP_NAME="myResourceGroup$RANDOM_SUFFIX"
export REGION="WestUS2"
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```
<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxx",
  "location": "WestUS2",
  "managedBy": null,
  "name": "myResourceGroupxxx",
  "properties": {
      "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

Create the virtual network with [az network vnet create](/cli/azure/network/vnet). The following example creates a virtual network named *myVnet* and subnet named *mySubnetFrontEnd*:

```azurecli
export VNET_NAME="myVnet"
export FRONTEND_SUBNET="mySubnetFrontEnd"
az network vnet create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $VNET_NAME \
    --address-prefix 10.0.0.0/16 \
    --subnet-name $FRONTEND_SUBNET \
    --subnet-prefix 10.0.1.0/24
```

Create a subnet for the back-end traffic with [az network vnet subnet create](/cli/azure/network/vnet/subnet). The following example creates a subnet named *mySubnetBackEnd*:

```azurecli
export BACKEND_SUBNET="mySubnetBackEnd"
az network vnet subnet create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --vnet-name $VNET_NAME \
    --name $BACKEND_SUBNET \
    --address-prefix 10.0.2.0/24
```

Create a network security group with [az network nsg create](/cli/azure/network/nsg). The following example creates a network security group named *myNetworkSecurityGroup*:

```azurecli
export NSG_NAME="myNetworkSecurityGroup"
az network nsg create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $NSG_NAME
```

## Create and configure multiple NICs
Create two NICs with [az network nic create](/cli/azure/network/nic). The following example creates two NICs, named *myNic1* and *myNic2*, connected to the network security group, with one NIC connecting to each subnet:

```azurecli
export NIC1="myNic1"
export NIC2="myNic2"
az network nic create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $NIC1 \
    --vnet-name $VNET_NAME \
    --subnet $FRONTEND_SUBNET \
    --network-security-group $NSG_NAME
az network nic create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $NIC2 \
    --vnet-name $VNET_NAME \
    --subnet $BACKEND_SUBNET \
    --network-security-group $NSG_NAME
```

## Create a VM and attach the NICs
When you create the VM, specify the NICs you created with --nics. You also need to take care when you select the VM size. There are limits for the total number of NICs that you can add to a VM. Read more about [Linux VM sizes](../sizes.md).

Create a VM with [az vm create](/cli/azure/vm). The following example creates a VM named *myVM*:

```azurecli
export VM_NAME="myVM"
az vm create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $VM_NAME \
    --image Ubuntu2204 \
    --size Standard_DS3_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics $NIC1 $NIC2
```

Add routing tables to the guest OS by completing the steps in [Configure the guest OS for multiple NICs](#configure-guest-os-for-multiple-nics).

## Add a NIC to a VM
The previous steps created a VM with multiple NICs. You can also add NICs to an existing VM with the Azure CLI. Different [VM sizes](../sizes.md) support a varying number of NICs, so size your VM accordingly. If needed, you can [resize a VM](../resize-vm.md).

Create another NIC with [az network nic create](/cli/azure/network/nic). The following example creates a NIC named *myNic3* connected to the back-end subnet and network security group created in the previous steps:

```azurecli
export NIC3="myNic3"
az network nic create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $NIC3 \
    --vnet-name $VNET_NAME \
    --subnet $BACKEND_SUBNET \
    --network-security-group $NSG_NAME
```

To add a NIC to an existing VM, first deallocate the VM with [az vm deallocate](/cli/azure/vm). The following example deallocates the VM named *myVM*:

```azurecli
az vm deallocate --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME
```

Add the NIC with [az vm nic add](/cli/azure/vm/nic). The following example adds *myNic3* to *myVM*:

```azurecli
az vm nic add \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --vm-name $VM_NAME \
    --nics $NIC3
```

Start the VM with [az vm start](/cli/azure/vm):

```azurecli
az vm start --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME
```

Add routing tables to the guest OS by completing the steps in [Configure the guest OS for multiple NICs](#configure-guest-os-for-multiple-nics).

## Remove a NIC from a VM
To remove a NIC from an existing VM, first deallocate the VM with [az vm deallocate](/cli/azure/vm). The following example deallocates the VM named *myVM*:

```azurecli
az vm deallocate --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME
```

Remove the NIC with [az vm nic remove](/cli/azure/vm/nic). The following example removes *myNic3* from *myVM*:

```azurecli
az vm nic remove \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --vm-name $VM_NAME \
    --nics $NIC3
```

Start the VM with [az vm start](/cli/azure/vm):

```azurecli
az vm start --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME
```

## Create multiple NICs using Resource Manager templates
Azure Resource Manager templates use declarative JSON files to define your environment. You can read an [overview of Azure Resource Manager](/azure/azure-resource-manager/management/overview). Resource Manager templates provide a way to create multiple instances of a resource during deployment, such as creating multiple NICs. You use *copy* to specify the number of instances to create:

```json
"copy": {
    "name": "multiplenics"
    "count": "[parameters('count')]"
}
```

Read more about [creating multiple instances using *copy*](/azure/azure-resource-manager/templates/copy-resources).

You can also use a copyIndex() to then append a number to a resource name, which allows you to create myNic1, myNic2, etc. The following shows an example of appending the index value:

```json
"name": "[concat('myNic', copyIndex())]",
```

You can read a complete example of [creating multiple NICs using Resource Manager templates](/azure/virtual-network/template-samples).

Add routing tables to the guest OS by completing the steps in [Configure the guest OS for multiple NICs](#configure-guest-os-for-multiple-nics).

## Configure guest OS for multiple NICs

The previous steps created a virtual network and subnet, attached NICs, then created a VM. A public IP address and network security group rules that allow SSH traffic were not created. To configure the guest OS for multiple NICs, you need to allow remote connections and run commands locally on the VM.

To allow SSH traffic, create a network security group rule with [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) as follows:

```azurecli
az network nsg rule create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --nsg-name $NSG_NAME \
    --name allow_ssh \
    --priority 101 \
    --destination-port-ranges 22
```

Create a public IP address with [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) and assign it to the first NIC with [az network nic ip-config update](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-update):

```azurecli
export PUBLIC_IP_NAME="myPublicIP"
az network public-ip create --resource-group $MY_RESOURCE_GROUP_NAME --name $PUBLIC_IP_NAME

az network nic ip-config update \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --nic-name $NIC1 \
    --name ipconfig1 \
    --public-ip $PUBLIC_IP_NAME
```

To view the public IP address of the VM, use [az vm show](/cli/azure/vm#az-vm-show) as follows:

```azurecli
az vm show --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME -d --query publicIps -o tsv
```
<!-- expected_similarity=0.3 -->
```TEXT
x.x.x.x
```

Now SSH to the public IP address of your VM. The default username provided in a previous step was *azureuser*. Provide your own username and public IP address:

```bash
export IP_ADDRESS=$(az vm show --resource-group $MY_RESOURCE_GROUP_NAME --name $VM_NAME -d --query publicIps -o tsv)
ssh -o StrictHostKeyChecking=no azureuser@$IP_ADDRESS
```
To send to or from a secondary network interface, you have to manually add persistent routes to the operating system for each secondary network interface. In this article, *eth1* is the secondary interface. Instructions for adding persistent routes to the operating system vary by distro. See documentation for your distro for instructions.

When adding the route to the operating system, the gateway address is the first address of the subnet the network interface is in. For example, if the subnet has been assigned the range 10.0.2.0/24, the gateway you specify for the route is 10.0.2.1 or if the subnet has been assigned the range 10.0.2.128/25, the gateway you specify for the route is 10.0.2.129. You can define a specific network for the route's destination, or specify a destination of 0.0.0.0, if you want all traffic for the interface to go through the specified gateway. The gateway for each subnet is managed by the virtual network.

Once you've added the route for a secondary interface, verify that the route is in your route table with `route -n`. The following example output is for the route table that has the two network interfaces added to the VM in this article:

```output
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 eth1
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
168.63.129.16   10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
169.254.169.254 10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
```

Confirm that the route you added persists across reboots by checking your route table again after a reboot. To test connectivity, you can enter the following command, for example, where *eth1* is the name of a secondary network interface: `ping bing.com -c 4 -I eth1`

## Next steps
Review [Linux VM sizes](../sizes.md) when trying to creating a VM with multiple NICs. Pay attention to the maximum number of NICs each VM size supports.

To further secure your VMs, use just in time VM access. This feature opens network security group rules for SSH traffic when needed, and for a defined period of time. For more information, see [Manage virtual machine access using just in time](/azure/security-center/security-center-just-in-time).