---
title: Tutorial - Autoscale a scale set with the Azure CLI
description: Learn how to use the Azure CLI to automatically scale a Virtual Machine Scale Set as CPU demands increases and decreases
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: azure-virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 04/06/2025
ms.reviewer: mimckitt
ms.custom: avverma, devx-track-azurecli, linux-related-content
# Customer intent: "As a cloud engineer, I want to implement autoscaling for a Virtual Machine Scale Set using command-line interfaces, so that I can efficiently manage resources in response to changing application demands while optimizing cost and performance."
---

# Tutorial: Automatically scale a Virtual Machine Scale Set with the Azure CLI

When you create a scale set, you define the number of VM instances that you wish to run. As your application demand changes, you can automatically increase or decrease the number of VM instances. The ability to autoscale lets you keep up with customer demand or respond to application performance changes throughout the lifecycle of your app. In this tutorial you learn how to:

> [!div class="checklist"]
> * Use autoscale with a scale set
> * Create and use autoscale rules
> * Simulate CPU load to trigger autoscale rules
> * Monitor autoscale actions as demand changes

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This tutorial requires version 2.0.32 or later of the Azure CLI. If using Azure Cloud Shell, the latest version is already installed.

## Create a scale set
Create a resource group with [az group create](/cli/azure/group).

```azurecli-interactive
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export REGION="WestUS2"
export MY_RESOURCE_GROUP_NAME="myResourceGroup$RANDOM_SUFFIX"
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

Now create a Virtual Machine Scale Set with [az vmss create](/cli/azure/vmss). The following example creates a scale set with an instance count of 2, generates SSH keys if they don't exist, and uses a valid image *Ubuntu2204*.

```azurecli-interactive
export MY_SCALE_SET_NAME="myScaleSet$RANDOM_SUFFIX"
az vmss create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name $MY_SCALE_SET_NAME \
  --image Ubuntu2204 \
  --orchestration-mode Flexible \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys
```

## Define an autoscale profile
To enable autoscale on a scale set, you first define an autoscale profile. This profile defines the default, minimum, and maximum scale set capacity. These limits let you control cost by not continually creating VM instances, and balance acceptable performance with a minimum number of instances that remain in a scale-in event. Create an autoscale profile with [az monitor autoscale create](/cli/azure/monitor/autoscale#az-monitor-autoscale-create). The following example sets the default and minimum capacity of 2 VM instances, and a maximum of 10:

```azurecli-interactive
az monitor autoscale create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --resource $MY_SCALE_SET_NAME \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

## Create a rule to autoscale out
If your application demand increases, the load on the VM instances in your scale set increases. If this increased load is consistent, rather than just a brief demand, you can configure autoscale rules to increase the number of VM instances. When these instances are created and your application is deployed, the scale set starts to distribute traffic to them through the load balancer. You control which metrics to monitor, how long the load must meet a given threshold, and how many VM instances to add.

Create a rule with [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) that increases the number of VM instances when the average CPU load is greater than 70% over a 5-minute period. When the rule triggers, the number of VM instances is increased by three.

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 3
```

## Create a rule to autoscale in
When application demand decreases, the load on the VM instances drops. If this decreased load persists over a period of time, you can configure autoscale rules to decrease the number of VM instances in the scale set. This scale-in action helps reduce costs by running only the necessary number of instances required to meet current demand.

Create another rule with [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) that decreases the number of VM instances when the average CPU load drops below 30% over a 5-minute period. The following example scales in the number of VM instances by one.

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

## Simulate CPU load on scale set
To test the autoscale rules, you need to simulate sustained CPU load on the VM instances in the scale set. In this minimalist approach, we avoid installing additional packages by using the built-in `yes` command to generate CPU load. The following command starts 3 background processes that continuously output data to `/dev/null` for 60 seconds and then terminates them.

```bash
for i in {1..3}; do
  yes > /dev/null &
done
sleep 60
pkill yes
```

This command simulates CPU load without introducing package installation errors.

## Monitor the active autoscale rules
To monitor the number of VM instances in your scale set, use the `watch` command. It may take up to 5 minutes for the autoscale rules to begin the scale-out process in response to the CPU load. However, once it happens, you can exit watch with *CTRL + C* keys. 

By then, the scale set will automatically increase the number of VM instances to meet the demand. The following command shows the list of VM instances in the scale set:

```azurecli-interactive
az vmss list-instances \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name $MY_SCALE_SET_NAME \
  --output table
```

Once the CPU threshold has been met, the autoscale rules increase the number of VM instances in the scale set. The output will show the list of VM instances as new ones are created.

```output
  InstanceId  LatestModelApplied    Location    Name              ProvisioningState    ResourceGroup         VmId
------------  --------------------  ----------  ---------------   -------------------  --------------------  ------------------------------------
           1  True                  WestUS2     myScaleSet_1      Succeeded            myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
           2  True                  WestUS2     myScaleSet_2      Succeeded            myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
           4  True                  WestUS2     myScaleSet_4      Creating             myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
           5  True                  WestUS2     myScaleSet_5      Creating             myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
           6  True                  WestUS2     myScaleSet_6      Creating             myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Once the CPU load subsides, the average CPU load returns to normal. After another 5 minutes, the autoscale rules then scale in the number of VM instances. Scale-in actions remove VM instances with the highest IDs first. When a scale set uses Availability Sets or Availability Zones, scale-in actions are evenly distributed across the VM instances. The following sample output shows one VM instance being deleted as the scale set autoscales in:

```output
6  True                  WestUS2     myScaleSet_6  Deleting             myResourceGroupxxxxx  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Clean up resources
To remove your scale set and associated resources, please manually delete the resource group using your preferred method. 

## Next steps
In this tutorial, you learned how to automatically scale in or out a scale set with the Azure CLI:

> [!div class="checklist"]
> * Use autoscale with a scale set
> * Create and use autoscale rules
> * Simulate CPU load to trigger autoscale rules
> * Monitor autoscale actions as demand changes