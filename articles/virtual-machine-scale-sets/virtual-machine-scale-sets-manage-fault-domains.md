---
title: Manage fault domains in Azure Virtual Machine Scale Sets
description: Learn how to choose the right number of fault domains while creating a Virtual Machine Scale Set.
author: mimckitt
ms.author: mimckitt
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 05/19/2026
ms.reviewer: wwilliams
ms.custom: mimckitt, devx-track-azurecli
ai-usage: ai-assisted
# Customer intent: As an IT administrator managing cloud resources, I want to select the appropriate number of fault domains for my Virtual Machine Scale Set, so that I can ensure high availability and minimize the risk of downtime due to hardware failures.
---
# Manage fault domains in Azure Virtual Machine Scale Sets

A fault domain is a fault isolation group within an availability zone or datacenter of hardware nodes that share the same power, networking, cooling, and platform maintenance schedule. Virtual machine (VM) instances that are on different fault domains are not likely to be impacted by the same planned or unplanned outage.

You can specify how instances are spread across fault domains within a region or zone. The fault domain configuration options available to you vary depending on the orchestration mode your scale set uses, and whether [your scale set uses availability zones](./virtual-machine-scale-sets-use-availability-zones.md).

## Fault domain spreading and alignment

There are two types of fault domain spreading: max and fixed.

### Max spreading

Azure spreads the scale set's VM instances across as many fault domains as possible, on a best-effort basis. It might use greater or fewer than five fault domains.

Set `platformFaultDomainCount` to a value of `1` for max spreading.

When you look at the scale set's VM instances, you only see one fault domain. This is expected behavior. The spreading is implicit.

> [!NOTE]
> We recommend using max spreading (`platformFaultDomainCount = 1`) for most scale sets, because it provides the best spreading in most cases. If you need your instances to be spread across distinct hardware isolation units, we recommend you configure max spreading and use multiple availability zones. Azure spreads the instances across fault domains within each zone.

### Fixed spreading

When your scale set uses fixed spreading, Azure spreads the VM instances across the specified number of fault domains. If the scale set can't allocate VMs that meet specified fault domain count, the request fails.

Set the `platformFaultDomainCount` to a value greater than 1 for fixed spreading.

There are specific numbers of fault domains you can select depending on your orchestration mode and deployment type.

### Compute-storage fault domain alignment

The fault domain count (`platformFaultDomainCount`) controls how VM instances distribute across compute fault domains. Managed disks reside in storage fault domains. Compute-storage fault domain alignment determines whether a VM's managed disks go to the storage fault domain that matches the VM's compute fault domain:

- **Aligned:** The VM and its disks share matching compute and storage fault domains. A failure in one fault domain affects only the compute and storage of VMs in that same fault domain. This setup helps prevent loss of quorum for replicated workloads. Alignment depends on available storage capacity for the desired disk type within the region or zone.
- **Unaligned:** Disks are placed without considering the VM's compute fault domain.

:::image type="content" source="media/manage-fault-domains/compute-storage-fault-domain-alignment.png" alt-text="Diagram comparing aligned and unaligned fault domains. Aligned disks A, B, and C occupy storage domains corresponding to their VMs' compute domains. In the unaligned example, disks A and B share SFD2 while disk C occupies SFD1.":::

For zonal and zone-spanning scale sets, use the [zonal fault domains preview](#use-fault-domains-with-zonal-deployments-preview) to select strict alignment, best-effort alignment, or no alignment. The *maximum aligned zonal fault domain count* is the highest compute fault domain count that aligns with storage in a zone. This value varies by region, zone, and disk type.

## Supported fault domain configurations

The following table summarizes the supported `platformFaultDomainCount` values for different orchestration modes and deployment types:

| Orchestration Mode | Deployment Type | Supported Values | Default Value |
|-------------------|----------------|------------------|---------------|
| Uniform | Zonal or zone-spanning | 1, 5; other fixed counts with strict alignment (zonal fault domains preview) | 1 |
| Uniform | Regional (nonzonal) | 1, 2, 3, 4, 5 | 5 |
| Flexible | Zonal or zone-spanning | 1; 2, 3, 4, 5 (zonal fault domains preview) | 1 |
| Flexible | Regional (nonzonal) | 1, 2, 3 | 1 |

### Uniform orchestration mode

Scale sets with Uniform orchestration support different fault domain configurations depending on the deployment type:

- **Zonal or zone-spanning scale sets:** Use max spreading by default (`platformFaultDomainCount = 1`). You can set up unaligned fixed spreading with five fault domains (`platformFaultDomainCount = 5`). By using the zonal fault domains preview, you can also use other fixed counts with strict alignment (`Aligned`), up to the maximum aligned zonal fault domain count. Uniform scale sets don't support best-effort alignment or per-disk overrides.

- **Regional (nonzonal) scale sets:** Use fixed spreading with five fault domains by default (`platformFaultDomainCount = 5`). You can optionally configure max spreading (`platformFaultDomainCount = 1`).

  You can also consider aligning the number of scale set fault domains with the number of managed disk fault domains (for example, `platformFaultDomainCount = 2`). This alignment can help prevent loss of quorum if an entire disk fault domain goes down. The fault domain count can be set to less than or equal to the number of managed disks fault domains available in each of the regions. Refer to the [availability sets documentation](../virtual-machines/availability-set-overview.md) to learn about the number of managed disk fault domains by region.

### Flexible orchestration mode

Scale sets with Flexible orchestration support different fault domain configurations depending on the deployment type:

- **Zonal or zone-spanning scale sets**: Use max spreading (`platformFaultDomainCount = 1`) by default. By using the zonal fault domains preview, you can configure fixed spreading with a fault domain count from `2` through `5` and choose a compute-storage alignment mode. For preview requirements, alignment behavior, and examples, see [Use fault domains with zonal deployments](#use-fault-domains-with-zonal-deployments-preview).

- **Regional (nonzonal) scale sets**: Use max spreading (`platformFaultDomainCount = 1`) by default. You can optionally configure fault domain counts of `2` or `3`.



## Create Virtual Machine Scale Sets with fault domains

You can configure fault domains by using standard Azure deployment tools and APIs.

### REST API

Configure the scale set's fault domain spreading by setting the property `properties.platformFaultDomainCount`. Refer to the REST API documentation for [Virtual Machine Scale Sets](/rest/api/compute/virtualmachinescalesets/createorupdate).

### Azure CLI

> [!IMPORTANT]
> VM scale sets created using PowerShell and Azure CLI default to Flexible orchestration mode if no orchestration mode is specified.

Configure the scale set's fault domain spreading by setting the `--platform-fault-domain-count` parameter. Refer to the Azure CLI documentation for [Virtual Machine Scale Sets](/cli/azure/vmss#az-vmss-create).

The following examples show how to use the Azure CLI to deploy scale sets with varying configurations:

- **Zone-spanning Flexible scale set that uses three zones, with max spreading in each zone:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --zones 1 2 3 \
    --generate-ssh-keys
  ```

- **Zonal (single-zone) Flexible scale set with max spreading:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --zones 1 \
    --generate-ssh-keys
  ```

- **Regional (nonzonal) Flexible scale set with fixed spreading:**

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Flexible \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --platform-fault-domain-count 3 \
    --generate-ssh-keys
  ```

- **Regional (nonzonal) Uniform scale set with max spreading:** 

  ```azurecli-interactive
  az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --orchestration-mode Uniform \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --platform-fault-domain-count 1 \
    --generate-ssh-keys
  ```

It takes a few minutes to create and configure all the scale set resources and VMs.

For a zonal fixed-spreading example, see [Create a zonal scale set with aligned fault domains](#create-a-zonal-scale-set-with-aligned-fault-domains).

## Use fault domains with zonal deployments (preview)

Max spreading (`platformFaultDomainCount = 1`) is available for zonal and zone-spanning scale sets without the preview. The zonal fault domains preview adds two capabilities:

- **Fixed spreading for zonal Flexible scale sets:** Set the scale set's compute fault domain count (`platformFaultDomainCount`) from `2` through `5`.
- **Compute-storage fault domain alignment:** Set `zonalPlatformFaultDomainAlignMode` to choose whether storage fault domains are strictly aligned (`Aligned`), aligned on a best-effort basis (`BestEffortAligned`), or unaligned (`Unaligned`) with compute fault domains.

> [!IMPORTANT]
> Zonal fault domains is currently in preview and isn't covered by an SLA. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). This feature isn't recommended for production workloads.

### Request access to the preview

To request access to the preview, complete the [zonal fault domains preview sign-up form](https://aka.ms/MultiFDsWithinAZone). Wait for confirmation that the preview is enabled for your subscription and region before deploying.

### Requirements for zonal fault domains

- Enable the preview for your subscription and region through the [preview sign-up process](#request-access-to-the-preview).
- Best-effort alignment and per-disk overrides require Flexible orchestration mode, `singlePlacementGroup: false`, and compute API version `2025-11-01` or later. Uniform scale sets support strict alignment (`Aligned`) only.
- For the single-zone scenario described here, specify one zone, such as `zones: ["1"]`. In zone-spanning configurations, disk alignment is evaluated within each VM's assigned zone.
- `Aligned` and `BestEffortAligned` require fixed spreading (`platformFaultDomainCount` of `2` or greater).
- During preview, the feature is available in East US, North Europe, West Europe, Canada Central, UK South, Central US, Southeast Asia, and West US 3.
- If you use `Aligned` (strict alignment), the create request can fail when the requested alignment can't be achieved. Use `BestEffortAligned` for more flexibility, or use `Unaligned` for fixed spreading without storage alignment.
- When you deploy across multiple zones, the scale set balances across zones independently of fault domain spreading within each zone.

### Choose an alignment mode

Use the `zonalPlatformFaultDomainAlignMode` property to control compute-storage fault domain alignment:

| Value | Behavior |
|-------|----------|
| `BestEffortAligned` (recommended) | For a valid compute fault domain count, the platform aligns each disk when its disk type supports that count in the VM's zone. Otherwise, the disk can remain unaligned. When a deallocated VM starts, Azure reevaluates storage capacity and aligns previously unaligned disks where possible. Restarting the guest operating system doesn't trigger this reevaluation. |
| `Aligned` | Strict alignment. The platform enforces a 1:1 match between compute and storage fault domains. If alignment can't be achieved (for example, if the zone doesn't support enough storage fault domains), the deployment fails. |
| `Unaligned` | No alignment. The platform spreads VMs across compute fault domains and doesn't match their disks to storage fault domains. |

For eligible Flexible scale sets, Microsoft recommends using `BestEffortAligned` with `platformFaultDomainCount` of `2` or `3`, up to the maximum aligned zonal fault domain count. This configuration permits individual disks to remain unaligned when their disk type doesn't support the requested count in the VM's zone. It doesn't raise the scale set's limit or guarantee allocation capacity.


### Determine aligned fault domain capabilities for a zone

Alignment involves two separate capability checks:

- **Scale-set validation:** For zonal Flexible scale sets in `Aligned` or `BestEffortAligned` mode, the compute fault domain count can't exceed the zone's maximum aligned zonal fault domain count or `5`, whichever is lower. Both modes reject a higher count. `Unaligned` allows fixed spreading up to `5`.
- **Per-disk placement:** For a valid compute fault domain count, the platform checks each disk type's maximum aligned zonal fault domain count in the VM's zone. `Aligned` requires the disk type to support the compute fault domain count. `BestEffortAligned` permits the disk to remain unaligned when the disk type doesn't support it.

The following examples illustrate the difference. These values aren't universal limits:

| Compute fault domain count | Zone's maximum aligned zonal fault domain count | Disk type's maximum aligned zonal fault domain count | Result |
|---|---|---|---|
| `2` | `3` | `3` | Alignment is possible. |
| `3` | `3` | `2` | A strictly aligned disk fails validation; a best-effort disk can remain unaligned. |
| `4` | `3` | Any | Scale-set creation is rejected for both `Aligned` and `BestEffortAligned`. |

Use the following query to check the disk type's current capabilities in your target location:

```azurecli
az vm list-skus --location eastus2euap --resource-type disks --query "[?size=='P40']" --output json
```

The response includes `zoneDetails` with `MaxFaultDomainCountInZone`, the disk type's maximum aligned zonal fault domain count, for each availability zone:

```json
{
  "name": "Premium_LRS",
  "size": "P40",
  "locationInfo": [
    {
      "location": "eastus2euap",
      "zoneDetails": [
        {
          "capabilities": [
            {
              "name": "MaxFaultDomainCountInZone",
              "value": "3"
            }
          ],
          "name": ["1"]
        },
        {
          "capabilities": [
            {
              "name": "MaxFaultDomainCountInZone",
              "value": "3"
            }
          ],
          "name": ["2"]
        }
      ]
    }
  ]
}
```

In this example, zones 1 and 2 support up to three aligned fault domains for the disk type. A zone that doesn't support alignment might not appear in the response.

> [!TIP]
> A disk with `Aligned` intent requires `MaxFaultDomainCountInZone` to be at least the scale set's compute fault domain count. A disk with `BestEffortAligned` intent can remain unaligned if the value is lower. In both cases, the compute fault domain count must still be within the zone's maximum aligned zonal fault domain count.

### Create a zonal scale set with aligned fault domains

The following examples show the key settings for a single-zone Flexible scale set. [Request access to the preview](#request-access-to-the-preview) and meet the [zonal fault domain requirements](#requirements-for-zonal-fault-domains) before deploying.

#### [Azure CLI](#tab/cli-aligned)

Set the fault domain count, placement group setting, and alignment mode explicitly:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --orchestration-mode Flexible \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --zones 1 \
  --platform-fault-domain-count 3 \
  --single-placement-group false \
  --zonal-fault-domain-align-mode BestEffortAligned \
  --generate-ssh-keys
```

#### [Bicep](#tab/bicep-aligned)

The following resource definition shows the scale-set properties for best-effort alignment:

```bicep
resource vmss 'Microsoft.Compute/virtualMachineScaleSets@2025-11-01' = {
  name: 'myScaleSet'
  location: resourceGroup().location
  zones: [
    '1'
  ]
  sku: {
    name: 'Standard_D2s_v5'
    capacity: 3
  }
  properties: {
    orchestrationMode: 'Flexible'
    singlePlacementGroup: false
    platformFaultDomainCount: 3
    zonalPlatformFaultDomainAlignMode: 'BestEffortAligned'
  }
}
```

---

### Assign a fault domain during VM creation

By default, Azure assigns the fault domain when you add a VM to a Flexible scale set. For advanced or troubleshooting scenarios, you can specify the fault domain for a new VM in a scale set that uses fixed spreading:

```azurecli-interactive
az vm create \
  --name myVM \
  --resource-group myResourceGroup \
  --image Ubuntu2204 \
  --vmss myScaleSet \
  --platform-fault-domain 1
```

### Per-disk alignment overrides

When you create a VM in an eligible zonal or zone-spanning Flexible scale set, you can set `storageFaultDomainAlignment` on an OS disk or data disk. Alignment is evaluated within the VM's zone. The platform rejects per-disk alignment on a standalone VM.

Valid values are `Aligned` and `BestEffortAligned`; there's no per-disk `Unaligned` value. An omitted value inherits the scale set's `zonalPlatformFaultDomainAlignMode`. To permit an individual disk to remain unaligned in an `Aligned` scale set, specify `BestEffortAligned`. This value relaxes the alignment requirement but doesn't force unaligned placement.

| Scale set alignment mode | Per-disk `Aligned` | Per-disk `BestEffortAligned` | Per-disk omitted |
|--------------------------|--------------------|------------------------------|------------------|
| `Aligned` | Allowed (matches the scale set). | Allowed (best effort within an aligned scale set). | Allowed (inherits strict alignment). |
| `BestEffortAligned` | Incompatible. | Allowed (matches the scale set). | Allowed (inherits best-effort behavior). |
| `Unaligned` | Incompatible. | Incompatible. | Allowed (disk remains unaligned). |

> [!NOTE]
> For compatibility, a disk with `optimizeForFrequentAttach` enabled can retain unaligned placement in an `Aligned` scale set when the per-disk alignment value is omitted. Set the per-disk value explicitly to request alignment for that disk.

The following examples assume a single-zone `Aligned` scale set with per-disk alignment enabled. For the VM creation and attachment workflow, see [Attach or detach a VM](virtual-machine-scale-sets-attach-detach-vm.md).

#### [Azure CLI](#tab/cli-disk)

When creating a VM in a zonal scale set, specify the per-disk alignment parameter:

```azurecli-interactive
az vm create \
  --name myVM \
  --resource-group myResourceGroup \
  --image Ubuntu2204 \
  --vmss myVMScaleSet \
  --zone 1 \
  --os-disk-name myOSDisk \
  --os-disk-storage-fd-alignment BestEffortAligned
```

To also override a data disk, add `--data-disk-storage-fd-alignment`. Omit the parameter to inherit the scale set's alignment mode.

#### [Azure PowerShell](#tab/powershell-disk)

When creating a VM in a zonal scale set, set `-StorageFaultDomainAlignment` on the OS disk and on each data disk whose alignment requirement you want to specify. The `Aligned` data disk in this example requires the scale set to use `Aligned` mode:

```azurepowershell-interactive
$vm = New-AzVMConfig -VMName 'myVM' -VMSize 'Standard_D2s_v5'

Set-AzVMOSDisk -VM $vm `
    -CreateOption 'FromImage' `
    -StorageFaultDomainAlignment 'BestEffortAligned'

# Optional: add a data disk and force alignment
Add-AzVMDataDisk -VM $vm -Lun 0 -CreateOption 'Empty' `
    -DiskSizeInGB 128 `
    -StorageFaultDomainAlignment 'Aligned'

New-AzVM `
    -ResourceGroupName 'myResourceGroup' `
    -Location 'East US' `
    -VM $vm `
    -VirtualMachineScaleSetId $vmss.Id
```

---

> [!IMPORTANT]
> Per-disk alignment requires a VM in an eligible zonal Flexible scale set. Setting `storageFaultDomainAlignment` on a standalone VM returns `BadRequest`. A `BestEffortAligned` scale set doesn't permit the explicit `Aligned` data disk shown in the Azure PowerShell example.

### Validate disk alignment across scale set VMs

Use the VM model to check the requested alignment and the VM instance view to check the actual disk alignment. The `storageFaultDomainAlignment` property in the VM model shows the alignment you want. The `storageAlignmentStatus` property in the instance view shows the actual alignment, which can be different for best-effort disks.

The following Azure CLI example uses REST API version `2025-11-01` so that both alignment properties are available:

```azurecli
# Get all VMs in the scale set
vmIds=$(az vmss list-instances \
  --resource-group myRg \
  --name myVmss \
  --query "[].id" -o tsv)

# For each VM, compare requested alignment with actual alignment
for vmId in $vmIds; do
  az rest --method get \
    --url "https://management.azure.com${vmId}?api-version=2025-11-01" \
    --query "{vmName:name, osDisk:properties.storageProfile.osDisk.{diskName:name, targetAlignment:storageFaultDomainAlignment}, dataDisks:properties.storageProfile.dataDisks[].{diskName:name, targetAlignment:storageFaultDomainAlignment}}"

  az rest --method get \
    --url "https://management.azure.com${vmId}/instanceView?api-version=2025-11-01" \
    --query "disks[].{diskName:name, alignmentState:storageAlignmentStatus}"
done
```

## Next steps

- Learn more about [availability and redundancy features](../virtual-machines/availability.md) for Azure environments.
