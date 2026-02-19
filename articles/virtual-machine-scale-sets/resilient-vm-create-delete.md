---
title: Resilient create and delete for Virtual Machine Scale Sets
description: Learn how to enable retries on failed Virtual Machine (VM) creates and deletes. 
author: manasisoman-work
ms.author: manasisoman
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 01/06/2026
ms.reviewer: cynthn
ms.custom: portal
# Customer intent: "As a cloud infrastructure administrator, I want to enable resilient create and delete for Virtual Machine Scale Sets, so that I can minimize manual intervention when handling errors during VM provisioning and deletion processes."
---


# Resilient create and delete for Virtual Machine Scale Sets

Resilient create and delete enables your Virtual Machine Scale Sets to automatically recover from VM provisioning and deletion failures without manual intervention. The feature retries failed Virtual Machine (VM) create and delete operations on your behalf. Resilient create increases the overall success rate of VM provisioning by retrying through provisioning timeout errors. Resilient delete ensures VMs are properly cleaned up by automatically retrying deletions that fail due to transient platform issues.

## Resilient create

Resilient create automatically retries VM provisioning timeout failures during scale set creation or scale-out operations.

It exclusively addresses:
- `OSProvisioningTimedOut`
- `VMStartTimedOut` 

When enabled, Resilient create automatically retries provisioning until the operation succeeds or reaches the maximum retry duration. VMs that can't be provisioned after all retry attempts remain in a failed state for investigation.

_Note: This feature deletes the failed VM and launches a new one, so the original VM name and ID changes after each retry._

## Resilient delete

Resilient delete automatically retries VM deletions that fail during scale set deletion or scale-in operations. It addresses all transient platform errors, such as `InternalExecutionError`, `TransientFailure`, or `InternalOperationError`. This automated cleanup ensures that VMs are properly removed even when temporary issues occur during the delete operation. To check the status of your VMs throughout the retries, see [Get status for Resilient create or delete](#get-status-of-retries).

## Enable Resilient create and delete

You can enable Resilient create and delete on a new or existing Virtual Machine Scale Set.

### [Portal](#tab/portal-1)

Enable Resilient create and delete on a *new* scale set on Azure portal:

1. In the [Azure portal](https://portal.azure.com) search bar, search for and select **Virtual Machine Scale Sets**.
1. Select **Create** on the **Virtual Machine Scale Sets** page.
1. Go through the steps of [creating your scale set](flexible-virtual-machine-scale-sets-portal.md), by making selection in the **Basics**, **Spot**, **Disks**, **Networking**, and **Management** tabs. 
1. On the **Health** tab, go to the **Recovery** section. 
1. Select checkboxes *Resilient VM create* and *Resilient VM delete*.
1. Finish creating your Virtual Machine Scale Set. 

Enable Resilient create and delete on an *existing* scale set on Azure portal:

1. Navigate to your Virtual Machine Scale Set in the [Azure portal](https://portal.azure.com).
1. Under **Capabilities** select **Health and repair**.
1. Under **Recovery**, enable *Resilient VM create* and *Resilient VM delete*.

### [CLI](#tab/cli-1)

Enable Resilient create and delete on an *existing* scale set using the Azure CLI:

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

Enable Resilient create and delete on a *new* scale set using the Azure CLI:

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

Enable Resilient create and delete after creating a scale set using PowerShell. 

```azurepowershell-interactive
#Create a VM Scale Set profile 
$vmss = new-azvmssconfig -EnableResilientVMCreate -EnableResilientVMDelete 

#Update the VM Scale Set with Resilient create and Resilient delete 
Update-azvmss -ResourceGroupName <resourceGroupName> -VMScaleSetName <scaleSetName> -EnableResilientVMCreate $true -EnableResilientVMDelete $true 
```

### [REST](#tab/rest-1)

Use a `PUT` call for a new scale set and a `PATCH` call for an existing scale set through REST API. 

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

### Prerequisites
Resilient create and delete is supported for Compute API version 2024-07-01 or higher.

## Get status of retries
Because Resilient create and delete operations run automatically in the background, monitoring their progress helps you understand whether retries are still in progress or if a VM requires manual attention.

### Resilient create
Your VM shows a state of `Creating` while the retries are in progress. In rare circumstances, if all retry attempts are exhausted without success, the VM moves to a `Failed` state.

### Resilient delete
During retries, VMs alternate between a provisioning state of `Deleting` and `Failed` as each attempt is made. To determine whether Resilient delete is still actively retrying or has exhausted all attempts, retrieve the `ResilientVMDeletionStatus` property for your VM.

The following return values of `ResilientVMDeletionStatus` indicate the progress of Resilient delete.

| ResilientVMDeletionStatus | State of delete |
|---------------------------|-----------------|
| In Progress | Resilient delete is actively attempting to delete the VM. |
| Failed | Resilient delete reached the maximum retry count without successfully deleting the VM. |
| Enabled | Resilient VM deletion policy is enabled on your scale set. |
| Disabled | Resilient VM deletion policy isn't enabled on your scale set. |

#### [CLI](#tab/cli-2)
Use the Azure CLI to check the resiliency status of your VMs by retrieving the value returned by the `ResilientVMDeletionStatus` property. You can view the status for all VMs in a scale set or query a specific VM instance.

**View resiliency status for all VMs in a scale set:**
```azurecli-interactive
az vmss list-instances 
--resiliencyView \
--resource-group <myResourceGroup> \ 
--subscription <mySubscriptionId> \ 
--virtual-machine-scale-set-name <myScaleSet>
```

**View resiliency status for a specific VM:**
```azurecli-interactive
az vmss get-resiliency-view
--resource-group <myResourceGroup> \ 
--name <myScaleSet> \
--instance <instance-name-or-id>
```
_Note: Use instance name for Flexible orchestration mode and instance ID for Uniform orchestration mode._

#### [PowerShell](#tab/powershell-2)
Use PowerShell to check the resiliency status of your VMs by retrieving the value returned by the `ResilientVMDeletionStatus` property. 

```azurepowershell-interactive
Get-AzVmssVM -ResiliencyView -ResourceGroupName <resourceGroupName> -VMScaleSetName <myScaleSetName> -InstanceId <instance-name-or-id>
```
_Note: Use instance name for Flexible orchestration mode and instance ID for Uniform orchestration mode._

#### [REST](#tab/rest-2)

Use the REST API to check the resiliency status of your VMs by retrieving the value returned by the `ResilientVMDeletionStatus` property. 

**For both Uniform and Flexible orchestration, which queries for a specific VM:**

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{ResourceName}}/VirtualMachines/{{VMName}}?$expand=resiliencyView&api-version=2024-07-01
```

**For Uniform orchestration only, which queries for all VMs in a scale set:**

```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{VMSSName}}/virtualMachines?api-version=2024-07-01 
```

---

## FAQ

### Can I disable Resilient create or delete after enabling it?
Yes, you can disable Resilient create or delete at any time by updating the resiliency policy on your scale set. However, any in-progress retries are completed before the policy change takes effect.

### Can I configure the retry count or timeout values?
No, the retry behavior is predefined and can't be customized.

### Why is my VM stuck in 'Creating' state for a long time?
Resilient create can take up to 30 minutes to complete all retry attempts. If your VM remains in 'Creating' state, Resilient create is still attempting to provision it. After 30 minutes, if unsuccessful, the VM will move to a 'Failed' state.

### Why does my VM show a 'Failed' state even though Resilient delete is enabled?
When a delete attempt fails, the VM temporarily returns to a 'Failed' state before the next retry begins. This behavior is expected. Resilient delete makes up to five retry attempts, so you'll see the VM alternate between 'Deleting' and 'Failed' states during this process. To check if Resilient delete is still actively retrying, see [Get status for Resilient create or delete](#get-status-of-retries).

### Does Resilient create work when I attach a new virtual machine to my scale set? 
No, Resilient create operates during a scale-out of a scale set or when you create a new scale set. 

### Is the provisioning of my virtual machine accelerated with Resilient create?
No, Resilient create improves the odds of provisioning the virtual machine, but doesn't improve the speed of the provisioning itself. 

## Next steps
Once your virtual machine is successfully created, learn how to [configure automatic instance repairs](./virtual-machine-scale-sets-automatic-instance-repairs.md) on your Virtual Machine Scale Sets.
