---
title: zone resilience for Azure VMs (Preview)
description: Learn how zone resilience for Azure VMs uses ZRS managed disks for customer-initiated failover to another availability zone.
author: AjKundnani
ms.author: ajkundna
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 10/20/2026
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to fail over an eligible zonal VM to another availability zone while preserving its resource configuration."
---

# Zone resilience for Azure VMs (Preview)

Zone resilience for Azure VMs lets you initiate failover of an eligible zonal Azure virtual machine (VM) to another availability zone in the same region. Failover changes the zone placement of the existing VM without creating a replacement. The VM retains its resource ID and VM ID, attached network interface (NIC) and its IP configuration, supported zone-redundant storage (ZRS) managed disks, and other supported resource settings. The zone placement changes; the supported configuration remains associated with the VM.

Zone failover is customer-initiated. Don't use it for production VMs during a zone outage. This limitation is temporary and in preview. Support for this scenario is planned for a future update. See [Limitations](#limitations). For high availability across zones, [deploy multiple VMs across availability zones](disks-high-availability.md#deploy-vms-and-disks-across-three-availability-zones) and distribute traffic between them.

Because ZRS synchronously replicates managed disk data across availability zones, zone resilience provides a near-zero recovery point objective (RPO) for data committed to supported ZRS disks.

## How zone failover works

For an eligible VM affected by a zone failure, the failover workflow moves the VM to a healthy zone in the same region. Don't use this workflow for production VMs during a zone outage. This limitation is temporary and in preview. Support for this scenario is planned for a future update. See [Limitations](#limitations).

:::image type="complex" source="media/zone-resilience/zone-recovery-workflow.png" alt-text="Diagram of Customer-initiated zone failover: confirm impact, force-deallocate, change zones, start, and verify the VM.":::
  Confirm the VM is affected and select a healthy zone in the same region. Force-deallocate the VM, which stops the workload. Update the zone placement of the existing VM and reuse its supported ZRS disks. Start the VM in the target zone, subject to available capacity. Verify the VM, disks, network, and workload before restoring traffic.
:::image-end:::

The workload is unavailable while the VM is deallocated and started in the target zone. Active requests, in-memory data, and runtime state aren't preserved.

The target zone must support the VM size and have enough capacity when you start the VM.

## Requirements

Before you enable zone resilience for Azure VMs, confirm that your configuration meets these requirements:

| Resource | Requirement |
| --- | --- |
| VM placement | Use a new or existing VM deployed in a specific availability zone. |
| OS disk | Use a Premium SSD ZRS or Standard SSD ZRS managed disk. |
| Data disks | Use Premium SSD ZRS or Standard SSD ZRS for every attached data disk. |
| Private IP address | Use a static or dynamic private IP address. |
| Public IP address | If used, configure a zone-redundant Standard SKU public IP address. |
| Network appliance | If used, configure a zone-redundant Standard SKU load balancer or supported gateway. |
| Management tool | Use the Azure portal, Azure CLI version 2.87.0 or later, Azure PowerShell with zone movement and force deallocation support, or Compute REST API version `2026-03-01`. |

For current ZRS disk types and regional availability, see [Redundancy options for managed disks](disks-redundancy.md#zone-redundant-storage-for-managed-disks). To create ZRS disks, see [Deploy a managed disk that uses zone-redundant storage](disks-deploy-zrs.md).

## Limitations

> [!IMPORTANT]
> Don't use zone resilience for Azure VMs (Preview) to fail over production VMs during an availability zone outage. Doing so can cause data or disk corruption. This limitation is temporary; support for this scenario is planned for a future update. Microsoft will announce when production VM failover during a zone outage is supported.

The following limitations result from the requirement to use ZRS managed disks:

- Locally redundant storage (LRS) OS and data disks aren't supported. Every attached OS and data disk must use Premium SSD ZRS or Standard SSD ZRS.
- Ephemeral OS disks aren't supported because they use local VM storage.
- Premium SSD v2, Ultra Disks, and Standard HDD managed disks aren't supported because these disk types don't offer ZRS. See [ZRS limitations](disks-redundancy.md#zrs-limitations).
- Data disks that use write acceleration aren't supported.

To prepare a zone VM with LRS disks, follow [Convert a disk from LRS to ZRS](disks-migrate-lrs-zrs.md#zonal-migration) to create ZRS disks from snapshots of the OS and data disks. You can't change a zone LRS disk's redundancy in place, but you can keep the existing VM if its disks meet the [OS disk swap requirements](linux/os-disk-swap.md#os-disk-swap-requirements). Swap in the ZRS OS disk, then detach the LRS data disks and attach the corresponding ZRS disks. Plan for downtime, and confirm the OS and every attached data disk use a supported ZRS type before you enable zone movement. The [regional disk conversion](disks-migrate-lrs-zrs.md#regional-migration) procedure doesn't apply to zone disks.

The following limitations apply to zone movement and its supported VM configurations:

- Failover is limited to another availability zone in the same region. It doesn't protect against a regional outage or an outage that affects multiple zones.
- Failover can fail when the VM size has insufficient capacity in the target zone.
- VM scale sets, availability set VMs, and Spot VMs aren't supported.
- Azure Site Recovery isn't supported for VMs with zone resilience enabled.
- On-demand capacity reservations, dedicated hosts, and proximity placement groups aren't supported.
- The feature doesn't provide automatic failover, application-level consistency, traffic failover, recovery plans, or failback orchestration.

## Prepare a VM for zone failover

Create a VM that meets every requirement before you enable zone movement. Enablement must occur while the source availability zone is healthy.

1. In the [Azure portal](https://portal.azure.com/), create a VM in a specific availability zone.
1. On the **Disks** tab, select **Premium SSD ZRS** or **Standard SSD ZRS** for the OS disk and every data disk.
1. Configure only supported zone-redundant Standard SKU network resources.
1. Create the VM.

## Enable zone movement

Enable zone movement when you create a VM or on an eligible existing zonal VM. You must deallocate an existing VM before you can enable zone movement; a new VM doesn't need a separate deallocation step. Enable zone movement while the source zone is healthy.

# [Azure portal](#tab/azure-portal-enable)

To enable zone movement when you create a VM:

1. Sign in to the [Azure portal](https://portal.azure.com/), and start creating a VM.
1. On the **Basics** tab, select a region that supports availability zones. Set **Availability options** to **Availability zone**, and select a specific **Availability zone**.
1. Under **Availability zone**, select **Protect VM from zonal-failure with Zone resilience (Preview)**.
1. In the **Zone resilience (Preview)** pane, select **Enable Zone resilience (Preview)**. Review the configuration requirements, and select **Save**.

    :::image type="content" source="media/zone-resilience/zone-resilience-enable-new-vm.png" alt-text="Screenshot of the VM creation page with the zone resilience link, enable checkbox, and Save button highlighted." lightbox="media/zone-resilience/zone-resilience-enable-new-vm.png":::

1. On the **Disks** tab, confirm that the OS disk and every data disk use **Premium SSD ZRS** or **Standard SSD ZRS**. Configure supported network resources and the remaining VM settings.
1. Select **Review + create**, and then select **Create** after validation succeeds.

To enable zone movement on an eligible existing zonal VM:

1. In the Azure portal, open the VM, and select **Overview**.
1. Select **Stop**, confirm the operation, and wait until the VM status is **Stopped (deallocated)**. Stopping the VM interrupts the workload.
1. On **Overview**, under **Availability + scaling**, select the current value of **Zone Resilience (Preview)**.
1. In the **Zone resilience (Preview)** pane, select **Enable Zone resilience (Preview)**, review the configuration requirements, and select **Save**.

    :::image type="content" source="media/zone-resilience/zone-resilience-enable-existing-vm.png" alt-text="Screenshot of an existing VM overview and the zone resilience pane with the enable checkbox highlighted." lightbox="media/zone-resilience/zone-resilience-enable-existing-vm.png":::

1. After the update succeeds, select **Start** on **Overview**.
1. Confirm that **Zone Resilience (Preview)** shows **Enabled**.

# [Azure CLI](#tab/azure-cli-enable)

Use Azure CLI 2.87.0 or later. Confirm your installed version with `az version`. This version introduces zone movement and force deallocation support; see the [Azure CLI release notes](/cli/azure/release-notes-azure-cli#june-02-2026).

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

When the update succeeds, start the VM:

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

Use a version of Az.Compute that includes [Set-AzVMZoneMovement](/powershell/module/az.compute/set-azvmzonemovement) and supports `Stop-AzVM -ForceDeallocate`. Confirm that the zone movement cmdlet is available with `Get-Command Set-AzVMZoneMovement`, and check the deallocation parameters with `Get-Command Stop-AzVM -Syntax`.

For a new VM, pass the `PSVirtualMachine` configuration from `New-AzVMConfig` through `Set-AzVMZoneMovement -IsEnabled $true` before you call `New-AzVM`. Configure the required zone, ZRS disks, and network resources on the VM configuration.

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

To create a new zonal VM with a Premium SSD ZRS OS disk and zone movement enabled, add the following VM resource excerpt to a complete Bicep template. Define `vmName` and `location` in that template, and include the other VM properties required for your deployment. You don't need to deallocate the VM when creating it.

```bicep
@description('Availability zone for the virtual machine.')
@allowed([
  '1'
  '2'
  '3'
])
param availabilityZone string = '1'

resource vm 'Microsoft.Compute/virtualMachines@2026-03-01' = {
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

When the deployment succeeds, start the VM:

```azurecli
az vm start \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

---

## Fail over a VM to another zone

Don't use this procedure to fail over a production VM during a zone outage because it can cause data or disk corruption. This limitation is temporary and is currently in preview. Support for this scenario is planned for a future update. See [Limitations](#limitations). For eligible nonproduction VMs, first confirm that an availability zone failure affects the VM. Monitor individual VM health with [Azure Resource Health](/azure/service-health/resource-health-overview), and monitor broader service issues with [Azure Service Health](/azure/service-health/overview).

Choose a target zone that supports your VM size and has sufficient capacity.

> [!WARNING]
> Force deallocation stops the VM and discards its in-memory state. Start failover only after you confirm the impact on your workload and its dependencies.

Use Azure CLI to orchestrate failover. With the Azure portal, Azure PowerShell, or Bicep, perform each failover operation in order.

# [Azure portal](#tab/azure-portal-recover)

Use the portal when the VM is already deallocated or can be stopped normally. If normal deallocation can't complete, use the Azure CLI, Azure PowerShell, or Bicep procedure for force deallocation.

1. In the [Azure portal](https://portal.azure.com/), open the VM, and select **Overview**.
1. If the VM isn't already deallocated, select **Stop**, confirm the operation, and wait until its status is **Stopped (deallocated)**. Don't continue if deallocation fails.
1. Under **Availability + scaling**, next to **Availability zone**, select **(edit)**.

    :::image type="content" source="media/zone-resilience/zone-resilience-edit-availability-zone.png" alt-text="Screenshot of the VM overview with the edit link next to Availability zone highlighted." lightbox="media/zone-resilience/zone-resilience-edit-availability-zone.png":::

1. On the **Availability Zones** tab, select the target **Availability zone**, and select **Apply**.

    :::image type="content" source="media/zone-resilience/zone-resilience-apply-availability-zone.png" alt-text="Screenshot of the Availability Zones tab with the deallocated status, target zone selector, and Apply button highlighted." lightbox="media/zone-resilience/zone-resilience-apply-availability-zone.png":::

1. Wait for the update to succeed, and confirm that **Availability zone** reports the target zone. If the update fails, review the error and VM state before continuing.
1. Return to **Overview**, and select **Start**.
1. Complete the checks in [Verify failover succeeded](#verify-failover-succeeded) before restoring traffic.

The VM remains deallocated after you apply the zone change. Start the VM in target zone post zone change operation completes successfully to resume normal operations by selecting **Start** in the Azure portal.

# [Azure CLI](#tab/azure-cli-recover)

Run [az vm update](/cli/azure/vm#az-vm-update) with the target zone:

```azurecli
az vm update \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --zone "<target-zone>"
```

The command force deallocates the VM, updates its availability zone, and starts the VM in the target zone. The command waits for each operation to finish. If the zone update fails, Azure CLI attempts to restart the VM before it returns the update error.

> [!NOTE]
> Don't deallocate the VM separately. Azure CLI handles deallocation as part of the zone update.

# [Azure PowerShell](#tab/azure-powershell-recover)

Azure PowerShell doesn't provide a single cmdlet that orchestrates zone failover. Run the following cmdlets in order, and wait for each cmdlet to finish before you continue:

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
> Unlike the Azure CLI failover command, this PowerShell sequence isn't transactional. If `Update-AzVM` fails, review the error and the VM state before you run `Start-AzVM`.

# [Bicep](#tab/bicep-recover)

Bicep updates the VM's zone placement, but doesn't deallocate or start the VM. Run the following steps in order, waiting for each command to finish before continuing.

1. Force-deallocate the existing VM.

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

    resource vm 'Microsoft.Compute/virtualMachines@2026-03-01' = {
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

    This example is an excerpt, not a deployable VM definition. Keep the existing VM's full configuration, disk attachments, and network interfaces in your Bicep file. Don't reuse a new-VM template with `osDisk.createOption: 'FromImage'` to update an existing VM.

    Deploy the complete file after force-deallocation succeeds.

    ```azurecli
    az deployment group create \
      --resource-group "<resource-group-name>" \
      --template-file "./move-vm-zone.bicep" \
      --parameters vmName="<vm-name>" targetZone="<target-zone>"
    ```

1. After the deployment succeeds and the VM reports the target zone, start it.

    ```azurecli
    az vm start \
      --resource-group "<resource-group-name>" \
      --name "<vm-name>"
    ```

If the deployment fails, inspect the error and VM state before starting the VM. Verify the disks, network configuration, and workload before restoring traffic.

---

## Verify failover succeeded

After failover finishes, verify the VM and the workload before you restore traffic:

1. Confirm that the VM power state is **Running**.
1. Confirm that the VM reports the selected target availability zone.
1. Confirm that the expected OS and data disks are attached.
1. Confirm that the NIC, subnet, network security group, load balancer, and public IP associations are correct for your configuration.
1. Confirm that VM extensions and management settings are present and healthy.
1. Connect to the VM and verify operating system and application health.
1. Run workload-specific data consistency checks.
1. Restore traffic only after your health checks succeed.

Use the Azure portal, Azure CLI, or Azure PowerShell to check the VM state and zone:

# [Azure portal](#tab/azure-portal-verify)

1. In the [Azure portal](https://portal.azure.com/), open the VM, and select **Overview**.
1. Confirm that **Status** is **Running**.
1. Under **Availability + scaling**, confirm that **Availability zone** is the target zone and **Zone Resilience (Preview)** is **Enabled**.

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

## Troubleshoot failover

Match the error identifier or its message to the operation you attempted:

| Error identifier | When it occurs | What to do |
| --- | --- | --- |
| `ZoneMovementUpdateRequiresDeallocatedVM` | Enabling or disabling zone movement on a VM that isn't deallocated. | Deallocate the VM, retry the enable or disable update, and start the VM after it succeeds. |
| `OperationNotAllowedForNonZRSDisks` | The OS disk or an attached data disk doesn't use supported ZRS storage. | Use Premium SSD ZRS or Standard SSD ZRS for every disk before enabling zone movement. |
| `ForceDeallocateVMNotAllowedForNonResilientVM` | Force deallocation was requested for a VM without zone movement enabled. | If the source zone is healthy, deallocate normally and enable zone movement before retrying. If the source zone is unavailable, contact Azure support. |
| `OperationNotAllowedOnNullCurrentPowerStateForZonePatch` | A zone update was attempted before the affected VM reached **Deallocated**. | Force-deallocate the enabled VM, confirm it reaches **Deallocated**, and retry the zone update. |
| `OperationNotAllowedForNonZonalVM` | A zone update was requested for a regional VM without a pinned zone. | Confirm that the VM is zonal. This zone-update workflow applies only to zonal VMs. |
| `VMZoneMigrationNotCompleted` | The VM was started before zone migration finished or after migration failed. | Wait for the zone update to succeed, then retry the start. If the error persists, open an Azure support request. |

If the VM can't start because the target zone lacks capacity, select another supported zone. If the VM starts but the workload is unhealthy, complete the checks in [Verify failover succeeded](#verify-failover-succeeded) before restoring traffic. For other errors, review Azure service health and the operation details. If failover remains blocked, open an Azure support request.

## Disable zone movement

You must deallocate the VM before you disable zone movement. Deallocation interrupts the workload. Complete each operation before continuing, and start the VM only after the update succeeds:

# [Azure portal](#tab/azure-portal-disable)

1. In the [Azure portal](https://portal.azure.com/), open the VM, and select **Overview**.
1. Select **Stop**, confirm the operation, and wait until the VM status is **Stopped (deallocated)**.
1. Under **Availability + scaling**, select **Enabled** next to **Zone Resilience (Preview)**.
1. In the **Zone resilience (Preview)** pane, clear **Enable Zone resilience (Preview)**, and select **Save**.
1. After the update succeeds, confirm that **Zone Resilience (Preview)** shows **Disabled**, and select **Start**.

# [Azure CLI](#tab/azure-cli-disable)

```azurecli
az vm deallocate \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"
```

Deallocate the VM, and then disable zone movement:

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
resource vm 'Microsoft.Compute/virtualMachines@2026-03-01' = {
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

When the VM is deallocated, deploy the complete template from a local file named `disable-zone-movement.bicep`:

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

## Frequently asked questions

### Does failover happen automatically or keep the workload running?

No. You must initiate failover for a VM with zone resilience enabled. The workload is unavailable while the VM is force-deallocated and started in the target zone. Zone resilience doesn't provide automatic failover, uninterrupted service, or application-level recovery orchestration. For high availability across zones, [deploy multiple VMs across availability zones](disks-high-availability.md#deploy-vms-and-disks-across-three-availability-zones) and distribute traffic between them.

### What data does the near-zero RPO cover?

The near-zero RPO applies to data committed to supported ZRS disks. It doesn't apply to active requests, in-memory data, temporary disks, or application state that isn't written to those disks.

### When should I use zone resilience instead of Azure Site Recovery for zonal failover?

Both options support customer-initiated failover to another zone in the same region, but they use different disk and VM recovery models:

Don't use zone resilience to fail over production VMs during a zonal outage. This is a temporary preview limitation; support for this scenario is planned for a future update. See [Limitations](#limitations).

| Decision factor | Zone resilience for Azure VMs (Preview) | Azure Site Recovery zone-to-zone disaster recovery |
| --- | --- | --- |
| Managed disks | Requires Premium SSD ZRS or Standard SSD ZRS for the OS disk and every data disk. Data committed to these disks has a near-zero RPO. | Supports eligible LRS disks through replication. ZRS-managed disks aren't supported for zone-to-zone disaster recovery. There's no defined RPO SLA. |
| Failover model | Changes the zone of the existing VM and reuses its supported disks and configuration. | Replicates the VM before an outage and creates a recovery VM in a different resource group during failover. Supports failback. |
| Choose this option when | You need to retain the existing VM resource and its supported configuration, and you can use ZRS disks. | You need zone-to-zone disaster recovery for a VM with supported LRS disks in a supported region, and you can set up replication in advance. |

For supported regions, disk restrictions, and networking behavior for Azure Site Recovery, see [Enable Azure VM disaster recovery between availability zones](/azure/site-recovery/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery) and the [Azure Site Recovery support matrix](/azure/site-recovery/azure-to-azure-support-matrix#replicated-machines-storage).

### Which advanced capabilities aren't supported?

Zone resilience doesn't provide application-level consistency, traffic failover, recovery plans, or failback orchestration. Azure Site Recovery, on-demand capacity reservations, dedicated hosts, and proximity placement groups aren't supported for VMs with zone resilience enabled. Review [Limitations](#limitations) before you enable the feature.

## Cost

Standard charges apply for VMs, managed disks, and network resources. For managed disk pricing, see [Azure managed disks pricing](https://azure.microsoft.com/pricing/details/managed-disks/).

## Related content

- [Reliability in Azure Virtual Machines](/azure/reliability/reliability-virtual-machines)
- [Availability options for Azure Virtual Machines](availability.md)
- [Reliability in Azure Disk Storage](/azure/reliability/reliability-storage-disk)
- [Azure Site Recovery overview](/azure/site-recovery/site-recovery-overview)