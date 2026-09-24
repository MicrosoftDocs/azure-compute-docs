---
title: Zone-resilient virtual machines (Preview)
description: Learn how zone-resilient virtual machines use zone-redundant storage disks to support customer-initiated recovery in another availability zone.
author: ajkundna
ms.author: ajkundna
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 09/19/2026
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to recover an eligible zonal VM in another availability zone while preserving its resource configuration."
---

Zone Resilient virtual machines (ZR VMs) help you recover an eligible zonal Azure virtual machine (VM) in another availability zone in the same region. The recovery process updates the zone placement of the existing VM and reuses its supported zone-redundant storage (ZRS) managed disks. You don't have to create a replacement VM.

Zone Resilient VM provides a customer-initiated zonal recovery option for a single VM. It doesn't provide automatic failover, uninterrupted service, or application-level recovery orchestration. For higher availability, deploy multiple VM instances across availability zones and distribute traffic between them.

Because ZRS synchronously replicates managed disk data across availability zones, ZR VMs provide a near-zero recovery point objective (RPO) for data committed to supported ZRS disks. This RPO doesn't apply to active requests, in-memory data, temporary disks, or application state that hasn't been written to the ZRS disk.

## How zone recovery works

After a failure affects a VM that's enabled for zone movement, you can initiate recovery in a healthy zone in the same region:

:::image type="complex" source="media/zone-resilient-vm/zone-recovery-workflow.png" alt-text="Customer-initiated zone recovery: confirm impact, force-deallocate, change zones, start, and verify the VM.":::
  Confirm the VM is affected and select a healthy zone in the same region. Force-deallocate the VM, which stops the workload. Update the zone placement of the existing VM and reuse its supported ZRS disks. Start the VM in the target zone, subject to available capacity. Verify the VM, disks, network, and workload before restoring traffic.
:::image-end:::

The workload is unavailable while the VM is deallocated and started in the target zone. Active requests, in-memory data, and runtime state aren't preserved.

The target zone must support the VM size and have enough capacity when you start the VM.

## Requirements

Before you enable Zone Resilient VM, confirm that your configuration meets these requirements:

| Resource | Requirement |
| --- | --- |
| VM placement | Use a new or existing VM deployed in a specific availability zone. |
| OS disk | Use a Premium SSD ZRS or Standard SSD ZRS managed disk. |
| Data disks | Use Premium SSD ZRS or Standard SSD ZRS for every attached data disk. |
| Private IP address | You can use a static or dynamic private IP address. |
| Public IP address | If used, configure a zone-redundant Standard SKU public IP address. |
| Network appliance | If used, configure a zone-redundant Standard SKU load balancer or supported gateway. |
| Management tool | Use Azure CLI version 2.86.0 or later, Az.Compute version 11.9.0 or later, or Compute REST API version `2025-04-01`. |

For current ZRS disk types and regional availability, see [Redundancy options for managed disks](disks-redundancy.md#zone-redundant-storage-for-managed-disks). To create ZRS disks, see [Deploy a managed disk that uses zone-redundant storage](disks-deploy-zrs.md).

## Limitations

Zone Resilient VM has the following limitations:

- Recovery is customer initiated. Azure doesn't automatically move a standalone VM when a zone fails.
- Recovery causes downtime because the VM is force-deallocated and restarted.
- Recovery is limited to another availability zone in the same region. It doesn't protect against a regional outage or an outage that affects multiple zones.
- Recovery doesn't preserve active requests, memory, or runtime state.
- Recovery can fail when the VM size has insufficient capacity in the target zone.
- Locally redundant storage (LRS) OS and data disks aren't supported.
- Ephemeral OS disks aren't supported.
- Data disks that use write acceleration aren't supported.
- VM scale sets, Availability set VMs and Spot VMs aren't supported.
- The feature doesn't provide application-level consistency, traffic failover, recovery plans, or failback orchestration.
- Azure Site Recovery isn't supported.
- On-demand capacity reservations aren't supported.
- Dedicated hosts aren't supported.
- Proximity placement groups aren't supported.

<!-- Confirm whether hibernation, InfiniBand need explicit preview validation before publication. -->

## Prepare a VM for zone recovery

Create a VM that meets every requirement before you enable zone movement. Enablement must occur while the source availability zone is healthy.

1. In the [Azure portal](https://portal.azure.com/), create a VM in a specific availability zone.
1. On the **Disks** tab, select **Premium SSD ZRS** or **Standard SSD ZRS** for the OS disk and every data disk.
1. Configure only supported zone-redundant Standard SKU network resources.
1. Create the VM.

## Enable zone movement

Enable zone movement when you create a VM or on an eligible existing zonal VM. An existing VM must be deallocated before enablement; a new VM doesn't need a separate deallocation step. Enable zone movement while the source zone is healthy.

# [Azure CLI](#tab/azure-cli-enable)

Use Azure CLI 2.86.0 or later. Confirm your installed version with `az version`.

To create a zonal Linux VM with a ZRS OS disk and zone movement enabled, run [az vm create](/cli/azure/vm#az-vm-create). This example doesn't create a public IP address. Add supported zone-redundant network resources for your workload.

```azurecli
az vm create \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --location "<region>" \
  --zone "<source-zone>" \
  --image Ubuntu2204 \
  --size "<vm-size>" \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Premium_ZRS \
  --public-ip-address "" \
  --zone-movement true
```

To enable zone movement on an eligible existing zonal VM, deallocate it first:

```azurecli
az vm deallocate \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

After the VM is deallocated, run [az vm update](/cli/azure/vm#az-vm-update):

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --zone-movement true
```

After the update succeeds, start the VM:

```azurecli
az vm start \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

Confirm that zone movement is enabled:

```azurecli
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query resiliencyProfile.zoneMovement.isEnabled \
  --output tsv
```

The command returns `true`.

# [Azure PowerShell](#tab/azure-powershell-enable)

Use Az.Compute 11.9.0 or later. Confirm your newest installed version with `(Get-Module Az.Compute -ListAvailable | Sort-Object Version -Descending | Select-Object -First 1).Version`.

For a new VM, pass the `PSVirtualMachine` configuration from `New-AzVMConfig` through `Set-AzVMZoneMovement -IsEnabled $true` before you call `New-AzVM`. Configure the required zone, ZRS disks, network resources, and tag on the VM configuration.

To enable zone movement on an eligible existing zonal VM, use [Get-AzVM](/powershell/module/az.compute/get-azvm), [Set-AzVMZoneMovement](/powershell/module/az.compute/set-azvmzonemovement), and [Update-AzVM](/powershell/module/az.compute/update-azvm):

```azurepowershell
$resourceGroupName = "<resource-group-name>"
$vmName = "<vm-name>"

Stop-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName `
  -Force
```

After the VM is deallocated, enable zone movement:

```azurepowershell
$vm = Get-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName

$vm = Set-AzVMZoneMovement -VM $vm -IsEnabled $true

Update-AzVM `
  -ResourceGroupName $resourceGroupName `
  -VM $vm
```

After the update succeeds, start the VM:

```azurepowershell
Start-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName
```

Confirm that zone movement is enabled:

```azurepowershell
$vm = Get-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName

$vm.ResiliencyProfile.ZoneMovement.IsEnabled
```

The command returns `True`.

# [Bicep](#tab/bicep-enable)

To create a new zonal VM with a Premium SSD ZRS OS disk and zone movement enabled, add the following VM resource excerpt to a complete Bicep template. Define `vmName` and `location` in that template, and include the other VM properties required for your deployment. No deallocation is needed when creating a VM.

```bicep
@description('Availability zone for the virtual machine.')
@allowed([
  '1'
  '2'
  '3'
])
param availabilityZone string = '1'

resource vm 'Microsoft.Compute/virtualMachines@2025-04-01' = {
  name: vmName
  location: location
  zones: [
    availabilityZone
  ]
  properties: {
    storageProfile: {
      osDisk: {
        name: '${vmName}-osdisk'
        createOption: 'FromImage'
        caching: 'ReadWrite'
        deleteOption: 'Delete'
        managedDisk: {
          storageAccountType: 'Premium_ZRS'
        }
      }
      dataDisks: []
    }
    resiliencyProfile: {
      zoneMovement: {
        isEnabled: true
      }
    }
  }
}
```

Next, deploy the complete template from a local file named `enable-zone-movement.bicep`:

```azurecli
az deployment group create \
  --resource-group "<resource-group-name>" \
  --template-file "./enable-zone-movement.bicep" \
  --parameters vmName="<vm-name>" availabilityZone="<source-zone>"
```

Wait for deployment to succeed before you initiate a zone move.

For an eligible existing zonal VM, set `zoneMovement.isEnabled` to `true` in its complete Bicep definition while the source zone is healthy. Preserve its current zone, disks, network interfaces, and other settings. Don't use the new-VM excerpt's `osDisk.createOption: 'FromImage'` to update an existing VM.

Deallocate the existing VM:

```azurecli
az vm deallocate \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

After the VM is deallocated, deploy its complete Bicep template:

```azurecli
az deployment group create \
  --resource-group "<resource-group-name>" \
  --template-file "./enable-existing-zone-movement.bicep" \
  --parameters vmName="<vm-name>"
```

After the deployment succeeds, start the VM:

```azurecli
az vm start \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

---

## Recover a VM in another zone

Use zone recovery after you confirm that an availability zone failure affects the VM. Monitor individual VM health with [Azure Resource Health](/azure/service-health/resource-health-overview), and monitor broader service issues with [Azure Service Health](/azure/service-health/overview).

Choose a target zone that supports your VM size and has sufficient capacity.

> [!WARNING]
> Force deallocation stops the VM and discards its in-memory state. Start recovery only after you confirm the impact on your workload and its dependencies.

Use Azure CLI to orchestrate recovery. With Azure PowerShell or Bicep, perform each recovery operation in order.

# [Azure CLI](#tab/azure-cli-recover)

Run [az vm update](/cli/azure/vm#az-vm-update) with the target zone:

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --zone "<target-zone>"
```

The command force-deallocates the VM, updates its availability zone, and starts the VM in the target zone. The command waits for each operation to finish. If the zone update fails, Azure CLI attempts to restart the VM before it returns the update error.

> [!NOTE]
> Don't deallocate the VM separately. Azure CLI handles deallocation as part of the zone update.

# [Azure PowerShell](#tab/azure-powershell-recover)

Azure PowerShell doesn't provide a single cmdlet that orchestrates zone recovery. Run the following cmdlets in order, and wait for each cmdlet to finish before you continue:

```azurepowershell
$resourceGroupName = "<resource-group-name>"
$vmName = "<vm-name>"
$targetZone = "<target-zone>"

Stop-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName `
  -ForceDeallocate `
  -Force

$vm = Get-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName

$vm.Zones = [string[]]@($targetZone)

Update-AzVM `
  -ResourceGroupName $resourceGroupName `
  -VM $vm

Start-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName
```

`Stop-AzVM -ForceDeallocate` releases or isolates the source-zone resources. Setting `Zones` and calling `Update-AzVM` changes the VM placement. `Start-AzVM` then starts the VM in the target zone.

> [!IMPORTANT]
> Unlike the Azure CLI recovery command, this PowerShell sequence isn't transactional. If `Update-AzVM` fails, review the error and the VM state before you run `Start-AzVM`.

# [Bicep](#tab/bicep-recover)

Bicep updates the VM's zone placement, but doesn't deallocate or start the VM. Run the following steps in order, waiting for each command to finish before continuing.

1. Force-deallocate the existing VM:

    ```azurecli
    az vm deallocate \
      --resource-group "<resource-group-name>" \
      --name "<vm-name>" \
      --force-deallocate
    ```

1. In a complete Bicep definition of the **existing** VM, set `zones` to the healthy target zone:

    ```bicep
    @description('Healthy target zone in the VM region.')
    param targetZone string

    resource vm 'Microsoft.Compute/virtualMachines@2025-04-01' = {
      name: vmName
      location: location
      zones: [
        targetZone
      ]
      properties: {
        resiliencyProfile: {
          zoneMovement: {
            isEnabled: true
          }
        }
      }
    }
    ```

    This is an excerpt, not a deployable VM definition. Keep the existing VM's full configuration, disk attachments, and network interfaces in your Bicep file. Don't reuse a new-VM template with `osDisk.createOption: 'FromImage'` to update an existing VM.

    Deploy the complete file after force-deallocation succeeds:

    ```azurecli
    az deployment group create \
      --resource-group "<resource-group-name>" \
      --template-file "./move-vm-zone.bicep" \
      --parameters vmName="<vm-name>" targetZone="<target-zone>"
    ```

1. After the deployment succeeds and the VM reports the target zone, start it:

    ```azurecli
    az vm start \
      --resource-group "<resource-group-name>" \
      --name "<vm-name>"
    ```

If the deployment fails, inspect the error and VM state before starting the VM. Verify the disks, network configuration, and workload before restoring traffic.

---

## Verify recovery succeeded

After the recovery operation finishes, verify the VM and the workload before you restore traffic:

1. Confirm that the VM power state is **Running**.
1. Confirm that the VM reports the selected target availability zone.
1. Confirm that the expected OS and data disks are attached.
1. Confirm that the NIC, subnet, network security group, load balancer, and public IP associations are correct for your configuration.
1. Confirm that VM extensions and management settings are present and healthy.
1. Connect to the VM and verify operating system and application health.
1. Run workload-specific data consistency checks.
1. Restore traffic only after your health checks succeed.

Use Azure CLI or Azure PowerShell to check the VM state and zone:

# [Azure CLI](#tab/azure-cli-verify)

```azurecli
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --show-details \
  --query "{zone:zones[0], zoneMovementEnabled:resiliencyProfile.zoneMovement.isEnabled, powerState:powerState}" \
  --output table
```

Confirm that `zone` is the target zone, `zoneMovementEnabled` is `true`, and `powerState` is `VM running`.

# [Azure PowerShell](#tab/azure-powershell-verify)

```azurepowershell
$vm = Get-AzVM `
    -ResourceGroupName "<resource-group-name>" `
    -Name "<vm-name>" `
    -Status

$vm.Zones
$vm.ResiliencyProfile.ZoneMovement.IsEnabled
$vm.Statuses | Select-Object Code, DisplayStatus
```

Confirm that `Zones` contains the target zone, `IsEnabled` is `True`, and the power state is **VM running**.

---

## Troubleshoot recovery

Match the error identifier or its message to the operation you attempted:

| Error identifier | When it occurs | What to do |
| --- | --- | --- |
| `ZoneMovementUpdateRequiresDeallocatedVM` | Enabling or disabling zone movement on a VM that isn't deallocated. | Deallocate the VM, retry the enable or disable update, and start the VM after it succeeds. |
| `OperationNotAllowedForNonZRSDisks` | The OS disk or an attached data disk doesn't use supported ZRS storage. | Use Premium SSD ZRS or Standard SSD ZRS for every disk before enabling zone movement. |
| `ForceDeallocateVMNotAllowedForNonResilientVM` | Force deallocation was requested for a VM without zone movement enabled. | If the source zone is healthy, deallocate normally and enable zone movement before retrying. If the source zone is unavailable, contact Azure support. |
| `OperationNotAllowedOnNullCurrentPowerStateForZonePatch` | A zone update was attempted before the affected VM reached **Deallocated**. | Force-deallocate the enabled VM, confirm it reaches **Deallocated**, and retry the zone update. |
| `OperationNotAllowedForNonZonalVM` | A zone update was requested for a regional VM without a pinned zone. | Confirm that the VM is zonal. This zone-update workflow applies only to zonal VMs. |
| `VMZoneMigrationNotCompleted` | The VM was started before zone migration finished or after migration failed. | Wait for the zone update to succeed, then retry the start. If the error persists, open an Azure support request. |

If the VM can't start because the target zone lacks capacity, select another supported zone. If the VM starts but the workload is unhealthy, complete the checks in [Verify recovery succeeded](#verify-recovery-succeeded) before restoring traffic. For other errors, review Azure service health and the operation details; open an Azure support request if recovery remains blocked.

## Disable zone movement

The VM must be deallocated before you disable zone movement. Deallocation interrupts the workload. Complete each operation before continuing, and start the VM only after the update succeeds:

# [Azure CLI](#tab/azure-cli-disable)

```azurecli
az vm deallocate \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

After the VM is deallocated, disable zone movement:

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --zone-movement false
```

After the update succeeds, start the VM:

```azurecli
az vm start \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

# [Azure PowerShell](#tab/azure-powershell-disable)

```azurepowershell
$resourceGroupName = "<resource-group-name>"
$vmName = "<vm-name>"

Stop-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName `
  -Force
```

After the VM is deallocated, disable zone movement:

```azurepowershell
$vm = Get-AzVM `
    -ResourceGroupName $resourceGroupName `
    -Name $vmName

$vm = Set-AzVMZoneMovement -VM $vm -IsEnabled $false

Update-AzVM `
    -ResourceGroupName $resourceGroupName `
    -VM $vm
```

After the update succeeds, start the VM:

```azurepowershell
Start-AzVM `
  -ResourceGroupName $resourceGroupName `
  -Name $vmName
```

# [Bicep](#tab/bicep-disable)

In the complete Bicep definition of the existing VM, set `zoneMovement.isEnabled` to `false`. The following resource block is an excerpt. Keep the VM's current zone, disks, network interfaces, and other configuration in the deployed template.

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2025-04-01' = {
  name: vmName
  location: location
  properties: {
    resiliencyProfile: {
      zoneMovement: {
        isEnabled: false
      }
    }
  }
}
```

Deallocate the VM:

```azurecli
az vm deallocate \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

After the VM is deallocated, deploy the complete template from a local file named `disable-zone-movement.bicep`:

```azurecli
az deployment group create \
  --resource-group "<resource-group-name>" \
  --template-file "./disable-zone-movement.bicep" \
  --parameters vmName="<vm-name>"
```

After the deployment succeeds, start the VM:

```azurecli
az vm start \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

---

## Cost

Pricing for the Zone Resilient VM feature hasn't been disclosed. Feature pricing will be announced by December 2026.

Feature pricing will be in addition to existing standard charges for VMs, managed disks, and network resources.

<!-- The preview repository doesn't define a feature-specific charge. Confirm pricing before publication. -->

## Related content

- [Reliability in Azure Virtual Machines](/azure/reliability/reliability-virtual-machines)
- [Availability options for Azure Virtual Machines](availability.md)
- [Reliability in Azure Disk Storage](/azure/reliability/reliability-storage-disk)
- [Azure Site Recovery overview](/azure/site-recovery/site-recovery-overview)