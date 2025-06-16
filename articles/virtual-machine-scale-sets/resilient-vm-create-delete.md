---
title: Resilient create and delete for Virtual Machine Scale Sets (Preview)
description: Learn how to enable retries on failed Virtual Machine (VM) creates and deletes. 
author: manasisoman-work
ms.author: manasisoman
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 04/28/2025
ms.reviewer: ju-shim
---


# Resilient create and delete for Virtual Machine Scale Sets (Preview)

> [!IMPORTANT]
> Resilient virtual machine create and delete for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

Resilient create and delete for Virtual Machine Scale Sets helps reduce Virtual Machine (VM) create and delete errors by automatically retrying failed operations. Failed VMs can accumulate and result in unusable capacity, requiring manual effort to detect and clean up. These errors are rare, but the Resilient create and delete mechanism is built for customers who are deploying or deleting large volumes of Virtual Machine Scale Sets or VMs. 
 
## Prerequisites

Before utilizing Resilient create and delete, complete the feature registration and ensure your API policy is on at least version `2023-07-01`.

### Feature Registration 

Register for the *ResilientScaleSetVMCreation* and *ReliableVMDeletion* feature flags using the `az feature register` command: 

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "ResilientVMScaleSetVMCreation" 
az feature register --namespace "Microsoft.Compute" --name "ReliableVMDeletion" 
```

It takes a few moments for the feature to register. Verify the registration status by using the `az feature show` command:

```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "ResilientVMScaleSetVMCreation"
az feature show --namespace "Microsoft.Compute" --name "ReliableVMDeletion"
```

## Resilient create

Resilient create runs on Virtual Machines Scale Sets during the initial create of the scale set or during a scale-out. 

Resilient create initiates retries for OS Provisioning time-out and VM Start time-out errors. Time-outs are hit when a VM isn't provisioned after 20 minutes for Windows or 8 minutes for Linux.

Resilient create attempts the create operation for up to 30 total minutes. If unsuccessful, the VM remains in a failed state.

:::image type="content" source="./media/resilient-vm-create-delete/resilient-create-workflow.png" alt-text="A screenshot showing how Resilient create performs retries on your virtual machines.":::

## Resilient delete

Resilient delete initiates forced delete retries for any errors that occur during the delete process. For example, *InternalExecutionError*, *TransientFailure*, or *InternalOperationError*.

Resilient delete attempts the forced delete operation five times per VM with an exponential backoff. If unsuccessful, the VM remains in a failed state. For example, if you delete a scale set of five VMs and each VM enters a *failed* delete state, the scale set initiates one delete call on itself to delete those five VMs again. If four out of five virtual machines delete on the first retry, then the platform waits a period of 10 minutes before initiating the next delete call for the remaining VM.

To check the status of your VMs throughout the delete process, see [Get status for Resilient create or delete](#get-status).

:::image type="content" source="./media/resilient-vm-create-delete/resilient-delete-workflow.png" alt-text="Screenshot showing how Resilient delete performs retries on your VMs.":::

## Enable Resilient create and delete

You can enable Resilient create and delete on a new or existing Virtual Machine Scale Set.

### [Portal](#tab/portal-1)

Enable Resilient create and delete on a *new* scale set:

1. In the [Azure portal](https://portal.azure.com) search bar, search for and select **Virtual Machine Scale Sets**.
1. Select **Create** on the **Virtual Machine Scale Sets** page.
1. Go through the steps of [creating your scale set](flexible-virtual-machine-scale-sets-portal.md), by making selection in the **Basics**, **Spot**, **Disks**, **Networking**, and **Management** tabs. 
1. On the **Health** tab, go to the **Recovery** section. 
1. Select checkboxes *Resilient VM create (Preview)* and *Resilient VM delete (Preview)*.
1. Finish creating your Virtual Machine Scale Set. 

:::image type="content" source="./media/resilient-vm-create-delete/enable-on-new-scale-set.png" alt-text="A screenshot showing how to enable Resilient create and delete on a new Virtual Machine Scale Set in the Azure portal.":::

Enable Resilient create and delete on an *existing* scale set:

1. Navigate to your Virtual Machine Scale Set in the [Azure portal](https://portal.azure.com).
1. Under **Capabilities** select **Health and repair**.
1. Under **Recovery**, enable *Resilient VM create (Preview)* and *Resilient VM delete (Preview)*.

:::image type="content" source="./media/resilient-vm-create-delete/enable-on-existing-scale-set.png" alt-text="A screenshot showing how to enable Resilient create and delete on an existing Virtual Machine Scale Set in the Azure portal.":::

### [CLI](#tab/cli-1)

Enable Resilient create and delete on an *existing* scale set:

```azurecli-interactive
az vmss update \ 
--name <myScaleSet> \ 
--resource-group <myResourceGroup> \ 
--enable-resilient-creation true 
```

```azurecli-interactive
az vmss update 
--name <myScaleSet> \ 
--resource-group <myResourceGroup>\ 
--enable-resilient-deletion true 
```

Enable Resilient create and delete on a *new* scale set:

```azurecli-interactive
az vmss create \ 
--name <myScaleSet> \ 
--resource-group <myResourceGroup> \ 
--enable-resilient-creation true 
```

```azurecli-interactive
az vmss create 
--name <myScaleSet> \ 
--resource-group <myResourceGroup>\ 
--enable-resilient-deletion true 
```

### [PowerShell](#tab/powershell-1)

Enable Resilient create and delete after creating a scale set. 

```azurepowershell-interactive
#Create a VM Scale Set profile 
$vmss = new-azvmssconfig -EnableResilientVMCreate -EnableResilientVMDelete 

#Update the VM Scale Set with Resilient create and Resilient delete 
Update-azvmss -ResourceGroupName <resourceGroupName> -VMScaleSetName <scaleSetName> -EnableResilientVMCreate $true -EnableResilientVMDelete $true 
```

### [REST](#tab/rest-1)

Use a `PUT` call for a new scale set and a `PATCH` call for an existing scale set. 

```json
PUT or PATCH https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{yourScaleSetName}?api-version=2023-07-01
```

In the request body, add in the resiliency policies: 

```json
"properties": {  
  "overprovision": false,  
    "resiliencyPolicy": {
        "resilientVMCreationPolicy": {  
            "enabled": true  
        },  
        "resilientVMDeletionPolicy": {  
            "enabled": true  
        }  
    }
}
```
---

## Get status

Get the status of Resilient create and delete for your scale set. 

- **Resilient create**: Your VM status is *Creating* while Resilient create is in progress. 
- **Resilient Delete**: While the delete attempt is in progress, the state of the resource is listed as *Deleting*. If a delete retry fails on a particular VM, then the VM falls back to the *Failed* or *Running* state. However, those states only indicate that a retry of a deletion failed – and Resilient delete might still perform more retries. Therefore, while Resilient delete is going on, you may see the VM alternate states between *Deleting* and *Failed* or *Running*. 

### REST API

To know the status of your VM during Resilient delete, retrieve the return value of the `ResilientVMDeletionStatus` property through REST API. There are two different API endpoints available to get the `ResilientVMDeletionStatus`.

The following endpoint supports Virtual Machine Scale Sets with Uniform orchestration and Flexible orchestration.

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{ResourceName}}/VirtualMachines/{{VMName}}?$expand=resiliencyView&api-version=2024-07-01
```

The following endpoint supports Virtual Machine Scale Sets with Uniform orchestration only.

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{VMSSName}}/virtualMachines?api-version=2024-07-01 
```

The following return values of `ResilientVMDeletionStatus` indicate the progress of Resilient delete.

| ResilientVMDeletionStatus | State of delete |
|---------------------------|-----------------|
| Enabled | The `resilientVMDeletion` policy is set on your scale set. |
| Disabled | Your scale set either has the `resilientVMDeletion` policy enabled as false, has a resiliency policy but is missing a `resilientVMDeletion` policy, or doesn't have a resiliency policy. |
| In Progress | The `resilientVMDeletion` policy is enabled and the VM is currently being deleted or is marked for deletion. |
| Failed | The `resilientVMDeletion` policy is enabled and hit the max retry count. |

---

## FAQ

### What is the minimum API version to use this policy? 
Use API version `2023-07-01`.

### What do I do if my virtual machine is in a 'Failed' state for a long time? 
Resilient delete performs a maximum of five retries on your VM. Therefore, your virtual machine might show up in a 'Failed' state, even when Resilient delete is operating on that VM. For more information, see [Get status for Resilient create or delete](#get-status).

### Does Resilient create work when I attach a new virtual machine to my scale set? 
No, Resilient create operates during a scale-out of a scale set or when you create a new scale set. 

### Is the provisioning of my virtual machine accelerated with Resilient create?
No, Resilient create improves the odds of provisioning the virtual machine, but doesn't improve the speed of the provisioning itself. 

## Next steps
Once your virtual machine is successfully created, learn how to [configure automatic instance repairs](./virtual-machine-scale-sets-automatic-instance-repairs.md) on your Virtual Machine Scale Sets.
