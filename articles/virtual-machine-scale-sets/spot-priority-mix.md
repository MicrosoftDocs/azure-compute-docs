---
title: Get high availability and cost savings with Spot Priority Mix for Virtual Machine Scale Sets
description: Learn how to run a mix of Spot VMs and uninterruptible standard VMs for Virtual Machine Scale Sets to achieve high availability and cost savings.
author: ju-shim
ms.author: jushiman
ms.service: azure-virtual-machine-scale-sets
ms.subservice: azure-spot-vm
ms.topic: concept-article
ms.date: 06/14/2024
ms.reviewer: cynthn
ms.custom: engagement-fy23
# Customer intent: "As a cloud architect, I want to configure a mix of Spot and standard VMs in my Virtual Machine Scale Set, so that I can optimize for high availability while reducing infrastructure costs based on workload demands."
---

# Spot Priority Mix for high availability and cost savings

**Applies to:** :heavy_check_mark: Flexible scale sets

Spot Priority Mix allows you to run a combination of standard Virtual Machines (VMs) and Spot VMs within a single Virtual Machine Scale Set. This feature helps you balance cost savings with availability by letting Azure automatically manage the mix of VM types based on your requirements.

## Overview

With Spot Priority Mix, you can:

- **Save up to 90%** on compute costs by using Spot VMs for interruptible workloads
- **Ensure availability** with standard VMs that aren't evicted
- **Protect against mass evictions** by maintaining a guaranteed number of standard VMs
- **Simplify management** with automatic orchestration of VM creation and deletion

## How it works

Spot Priority Mix uses two key parameters to control your VM distribution:

1. `baseRegularPriorityCount`: The minimum number of standard (non-Spot) VMs that are always maintained
2. `regularPriorityPercentageAboveBase`: The percentage of standard VMs vs. Spot VMs for any capacity beyond the base count

**Example**: With a `baseRegularPriorityCount` of 10 and a `regularPriorityPercentageAboveBase` of 50:
- At 10 total VMs: All 10 are standard VMs (at or below base count)
- At 30 total VMs: 10 base standard VMs + 10 other standard VMs (50% of the 20 above base) + 10 Spot VMs (50% of the 20 above base)

## Prerequisites

Before you use Spot Priority Mix, ensure you have the following:

- A Virtual Machine Scale Set with Flexible orchestration mode
- Understanding of [Azure Spot VMs](../virtual-machines/spot-vms.md) and their eviction behavior
- Appropriate quota for Spot VMs

## Limitations

- Spot Priority Mix requires Flexible orchestration mode
- Not supported with `singlePlacementMode` enabled on the scale set
- Changes to the mix configuration apply only to future scaling operations (existing VMs aren't rebalanced)

## Configure your mix

The platform automatically orchestrates scale-out and scale-in operations to maintain your desired distribution of Spot and standard VMs. You configure the mix using two key settings `baseRegularPriorityCount` and `regularPriorityPercentageAboveBase`.

### Understanding the parameters

| Parameter | Description | Details |
|-----------|-------------|---------|
| `baseRegularPriorityCount` | Sets the minimum number of standard VMs | When total capacity is at or below this number, all VMs are standard. Provides guaranteed capacity that isn't evicted. |
| `regularPriorityPercentageAboveBase` | Defines the standard-to-Spot ratio for VMs beyond the base count | Value between 0-100 (for example, 50 means 50% standard, 50% Spot). Only applies when capacity exceeds the base count. |

### Common configurations

| Use Case | Base Count | Percentage Above Base | Description |
|----------|------------|----------------------|-------------|
| High availability | 20 | 80% | Most VMs are standard, minimal Spot usage |
| Balanced | 10 | 50% | Equal mix above base capacity |
| Cost-optimized | 5 | 20% | Maximum Spot usage with small standard baseline |
| Dev/Test | 0 | 0% | All Spot VMs (no availability guarantee) |

### Eviction policy

When Spot VMs are evicted due to capacity constraints or pricing, the eviction policy determines what happens:

| Policy | Behavior | Cost Impact |
|--------|----------|-------------|
| `Deallocate` (default) | Evicted VMs move to a stopped-deallocated state and can be restarted later | No compute charges, storage costs continue |
| `Delete` | Evicted VMs and their underlying disks are permanently deleted | No charges continue |

### Scale-in behavior

When you scale in, Spot Priority Mix maintains your configured percentage split by intelligently choosing which VMs to remove (Spot or standard) rather than simply removing the oldest or newest VMs. 

### ARM Template

Configure Spot Priority Mix in an ARM template by adding the `priorityMixPolicy` properties to a scale set with Flexible orchestration and Spot priority:

```json
"priorityMixPolicy": {
    "baseRegularPriorityCount": 10,
    "regularPriorityPercentageAboveBase": 50
},
```

This example configuration:
- Maintains a minimum of 10 standard VMs
- For any VMs beyond 10, maintains a 50/50 split between standard and Spot VMs
- At 30 total VMs: 10 base + 10 standard + 10 Spot

### [Portal](#tab/portal)

Configure Spot Priority Mix when you create a Virtual Machine Scale Set in the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for and select **Virtual Machine Scale Sets**.
1. Select **Create**.
1. On the **Basics** tab:
   - Fill out the required fields.
   - Set **Orchestration mode** to **Flexible**.
   - Select **Run with Azure Spot discount**.
1. On the **Spot** tab:
   - Select **Scale with VMs and Spot VMs** under **Scale with VMs and discounted Spot VMs**.
   - Set `baseRegularPriorityCount` by using **Base VM (uninterruptible) count**, the minimum number of standard VMs.
   - Set `regularPriorityPercentageAboveBase` by using **Instance distribution**, the percentage of standard VMs above the base count.
1. Complete the remaining configuration and create your scale set.

### [Azure CLI](#tab/cli)

Create a scale set with Spot Priority Mix using the Azure CLI:

```azurecli
az vmss create -n myScaleSet \
    -g myResourceGroup \
    --orchestration-mode flexible \
    --instance-count 4 \
    --image Ubuntu2204 \
    --priority Spot \
    --regular-priority-count 2 \
    --regular-priority-percentage 50 \
    --eviction-policy Deallocate \
    --single-placement-group False
```

This example creates:
- A scale set with four VMs total
- Two base standard VMs (`--regular-priority-count 2`)
- 50% standard VMs above base (`--regular-priority-percentage 50`)
- Result: Two standard VMs and two Spot VMs

### [Azure PowerShell](#tab/powershell)

Create a scale set with Spot Priority Mix using Azure PowerShell:

```azurepowershell
$vmssConfig = New-AzVmssConfig `
    -Location "East US" `
    -SkuCapacity 4 `
    -SkuName Standard_D2_v5 `
    -OrchestrationMode 'Flexible' `
    -Priority 'Spot' `
    -BaseRegularPriorityCount 2 `
    -RegularPriorityPercentage 50 `
    -EvictionPolicy 'Delete' `
    -PlatformFaultDomainCount 1

New-AzVmss `
    -ResourceGroupName myResourceGroup `
    -Name myScaleSet `
    -VirtualMachineScaleSet $vmssConfig
```

This example creates:
- A scale set with four VMs total
- Two base standard VMs (`-BaseRegularPriorityCount`)
- 50% standard VMs above base (`-RegularPriorityPercentage`)
- Result: Two standard VMs and two Spot VMs

---

## Update your Spot Priority Mix

You can modify the Spot Priority Mix configuration after your scale set is deployed. The updated configuration applies only to future scaling operations. Existing VMs remain unchanged until the scale set scales in or out.

### [Portal](#tab/portal)

Update your existing Spot Priority Mix in the Azure portal:

> [!NOTE]
> In the Azure portal, you can only update Spot Priority Mix for scale sets that already have this feature enabled.

1. Navigate to your Virtual Machine Scale Set in the [Azure portal](https://portal.azure.com).
1. In the left menu, select **Configuration**.
1. Under the Spot Priority Mix section, update:
   - `baseRegularPriorityCount` by using **Base VM (uninterruptible) count**
   - `regularPriorityPercentageAboveBase` by using the **Instance distribution** percentage
1. Select **Save** to apply your changes.

### [Azure CLI](#tab/cli)

Update your Spot Priority Mix using the Azure CLI:

```azurecli
az vmss update \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --regular-priority-count 10 \
    --regular-priority-percentage 80
```

This example updates to:
- 10 base standard VMs
- 80% standard VMs above base (20% Spot VMs)

### [Azure PowerShell](#tab/powershell)

Update your Spot Priority Mix using Azure PowerShell:

```azurepowershell
$vmss = Get-AzVmss `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet"

Update-AzVmss `
    -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -VirtualMachineScaleSet $vmss `
    -BaseRegularPriorityCount 10 `
    -RegularPriorityPercentage 80
```

This example updates to:
- 10 base standard VMs
- 80% standard VMs above base (20% Spot VMs)

---

## Examples

The following examples demonstrate how Spot Priority Mix works in different scenarios. Each example includes a configuration, a table that shows the VM distribution after various operations, and a detailed walk-through.

### Key terminology

- **Total capacity**: The total number of VMs in the Virtual Machine Scale Set.
- **Base standard VMs**: The guaranteed minimum number of standard VMs (set by `baseRegularPriorityCount`).
- **Extra standard VMs**: Standard VMs beyond the base count, calculated by using `regularPriorityPercentageAboveBase`.
- **Spot VMs**: Interruptible VMs that provide cost savings.

### Scenario 1: 50/50 split with 10 base VMs

**Configuration:**
- `baseRegularPriorityCount`: 10
- `regularPriorityPercentageAboveBase`: 50%
- Eviction policy: Delete
- Starting capacity: 10 VMs

**VM distribution during scaling:**

| Action                            | Total capacity | Base standard VMs | Extra standard VMs | Spot VMs |
|-----------------------------------|----------------|-------------------|-------------------|----------|
| Initial creation                  | 10             | 10                | 0                 | 0        |
| Scale out to 20                   | 20             | 10                | 5                 | 5        |
| Scale out to 30                   | 30             | 10                | 10                | 10       |
| Scale out to 40                   | 40             | 10                | 15                | 15       |
| Scale out to 41                   | 41             | 10                | 15                | 16       |
| Scale out to 42                   | 42             | 10                | 16                | 16       |
| All Spot VMs evicted              | 26             | 10                | 16                | 0        |
| Scale out to 30                   | 30             | 10                | 16                | 4        |
| Scale out to 42                   | 42             | 10                | 16                | 16       |
| Scale out to 44                   | 44             | 10                | 17                | 17       |

**Walk-through:**

1. **Initial state (10 VMs)**: All VMs are standard because the total is at the base count.
2. **Scale to 20 VMs**: Added 10 VMs above base, which is 5 standard (50%) and 5 Spot (50%).
3. **Scale to 30 VMs**: Added 20 VMs above base, which is 10 standard (50%) and 10 Spot (50%).
4. **Scale to 41 VMs**: With odd numbers, Spot VMs get the extra VM (16 Spot versus 15 standard).
5. **Scale to 42 VMs**: Balance restored with 16 of each type above the 10 base VMs.
6. **Eviction event**: All 16 Spot VMs are deleted, which leaves 26 total VMs (10 base and 16 extra standard).
7. **Scale to 30 VMs**: Only 4 Spot VMs added to rebalance toward the 50/50 target.
8. **Scale to 42 VMs**: Added 8 more Spot VMs and maintained 16 standard to restore balance.
9. **Scale to 44 VMs**: One of each type added to maintain the 50/50 split.

### Scenario 2: Cost-optimized with 25% standard VMs

**Configuration:**
- `baseRegularPriorityCount`: 10
- `regularPriorityPercentageAboveBase`: 25%
- Eviction policy: Deallocate (VMs stop but aren't deleted)
- Starting capacity: 20 VMs

**VM distribution during scaling:**

| Action                          | Total capacity | Base standard VMs | Extra standard VMs | Spot VMs (running) | Spot VMs (deallocated) |
|---------------------------------|----------------|-------------------|--------------------|-------------------|------------------------|
| Initial creation                | 20             | 10                | 2                  | 8                 | 0                      |
| Scale out to 50                 | 50             | 10                | 10                 | 30                | 0                      |
| Scale out to 110                | 110            | 10                | 25                 | 75                | 0                      |
| 10 Spot VMs evicted             | 110            | 10                | 25                 | 65                | 10                     |
| Scale out to 120                | 120            | 10                | 27                 | 73                | 10                     |

**Walk-through:**

1. **Initial state (20 VMs)**: 10 base standard, 2 extra standard (25% of 10), and 8 Spot (75% of 10).
   - Formula: For the 10 VMs above base, 25% standard (2.5 rounded to 2) and 75% Spot (8).
2. **Scale to 50 VMs**: Added 30 VMs above base. Total of 40 above base equals 10 standard (25%) and 30 Spot (75%).
3. **Scale to 110 VMs**: 100 VMs above base, which is 25 standard (25%) and 75 Spot (75%).
4. **Eviction event**: 10 Spot VMs are deallocated (stopped but not deleted).
   - Total capacity remains 110, but only 65 Spot VMs are running.
   - The deallocated VMs count toward capacity but aren't running.
5. **Scale to 120 VMs**: Added 10 more VMs, which is 2 standard and 8 Spot, to maintain the 25/75 ratio.
   - The 10 deallocated VMs remain deallocated.

## Troubleshooting

If Spot Priority Mix isn't available to you, be sure to configure the `priorityMixPolicy` to specify a *Spot* priority in the `virtualMachineProfile`. Without enabling the `priorityMixPolicy` setting, you won't be able to access this Spot feature.

## Frequently asked questions

### Why aren't my existing VMs changing after I updated the Spot Priority Mix?

Spot Priority Mix configuration applies only to future scaling operations. When you change the percentage split, existing VMs remain unchanged. The new distribution takes effect as the scale set scales in or out. To apply the new configuration, you can manually scale your set or wait for autoscale events.

### What happens when Spot VMs are evicted?

When Azure needs capacity back, Spot VMs are evicted based on your eviction policy.

Your base standard VMs are never evicted, ensuring minimum capacity availability.

### Can I use Spot Priority Mix with Uniform orchestration?

No, Spot Priority Mix is only available with Flexible orchestration mode. Uniform orchestration doesn't support this feature.

### Which Azure regions support Spot Priority Mix?

Spot Priority Mix is available in all Azure regions that support Spot VMs. This availability includes all global Azure regions. For the most current region availability, see [Spot VMs documentation](../virtual-machines/spot-vms.md).

### Does changing the mix trigger immediate VM creation or deletion?

No. Updating your Spot Priority Mix configuration doesn't immediately create or delete VMs. The new settings apply during the next scale operation (scale in, scale out, or replacement of evicted VMs).

### Can I have zero base standard VMs?

Yes, you can set `baseRegularPriorityCount` to 0, which means all VMs follow the percentage distribution. However, this means you could potentially lose all VMs to eviction during high-demand periods. For production workloads, we recommend that you maintain at least some base standard VMs.

## Next steps

> [!div class="nextstepaction"]
> [Learn more about Spot virtual machines](../virtual-machines/spot-vms.md)
