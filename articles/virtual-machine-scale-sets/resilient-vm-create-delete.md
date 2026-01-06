---
title: Resilient create and delete for Virtual Machine Scale Sets (Preview)
description: Learn how to enable retries on failed Virtual Machine (VM) creates and deletes. 
author: manasisoman-work
ms.author: manasisoman
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 01/06/2026
ms.reviewer: cynthn
# Customer intent: "As a cloud infrastructure administrator, I want to enable resilient create and delete for Virtual Machine Scale Sets, so that I can minimize manual intervention when handling errors during VM provisioning and deletion processes."
---


# Resilient create and delete for Virtual Machine Scale Sets

Resilient create and delete for Virtual Machine Scale Sets helps reduce Virtual Machine (VM) create and delete errors by automatically retrying those operations on your behalf. Failed VMs can accumulate and result in unusable capacity, requiring manual effort to detect and clean up. 

## Resilient create

Resilient create runs on Virtual Machines Scale Sets during the initial create of the scale set or during a scale-out. 

Resilient create initiates retries for the following errors:
- OS Provisioning Timeout
- VM Start Timeout 

Resilient create attempts the create operation for up to 30 total minutes. If retries are exhausted, then the VM enters a failed provisioning state.

:::image type="content" source="./media/resilient-vm-create-delete/resilient-create-workflow.png" alt-text="A screenshot showing how Resilient create performs retries on your virtual machines.":::

## Resilient delete

Resilient delete initiates retries for any error that occurs during the delete process. For example, *InternalExecutionError*, *TransientFailure*, or *InternalOperationError*.

If retries remain unsuccessful, the VM enters a failed provisioning state. For example, if you delete a scale set of five VMs and each VM enters a *failed* delete state, the scale set initiates one delete call on itself to delete those five VMs again. If four out of five virtual machines delete on the first retry, then the platform waits a period of 10 minutes before initiating the next delete call for the remaining VM. To check the status of your VMs throughout the delete process, see [Get status for Resilient create or delete](#get-status).

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

## Get status of retries

Get the status of Resilient create and delete for your scale set.

- **Resilient create**: Your VM shows a state of `Creating` while the retries are in progress.
- **Resilient Delete**: During deletion, VMs show a provisioning state of `Deleting`. If a delete attempt fails, the VM temporarily returns to `Failed` before the next retry. This means you may see VMs alternate between `Deleting` and `Failed` states while Resilient delete continues to retry. To check the status of your VM during Resilient delete, retrieve the `ResilientVMDeletionStatus` property, as instructed below.

### [REST](#tab/rest-1)

There are two endpoints are available:

The following endpoint supports Virtual Machine Scale Sets with Uniform orchestration and Flexible orchestration.

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{ResourceName}}/VirtualMachines/{{VMName}}?$expand=resiliencyView&api-version=2024-07-01
```

The following endpoint supports Virtual Machine Scale Sets with Uniform orchestration only.

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{VMSSName}}/virtualMachines?api-version=2024-07-01 
```

The following return values of `ResilientVMDeletionStatus` indicate the progress of Resilient delete.

### [PowerShell](#tab/powershell-1)
Get-AzVmssVM -ResiliencyView -ResourceGroupName <resourceGroupName> -VMScaleSetName <myScaleSetName>

### [CLI](#tab/cli-1)
```azurecli-interactive
az vmss list-instances 
--resiliencyView \
--resource-group <myResourceGroup> \ 
--subscription <mySubscriptionId> \ 
--virtual-machine-scale-set-name <myScaleSet>
```

```azurecli-interactive
az vmss get-resiliency-view
--resource-group <myResourceGroup> \ 
--name <myScaleSet> \
--instance <instance name for scale sets of Flexible Orchestration Mode and instance ID for scale sets of Uniform Orchestration Mode>
```

---
| ResilientVMDeletionStatus | State of delete |
|---------------------------|-----------------|
| Enabled | The `resilientVMDeletion` policy is set on your scale set. |
| Disabled | Your scale set either has the `resilientVMDeletion` policy enabled as false, has a resiliency policy but is missing a `resilientVMDeletion` policy, or doesn't have a resiliency policy. |
| In Progress | The `resilientVMDeletion` policy is enabled and the VM is currently being deleted or is marked for deletion. |
| Failed | The `resilientVMDeletion` policy is enabled and hit the max retry count. |

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
