---
title: Remote recovery for Azure VMs (Preview)
description: Learn how Windows Recovery Environment supports startup recovery on Azure virtual machines through Microsoft remediations and custom remediations.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: overview
ms.date: 11/09/2026
ms.custom: references_regions
ai-usage: ai-assisted
---

# Remote recovery for Azure VMs (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

[Windows Recovery Environment](/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference) (WinRE) runs separately from a virtual machine's Windows installation to diagnose and repair startup failures. On Azure virtual machines (VMs), WinRE supports two kinds of remediation: Microsoft remediations delivered through [quick machine recovery](/windows/configuration/quick-machine-recovery/) (QMR) for widespread outages, and custom remediations that you author and deliver through remote recovery for Azure VMs to fix other issues that prevent a VM from starting.

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Their eligibility and permissions are separate from QMR. For preview terms, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## When to use WinRE

Use WinRE to attempt in-place repairs for startup failures caused by drivers, updates, or configuration changes, without attaching the OS disk to another VM. Keep backups and a disaster-recovery plan for incidents that require restoring or replacing the VM.

## How it works

Prepare recovery while Windows and the Azure Guest Agent are healthy:

1. [Enable recovery on a supported VM](deploy-vm.md), and configure the remediation sources you need.
1. A startup failure occurs.
1. The VM enters WinRE.
1. Recovery plugins detect applicable remediations.
1. Windows Startup Repair selects and runs remediations.
1. The recovery environment manages the next boot attempt.
1. [Verify Windows startup and workload health](monitor-recovery.md).

The following components participate in recovery:

| Component | Purpose |
| --- | --- |
| Windows Startup Repair (SRT) | Coordinates detection and repair in WinRE. |
| Azure VM recovery plugin (`AzureVMRecoveryPlugin`) | Detects and runs custom remediations on the offline OS disk. |
| Boot Critical Updates (BCU) plugin | Retrieves Microsoft remediations from Windows Update for QMR. |

## Remediation sources

Microsoft remediations (QMR) and custom remediations are complementary sources. Both plugins can participate in the same WinRE session. Custom remediations have two delivery modes: pre-staged and on-demand.

| Source or delivery mode | Remediation source | How you use it |
| --- | --- | --- |
| Microsoft remediations (QMR) (Preview) | Windows Update. | Enable QMR cloud remediation, and download applicable fixes in WinRE. Windows Update connectivity is required. |
| Pre-staged custom remediations (Preview) | Azure Compute Gallery script version. | Download while Windows is healthy for use in a later WinRE session. |
| On-demand custom remediations (Preview) | Azure Compute Gallery script version. | Submit a recovery command to download the remediation while the VM is in WinRE. |

On Windows Server 2025, QMR cloud remediation is disabled by default. [Verify the effective QMR configuration after deployment](deploy-vm.md#configure-and-verify-qmr) before relying on Microsoft remediations.

Recovery Run Command also supports [inline PowerShell tests and diagnostics](apply-remediation.md#run-inline-diagnostics) in WinRE, with output available in instance view. Use the [QMR simulation](test-quick-machine-recovery.md) to test the Microsoft remediation path on an eligible Windows client VM. Test custom remediations separately on the supported Azure VM image.

## Preview access and scope

To use custom remediations through remote recovery for Azure VMs, meet these requirements:

- Use an active Azure subscription with `Microsoft.Compute/WindowsRecovery` feature access. [Feature registration](/azure/azure-resource-manager/management/preview-features) is separate from `Microsoft.Compute` provider registration.
- Deploy a new, single VM. The preview doesn't support existing-VM onboarding or VM scale sets.
- Use `MicrosoftWindowsServer:WindowsServer:2025-datacenter-azure-edition:latest`. This onboarding workflow doesn't support other editions, Server Core, small-disk, custom or gallery images, and ARM64.

### Supported regions

Remote recovery for Azure VMs preview is available in these regions. Verify that your subscription has access to the region before deployment.

| Region |
| --- |
| eastus2euap |
| eastasia |
| centraluseuap |
| uksouth |
| westcentralus |
| centralus |


### Management tools

Use the REST requests in this article set to configure remote recovery for Azure VMs. Submit them through Azure CLI `az rest` from PowerShell. The preview doesn't provide dedicated Azure CLI commands or native SDK support for configuring recovery.

See [Deploy a VM with WinRE](deploy-vm.md) and [Apply a remediation](apply-remediation.md) for examples.

## Prerequisites and limitations

Before you configure or test recovery, prepare the following resources and access:

- Use a supported region, a healthy [Azure Windows VM Agent](../../extensions/agent-windows.md), and enabled extension operations.
- Obtain the [permissions for each recovery operation](#permissions).
- Verify connectivity for the Guest Agent and remediation downloads. QMR also requires Windows Update access from WinRE.
- Maintain backups, a rollback plan, [Boot Diagnostics](../../boot-diagnostics.md), and guest-administration access. Test only on disposable, nonproduction systems.
- Setup resizes the existing OS partition to create a 1-GB recovery partition. [Disabling recovery](deploy-vm.md#disable-recovery) doesn't reclaim its space. VM, disk, snapshot, and storage charges still apply.

Windows Insider enrollment applies to the QMR simulation, not custom remediations. Review the simulation's [separate eligibility requirements](test-quick-machine-recovery.md#prerequisites) before testing.

### Permissions

Assign permissions at the resource scope required for each operation. VM deployment also requires permissions for its networking, disks, and other dependencies.

| Operation | Required Azure RBAC permissions or role |
| --- | --- |
| Enable recovery or change pre-staged assignments. | `Microsoft.Compute/virtualMachines/write` on the VM. |
| Submit an on-demand recovery command or inline diagnostics. | `Microsoft.Compute/virtualMachines/runCommands/write` on the VM. |
| Read VM configuration and recovery results. | `Microsoft.Compute/virtualMachines/read` and `Microsoft.Compute/virtualMachines/instanceView/read` on the VM. |
| Read a gallery script version for a remediation. | **Reader** on the gallery script version or a parent scope. |
| Publish gallery script definitions and versions. | **Contributor** on the gallery. Creating the gallery also requires permissions at its parent scope. |
| Upload the source blob and create a user-delegation SAS. | **Storage Blob Data Contributor** on the storage account. Creating a user-delegation key requires an assignment at the storage account or a parent scope. |

For details, see [Compute permissions](/azure/role-based-access-control/permissions/compute) and [permissions for a user-delegation SAS](/azure/storage/blobs/storage-blob-user-delegation-sas-create-cli#assign-permissions-with-azure-rbac). Guest-administration permissions are separate from Azure RBAC.

## Next steps

- [Deploy a VM with WinRE enabled](deploy-vm.md).
- [Author a remediation](author-remediation.md), [publish it](publish-remediation.md), and [pre-stage or apply it](apply-remediation.md).
- [Monitor recovery mode and results](monitor-recovery.md).
- [Test quick machine recovery](test-quick-machine-recovery.md).
