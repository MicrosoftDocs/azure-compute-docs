---
title: Apply a custom remediation to an Azure VM (Preview)
description: Pre-stage a gallery remediation on a healthy Azure VM, submit one on demand in WinRE, or run inline diagnostics and interpret the results.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Apply a custom remediation to an Azure VM (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

Pre-stage a custom remediation while Windows is healthy, or submit it on demand in Windows Recovery Environment (WinRE). You can also run [inline PowerShell diagnostics](#run-inline-diagnostics).

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Review the [preview access and scope](overview.md#preview-access-and-scope) and [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) before you begin. Test on disposable, nonproduction systems with backups and a rollback plan. Prepare or confirm WinRE entry separately from submitting a remediation.

## Prerequisites

- Verify that the target VM is in a [supported region](overview.md#supported-regions).
- Complete [remote recovery setup](deploy-vm.md) on a supported VM.
- For gallery-backed remediations, [author and test](author-remediation.md) and [publish](publish-remediation.md) the remediation. Wait for replication in the VM's region.
- Obtain the [VM write, recovery-command write, and gallery read permissions](overview.md#permissions) for the operations you perform.
- Coordinate with other VM updates and repairs.
- Install PowerShell 7.4 or later and [Azure CLI](/cli/azure/install-azure-cli) on your management workstation. Sign in with Microsoft Entra credentials or an authorized managed identity.

## Pre-stage a remediation

With Windows running and the Azure VM Agent healthy, add the gallery script version ID to `properties.recoveryProfile.wellKnownGalleryScriptRemediations`. The extension downloads the remediation for a future WinRE session.

[Update the VM](/rest/api/compute/virtual-machines/update?view=rest-compute-2025-11-01&preserve-view=true) by using API version `2025-11-01`. Replace the gallery resource identifiers in this properties fragment with your values:

```json
{
  "properties": {
    "recoveryProfile": {
      "enableWindowsRecovery": true,
      "wellKnownGalleryScriptRemediations": [
        "/subscriptions/<gallery-subscription-id>/resourceGroups/<gallery-resource-group>/providers/Microsoft.Compute/galleries/<gallery-name>/scripts/<script-name>/versions/1.0.0"
      ]
    }
  }
}
```

Include every remediation you want to keep, with one version per script. Omitting the list preserves it. An empty array clears it.

Wait for the update, and verify that the extension's download completed. In WinRE, the plugin runs detection, and Windows Startup Repair (SRT) selects applicable remediations. Pre-staged remediations are available for detection without an additional recovery command. See [Interpret results](monitor-recovery.md#interpret-results) for assignment, staging, and repair signals.

### Package location and file names

The system stores packages under `<extension-version>\Downloads\KnownRemediations`. Each folder uses the name `<gallery-script-name>_<32-character-GUID>` without GUID hyphens.

The following example shows a standalone script published as `ContosoBootRepair`:

```text
C:\Packages\Plugins\Microsoft.CPlat.Core.WindowsRecoveryExtension\
  <extension-version>\
    Downloads\KnownRemediations\
      ContosoBootRepair_7d9e2c1a4b8f430e9a6d51c2f0b37864\
        ContosoBootRepair_7d9e2c1a4b8f430e9a6d51c2f0b37864.ps1
```

`<extension-version>` isn't the gallery script version. The drive letter and handler folder can differ from this example.

| Uploaded artifact | Result in the generated package folder |
| --- | --- |
| Standalone `.ps1` | Renamed to `<generated-package-name>.ps1`. The script contents are unchanged. |
| Standalone `.exe` | Renamed to `<generated-package-name>.exe`. The executable contents are unchanged. |
| `.zip` | Extracted with file names and folder structure preserved. The system removes the downloaded ZIP. |

The system detects file types from content. To change a remediation, publish a new version and update the profile. Don't edit staged files.

## Submit an on-demand remediation

Create a Recovery Run Command with `isRecoveryRunCommand: true` on a prepared VM [running in WinRE](monitor-recovery.md#identify-the-reported-operating-environment). Keep the VM allocated and running.

Confirm WinRE entry from the reported OS name and [Boot Diagnostics](../../boot-diagnostics.md).

### Submit a gallery remediation

On your management workstation, save the following body as `recovery-command.json`. Replace `<vm-region>` with the VM's region and the gallery identifiers with your published version ID:

```json
{
  "location": "<vm-region>",
  "properties": {
    "source": {
      "galleryScriptReferenceId": "/subscriptions/<gallery-subscription-id>/resourceGroups/<gallery-resource-group>/providers/Microsoft.Compute/galleries/<gallery-name>/scripts/<script-name>/versions/1.0.0"
    },
    "isRecoveryRunCommand": true
  }
}
```

In PowerShell on your management workstation, replace `<vm-subscription-id>`, `<vm-resource-group>`, `<vm-name>`, and `<command-name>` with your values. Run the following command from the request file's directory:

```powershell
az rest --method put `
    --url "https://management.azure.com/subscriptions/<vm-subscription-id>/resourceGroups/<vm-resource-group>/providers/Microsoft.Compute/virtualMachines/<vm-name>/runCommands/<command-name>?api-version=2025-11-01" `
    --body '@.\recovery-command.json' `
    --query '{id:id,provisioningState:properties.provisioningState}' --output json
```

Use this [managed run-command operation](/rest/api/compute/virtual-machine-run-commands/create-or-update?view=rest-compute-2025-11-01&preserve-view=true), not `az vm run-command invoke`. The `installRunCommandAsService` and `runImmediately` properties aren't required.

The agent stages the remediation for plugin detection and SRT selection. Don't pass `Detect`, `Remediate`, or a context object as command parameters.

Save the command ID and submission time, and then [monitor that request](monitor-recovery.md#check-recovery-command-status). Refresh that request's status rather than submitting duplicates. Coordinate cancellation and staged-file cleanup before deleting a command.

### Run inline diagnostics

Recovery Run Command supports quick tests and diagnostics through `properties.source.script`. The agent runs the PowerShell script directly in WinRE. It doesn't stage a remediation for the plugin or use the `-Detect` and `-Remediate` contract. No gallery version is required.

The extension supplies the PowerShell runtime; it doesn't use PowerShell from the WinRE partition. `WindowsRecoveryExtension` version 1.0.12 packages PowerShell 7.4.22. Test diagnostic scripts and their dependencies against the runtime packaged with your VM's extension version. For details, see the [remediation invocation contract](author-remediation.md#supported-payloads-and-invocation).

For example, the following read-only script prints the running Windows directory and lists file system drives:

```json
{
  "location": "<vm-region>",
  "properties": {
    "source": {
      "script": "$ErrorActionPreference = 'Stop'; Write-Output ('SystemRoot: ' + $env:SystemRoot); Get-PSDrive -PSProvider FileSystem | Select-Object Name, Root"
    },
    "isRecoveryRunCommand": true
  }
}
```

Save this body as `recovery-command.json`, and use the same `az rest` command with a name such as `inspect-winre`. Use `source.script` instead of `source.galleryScriptReferenceId`, not both.

Read `executionState`, `exitCode`, `output`, and `error` in the command's [instance-view substatus](monitor-recovery.md#check-recovery-command-status). In this example, `$env:SystemRoot` identifies WinRE, not the offline Windows installation. Normal Windows services and cmdlets might be unavailable. Keep diagnostic scripts read-only, and don't include secrets in scripts or output.

## Verify the outcome

Verify Windows startup and workload health separately from command success. For gallery-backed remediations, also check the [plugin's repair results](monitor-recovery.md#check-remediation-results). Let SRT manage reboot. Don't restart the VM solely because a command succeeded.

For inline diagnostics, inspect the script's output and errors. Use the [results table](monitor-recovery.md#interpret-results) to distinguish diagnostics from repair execution.

## Manage assignments and test resources

Before removing assignments, coordinate with other VM updates and recovery operations, including any in-progress repair.

Preserve required diagnostics before removing unneeded test VMs, disks, snapshots, or storage. Don't delete shared resources or gallery versions that other VMs still reference.

## Next steps

- [Monitor recovery mode, commands, and repair results](monitor-recovery.md).
- [Review the detection and repair contract](author-remediation.md#remediation-contract).
