---
title: Zone-resilient virtual machines with ZRS disks
description: Learn how zone-resilient virtual machines use zone-redundant storage disks to support customer-initiated recovery in another availability zone.
author: ajkundna
ms.author: ajkundna
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 09/18/2026
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to recover an eligible zonal VM in another availability zone while preserving its resource configuration."
---

# Zone-resilient virtual machines with ZRS disks

Zone Resilient VM helps you recover an eligible zonal Azure virtual machine (VM) in another availability zone in the same region. The recovery process reuses supported zone-redundant storage (ZRS) managed disks and is designed to preserve the VM resource and its supported configuration. You don't have to create and configure an unrelated replacement VM.

Zone Resilient VM provides a customer-initiated recovery option for a single VM. It doesn't provide automatic failover, uninterrupted service, or application-level recovery orchestration. For higher availability, deploy multiple VM instances across availability zones and distribute traffic between them.

<!--
PUBLICATION BLOCKERS:
- Confirm the public feature name and release stage.
- Confirm the REST property names, API version, CLI parameters, PowerShell cmdlets, Bicep schema, and portal labels.
- Confirm the supported regions, VM sizes, disk SKUs, networking configurations, and cloud environments.
- Confirm the VM resource identity, private IP, MAC address, extension, Backup, and Site Recovery preservation contracts.
- Confirm feature-specific pricing and any recovery objective or service-level commitment.
Do not remove this comment until each item has an approved public source.
-->

## How zone recovery works

The recovery process has the following stages:

1. Deploy the VM in a specific availability zone and enable Zone Resilient VM.
1. Use supported ZRS managed disks for the persistent OS disk and every attached data disk.
1. Detect a failure that affects the VM, and select a healthy target zone in the same region.
1. Force-deallocate the VM to release or isolate its source-zone resources.
1. Update the VM placement to the target zone.
1. Start the VM in the target zone.
1. Verify the VM, its disks, its network configuration, and the workload.

The workload is unavailable while the VM is deallocated and started in the target zone. Active requests, in-memory data, and runtime state aren't preserved.

The target zone must support the VM size and have enough capacity when you start the VM. Without a matching [capacity reservation](capacity-reservation-overview.md), allocation is best effort.

## Requirements

Before you enable Zone Resilient VM, confirm that your configuration meets these requirements:

| Resource | Requirement |
| --- | --- |
| Region | The source and target zones must be in the same supported Azure region. |
| VM placement | The VM must be deployed in a specific availability zone. |
| VM size | The current VM size must be supported and available in the target zone. |
| OS disk | A persistent OS disk must use a supported ZRS managed disk SKU. |
| Data disks | Every attached data disk must use a supported ZRS managed disk SKU. |
| Network | The VM must use supported zone-redundant network resources. |

For current ZRS disk types and regional availability, see [Redundancy options for managed disks](disks-redundancy.md#zone-redundant-storage-for-managed-disks). To create ZRS disks, see [Deploy a managed disk that uses zone-redundant storage](disks-deploy-zrs.md).

<!-- PLACEHOLDER: Replace the general requirements with the approved support matrix before publication. -->

## Limitations

Zone Resilient VM has the following limitations:

- Recovery is customer initiated. Azure doesn't automatically move a standalone VM when a zone fails.
- Recovery causes downtime because the VM is force-deallocated and restarted.
- Recovery is limited to another availability zone in the same region. It doesn't protect against a regional outage or an outage that affects multiple zones.
- Locally redundant storage (LRS) OS and data disks aren't supported.
- Availability set VMs and Spot VMs aren't supported.
- Recovery doesn't preserve active requests, memory, or runtime state.
- Recovery can fail when the VM size has insufficient capacity in the target zone.
- The feature doesn't provide application-level consistency, traffic failover, recovery plans, or failback orchestration.

<!-- PLACEHOLDER: Confirm whether ephemeral OS disks, VM scale sets, dedicated hosts, proximity placement groups, hibernation, and InfiniBand are supported. -->

## Enable Zone Resilient VM

Enable Zone Resilient VM when you create an eligible zonal VM or update an eligible existing zonal VM. Azure validates the VM, disk, and network configuration when you enable the feature. After enablement, keep each dependency in a supported configuration.

> [!NOTE]
> The following procedures are placeholders. Replace them with steps generated from the approved portal experience, resource provider contract, and released command-line modules.

# [Azure portal](#tab/portal-enable)

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Create a zonal VM or open an eligible existing zonal VM.
1. Go to **[Placeholder: configuration page]**.
1. Set **[Placeholder: Zone Resilient VM setting]** to **Enabled**.
1. Review the eligibility validation results, and correct any unsupported dependencies.
1. Select **[Placeholder: Save or Create action]**.

# [Azure PowerShell](#tab/powershell-enable)

```azurepowershell
# PLACEHOLDER: Add the minimum Az.Compute version and approved cmdlets.
# Enable Zone Resilient VM on a new or existing eligible zonal VM.
```

# [Azure CLI](#tab/cli-enable)

```azurecli
# PLACEHOLDER: Add the minimum Azure CLI version and approved parameters.
# Enable Zone Resilient VM on a new or existing eligible zonal VM.
```

# [Bicep](#tab/bicep-enable)

```bicep
// PLACEHOLDER: Add the approved Microsoft.Compute API version and property schema.
// Enable Zone Resilient VM when you deploy an eligible zonal VM.
```

---

## Recover a VM in another zone

Use zone recovery after you confirm that an availability zone failure affects the VM. Monitor individual VM health with [Azure Resource Health](/azure/service-health/resource-health-overview), and monitor broader service issues with [Azure Service Health](/azure/service-health/overview).

Choose a target zone that supports your VM size. If you require stronger allocation assurance, associate an appropriate target-zone capacity reservation before an outage.

> [!WARNING]
> Force deallocation stops the VM and discards its in-memory state. Start recovery only after you confirm the impact on your workload and its dependencies.

# [Azure portal](#tab/portal-recover)

1. In the Azure portal, open the affected VM.
1. Select **[Placeholder: zone recovery action]**.
1. Select a healthy target availability zone.
1. Review the validation results and the expected downtime.
1. Select **[Placeholder: Recover action]**.
1. Monitor the operation until the VM is running in the target zone.

# [Azure PowerShell](#tab/powershell-recover)

```azurepowershell
# PLACEHOLDER: Add approved commands to force-deallocate the VM,
# update its target zone, start it, and monitor the operation.
```

# [Azure CLI](#tab/cli-recover)

```azurecli
# PLACEHOLDER: Add approved commands to force-deallocate the VM,
# update its target zone, start it, and monitor the operation.
```

# [Bicep](#tab/bicep-recover)

```bicep
// PLACEHOLDER: Document whether recovery is supported as a Bicep deployment.
// If recovery uses an action API, link to the approved operational procedure instead.
```

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

<!-- PLACEHOLDER: Add approved commands for each verification step and confirm the resource identity, private IP, Backup, and Site Recovery checks. -->

## Troubleshoot recovery

| Problem | Possible cause | Action |
| --- | --- | --- |
| You can't enable the feature. | A VM, disk, or network dependency isn't supported. | Review the validation error, update the named dependency, and try again. |
| The VM can't start in the target zone. | The VM size isn't available or the zone has insufficient capacity. | Select another supported zone or use a matching capacity reservation. |
| A disk isn't attached after recovery. | The disk doesn't use a supported ZRS SKU, or the disk operation didn't finish. | Confirm the disk SKU and operation status. Don't restore traffic until all required disks are attached. |
| The VM is running, but the workload is unhealthy. | A network, extension, guest configuration, or application dependency requires attention. | Validate each item in [Verify recovery succeeded](#verify-recovery-succeeded), and then follow the workload's recovery procedure. |
| Recovery doesn't complete. | A platform operation or required regional service is unavailable. | Review the operation error and Azure service health. If the problem continues, open an Azure support request. |

<!-- PLACEHOLDER: Replace general guidance with approved error codes, operation states, retry behavior, and support escalation criteria. -->

## Cost

Standard charges for VMs, managed disks, network resources, and capacity reservations apply. ZRS disks cost more than equivalent LRS disks because ZRS synchronously replicates data across availability zones.

<!-- PLACEHOLDER: Add approved Zone Resilient VM pricing or state that there is no additional feature charge. -->

## Related content

- [Reliability in Azure Virtual Machines](/azure/reliability/reliability-virtual-machines)
- [Availability options for Azure Virtual Machines](availability.md)
- [Reliability in Azure Disk Storage](/azure/reliability/reliability-storage-disk)
- [Azure Site Recovery overview](/azure/site-recovery/site-recovery-overview)