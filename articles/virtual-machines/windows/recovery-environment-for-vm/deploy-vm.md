---
title: Deploy an Azure VM with remote recovery enabled (Preview)
description: Enable remote recovery when you create a supported Azure virtual machine, verify recovery preparation, and troubleshoot setup issues.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Deploy an Azure VM with remote recovery enabled (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

Enable remote recovery when you create a supported Azure VM. Setup prepares Windows Recovery Environment (WinRE) while normal Windows is running. It resizes the existing OS partition to create a 1-GB recovery partition.

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Only new, single VMs are supported for onboarding. For preview terms, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Prerequisites

- Meet the [preview access and scope requirements](overview.md#preview-access-and-scope), and select a [supported region](overview.md#supported-regions).
- Confirm `Microsoft.Compute/WindowsRecovery` feature access and `Microsoft.Compute` provider registration for your subscription.
- Obtain the [VM deployment and instance-view permissions](overview.md#permissions), including access to the VM's dependencies.
- Enable VM Agent provisioning and extension operations. Verify Guest Agent and remediation-download connectivity.
- Install PowerShell 7.4 or later and [Azure CLI](/cli/azure/install-azure-cli) on your management workstation. Sign in with Microsoft Entra credentials or an authorized managed identity.
- Maintain backups and a rollback plan. Use a disposable, nonproduction VM for recovery tests.

## Select the image

Use `MicrosoftWindowsServer:WindowsServer:2025-datacenter-azure-edition:latest`. Set `properties.storageProfile.imageReference.version` to `latest` in the VM create request. Other variants aren't covered by this guide.

| Component | Value |
| --- | --- |
| Publisher | `MicrosoftWindowsServer` |
| Offer | `WindowsServer` |
| SKU | `2025-datacenter-azure-edition` |
| Version | `latest` |

## Create the VM with recovery enabled

Use `az rest` with the VM API version `2025-11-01`. Set `properties.recoveryProfile.enableWindowsRecovery` to `true` in the VM request body:

```json
{
  "properties": {
    "storageProfile": {
      "imageReference": {
        "publisher": "MicrosoftWindowsServer",
        "offer": "WindowsServer",
        "sku": "2025-datacenter-azure-edition",
        "version": "latest"
      }
    },
    "recoveryProfile": {
      "enableWindowsRecovery": true
    }
  }
}
```

Add this fragment to a complete [VM create request](/rest/api/compute/virtual-machines/create-or-update?view=rest-compute-2025-11-01&preserve-view=true), and save it as `vm.json`. Include the location, size, networking, and other required properties. The file must contain a VM request body, not an ARM deployment template. The fragment alone isn't a complete deployment request.

In PowerShell on your management workstation, replace `<subscription-id>`, `<resource-group>`, and `<vm-name>` with your values. Run the following command from the directory that contains `vm.json`:

```powershell
az rest --method put `
    --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/<vm-name>?api-version=2025-11-01" `
    --body '@.\vm.json' `
    --query '{id:id,provisioningState:properties.provisioningState}' --output json
```

Azure manages `WindowsRecoveryExtension`. Don't install or modify it manually.

## Verify preparation

On your management workstation, check that `WindowsRecoveryExtension` reports successful provisioning:

```powershell
az vm get-instance-view --subscription '<subscription-id>' `
    --resource-group '<resource-group>' --name '<vm-name>' `
    --query "instanceView.extensions[?name=='WindowsRecoveryExtension'].{name:name,provisioningState:statuses[0].code}" `
    --output table
```

Expected output:

```text
Name                      ProvisioningState
------------------------  ---------------------------
WindowsRecoveryExtension  ProvisioningState/succeeded
```

If no row appears, check deployment and Guest Agent status. Wait for in-progress provisioning to finish. For failures, [inspect the extension status messages](monitor-recovery.md).

Successful provisioning confirms preparation. For runtime state and repair results, see [Interpret results](monitor-recovery.md#interpret-results).

## Configure and verify QMR

QMR cloud remediation is disabled by default on Windows Server 2025. Verify the effective configuration after deployment before relying on Microsoft remediations. QMR requires Windows Server 2025 with the October 2026 security update or later.

1. Inside the VM, open an elevated command prompt and inspect the recovery configuration:

   ```cmd
   reagentc.exe /info
   reagentc.exe /getrecoverysettings
   ```

1. Confirm that WinRE is enabled. To use QMR, also verify that cloud remediation and the intended automatic-remediation settings are enabled.
1. If QMR isn't enabled, [configure QMR through Group Policy](/troubleshoot/windows-server/group-policy/configure-quick-machine-recovery-via-group-policy) or [apply recovery settings with REAgentC](/windows/configuration/quick-machine-recovery/#configuration).
1. Run `reagentc.exe /getrecoverysettings` again to verify the effective settings. Confirm Windows Update connectivity from WinRE.

Keep QMR settings and custom-remediation assignments separate when planning recovery. For the meaning of provisioning and runtime signals, see [Interpret results](monitor-recovery.md#interpret-results).

## Troubleshoot preparation

Use the following checks if deployment or extension preparation fails.

| Symptom | Action |
| --- | --- |
| The image or version is rejected. | Confirm the supported image version and region. |
| Feature access is rejected. | Check feature access, provider registration, and region support. |
| The Guest Agent is unhealthy or extension operations are disabled. | Restore Guest Agent health, and enable extension operations. |
| The recovery flag is true, but the extension is missing or failed. | Inspect the deployment and extension messages. |
| Recovery setup fails. | Check extension logs, disk layout, and available space. |

## Disable recovery

To disable recovery, set `properties.recoveryProfile.enableWindowsRecovery` to `false` in the VM configuration. Azure removes the extension but doesn't reclaim recovery-partition space. Disabling recovery isn't a substitute for restoring a failed workload.

Retain required diagnostics before deleting test resources. Remove unneeded test VMs, disks, snapshots, and storage to avoid ongoing charges. Don't delete shared resources.

## Next steps

- [Author and package a remediation](author-remediation.md).
- [Publish a gallery script version](publish-remediation.md).
- [Pre-stage or apply a remediation](apply-remediation.md).
- [Monitor recovery mode and remediation progress](monitor-recovery.md).

## Related content

- [Azure CLI VM image reference](/cli/azure/vm/image).
- [Azure Windows VM Agent](../../extensions/agent-windows.md).
- [REAgentC command-line options](/windows-hardware/manufacture/desktop/reagentc-command-line-options).
