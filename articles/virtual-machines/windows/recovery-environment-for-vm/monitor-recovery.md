---
title: Monitor remote recovery for Azure VMs (Preview)
description: Use Azure VM instance view to identify WinRE, inspect recovery command and remediation results, and verify Windows startup and workload health.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Monitor remote recovery for Azure VMs (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

Use VM instance view to check Windows Recovery Environment (WinRE) mode, command status, and repair results. The Azure queries in this article are read-only. To submit a remediation, see [Apply a custom remediation](apply-remediation.md).

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Review the [preview access and scope](overview.md#preview-access-and-scope) and [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Prerequisites

- Verify that the target VM is in a [supported region](overview.md#supported-regions).
- Complete [remote recovery setup](deploy-vm.md).
- Obtain [VM and instance-view read permissions](overview.md#permissions).
- Install PowerShell 7.4 or later and [Azure CLI](/cli/azure/install-azure-cli) on your management workstation.
- Keep the VM identifiers and, for on-demand recovery, the command name and submission time.

## Retrieve the instance view

Sign in to Azure CLI. On your management workstation, replace the placeholders and retrieve the VM's OS identity, agent status, and recovery reports:

```powershell
az vm get-instance-view --subscription '<subscription-id>' `
    --resource-group '<resource-group>' --name '<vm-name>' `
    --query "instanceView.{osName:osName,osVersion:osVersion,vmAgent:vmAgent,statuses:statuses,recovery:extensions[?name=='WindowsRecoveryExtension']}" `
    --output json
```

## Interpret results

Use the following table to distinguish preparation, staging, execution, and recovery. Match report timestamps to the incident or command you're investigating.

| Signal | Meaning and next check |
| --- | --- |
| `recoveryProfile.enableWindowsRecovery: true` | Recovery is configured. Check instance view for runtime state. Enabling recovery or submitting a remediation doesn't start WinRE. |
| VM or extension `ProvisioningState/succeeded` | Provisioning completed. Check command and plugin reports for staging and repair results. |
| `PowerState/running` | The VM is running at the platform level, including while in WinRE. Check `osName` for the operating environment. |
| Guest Agent unavailable or failed | Investigate agent health and Boot Diagnostics. An agent failure alone doesn't identify WinRE. |
| `osName: Windows PreInstallation Environment` | The VM reports WinRE. WinRE is built on Windows PE. |
| Gallery-backed command `executionState: Succeeded` and `exitCode: 0` | Download and staging completed. Check the plugin report for repair execution. |
| Inline diagnostic command `executionState: Succeeded` and `exitCode: 0` | The diagnostic script completed. Read its output and errors; it uses a separate execution path from plugin repairs. |
| `detect.exitCode: 0` | A remediation is applicable and eligible for selection. Check its repair result to see whether it ran. |
| `remediate.outcome: Succeeded` and `exitCode: 0` | The repair process reports success. Verify Windows startup and workload health separately. |
| `Failed`, `TimedOut`, or `LaunchFailed` | Investigate the reported failure and preserve diagnostics. |
| `Skipped`, `NotRun`, or missing execution data | Inspect the raw report and refresh status before determining the outcome. |
| `WindowsRecoverySummary` with state `succeeded` | Read the summary message. This state can also mean that no successful remediation was recorded. |
| `AgentStartup_BootFailureDetected/succeeded` or `FatalErrorCode: 0` | Boot-failure data was reported. Use it for diagnosis, not as confirmation of a healthy boot. |
| Deleted command, cleared assignment, or a monitoring timeout | Verify the operation's actual state. These actions don't confirm cancellation or staged-file cleanup. |

## Prepare detailed queries

For the detailed PowerShell examples, retrieve instance view into `$view` and define the [PowerShell status helpers](#locate-recovery-status-entries). Use the same session for subsequent examples.

```powershell
$ErrorActionPreference = 'Stop'
$PSNativeCommandUseErrorActionPreference = $true

$subscriptionId = '<subscription-id>'
$resourceGroup = 'rg-workload'
$vmName = 'myWindowsVM'

$view = az vm get-instance-view --subscription $subscriptionId `
    --resource-group $resourceGroup --name $vmName `
    --query instanceView --output json | ConvertFrom-Json
```

JSON examples show selected fields from the current preview report schema. To refresh the data, rerun the retrieval command.

## Identify the reported operating environment

The value `osName: Windows PreInstallation Environment` indicates the reported WinRE mode. WinRE is built on Windows Preinstallation Environment (Windows PE), which explains this OS name. Use [Boot Diagnostics](../../boot-diagnostics.md) to corroborate the reported state.

Example in WinRE:

```json
{
  "osName": "Windows PreInstallation Environment",
  "osVersion": "10.0.26100.7519",
  "statuses": [
    {
      "code": "PowerState/running",
      "displayStatus": "VM running"
    }
  ]
}
```

## Locate recovery status entries

Find `WindowsRecoveryExtension` in `extensions[]`. Substatus codes use `ComponentStatus/<subcomponent>/<state>[/<error-code>]`.

The detailed examples use the following helpers. Expand the section and define them in the same PowerShell session.

<details>
<summary>PowerShell status helpers</summary>

```powershell
function Get-WindowsRecoveryExtension {
    param([object] $InstanceView)

    $entries = @($InstanceView.extensions |
        Where-Object { $_.name -eq 'WindowsRecoveryExtension' })
    if ($entries.Count -ne 1) {
        throw 'A unique recovery extension report is unavailable. Check preparation and boot diagnostics; the outcome is unknown.'
    }
    return $entries[0]
}

function Get-RecoverySubstatus {
    param(
        [object] $Extension,
        [string[]] $Names
    )

    $entries = @($Extension.substatuses | Where-Object {
        $parts = $_.code -split '/'
        $parts.Count -ge 3 -and $parts[0] -eq 'ComponentStatus' -and
            $Names -contains $parts[1]
    })
    if ($entries.Count -ne 1) {
        throw "Expected one report for [$($Names -join ', ')]; found $($entries.Count). Inspect raw substatuses; missing or ambiguous status is not success."
    }
    return $entries[0]
}

$recovery = Get-WindowsRecoveryExtension -InstanceView $view
$recovery.statuses | Format-List code, level, message
$recovery.substatuses | Format-List code, level, message
```

</details>

The following example shows a typical layout. Substatus messages are omitted, and state values are placeholders:

```json
{
  "extensions": [
    {
      "name": "WindowsRecoveryExtension",
      "statuses": [
        {
          "code": "ProvisioningState/succeeded",
          "level": "Info",
          "message": "Windows Recovery Extension enabled successfully."
        }
      ],
      "substatuses": [
        {
          "code": "ComponentStatus/AgentStartup_BootFailureDetected/<state>"
        },
        {
          "code": "ComponentStatus/AgentStartup_BootDiagnosticOutput/<state>"
        },
        {
          "code": "ComponentStatus/<command-name>_RecoveryAgentStatus/<state>"
        },
        {
          "code": "ComponentStatus/RecoveryPluginStatus/<state>"
        },
        {
          "code": "ComponentStatus/WindowsRecoverySummary/<state>"
        }
      ]
    }
  ]
}
```

The `statuses` entries describe extension provisioning. The `substatuses` entries contain diagnostics and results. Read each report when it appears; WinRE reports and the post-boot summary represent different stages. The `<command-name>` value is your on-demand command name.

This article uses the current preview report names:

| Purpose | Report name |
| --- | --- |
| Boot-failure data | `AgentStartup_BootFailureDetected` |
| Additional boot diagnostics | `AgentStartup_BootDiagnosticOutput` |
| Recovery command status | `<command-name>_RecoveryAgentStatus` |
| Remediation results | `RecoveryPluginStatus`, with a `remediations` array. |
| Post-recovery summary | `WindowsRecoverySummary` |

## Read boot-failure data

Inspect the failure before authoring a remediation:

```powershell
$recovery = Get-WindowsRecoveryExtension -InstanceView $view
$bootFailure = Get-RecoverySubstatus -Extension $recovery `
    -Names @('AgentStartup_BootFailureDetected')
$bootFailure | Format-List code, level, message
```

Example entry in `WindowsRecoveryExtension.substatuses`:

```json
{
  "code": "ComponentStatus/AgentStartup_BootFailureDetected/succeeded",
  "level": "Info",
  "message": "Boot failure detected: OSStarted: 1\r\nFatalErrorCode: 0\r\n"
}
```

The message is labeled text, not JSON. Additional output is available in `AgentStartup_BootDiagnosticOutput` when reported. See [Interpret results](#interpret-results) before drawing conclusions from the status code.

## Check recovery command status

Interpret the command result according to its source:

- **Gallery-backed remediation:** Reports download and staging, not repair execution.
- **Inline PowerShell script:** Reports script execution, including stdout in `output` and stderr in `error`.

Replace `recovery-contoso` with your command name, such as `inspect-winre` for the [inline diagnostics example](apply-remediation.md#run-inline-diagnostics). For pre-staged remediations, skip to [repair results](#check-remediation-results).

```powershell
$runCommandName = 'recovery-contoso'
$recovery = Get-WindowsRecoveryExtension -InstanceView $view
$commandEntry = Get-RecoverySubstatus -Extension $recovery `
    -Names @("${runCommandName}_RecoveryAgentStatus")

$commandEntry | Format-List code, level, message
if ([string]::IsNullOrWhiteSpace($commandEntry.message)) {
    throw 'The command has no reported message yet; its output and result are unavailable.'
}

if ($commandEntry.message.TrimStart().StartsWith('{')) {
    $commandResult = $commandEntry.message | ConvertFrom-Json -ErrorAction Stop
    if (-not ($commandResult.PSObject.Properties.Name -contains 'executionState')) {
        throw 'This is not a recognized command result. Inspect the raw message.'
    }
    $commandResult | Format-List executionState, exitCode, executionMessage,
        startTime, endTime, output, error
}
else {
    Write-Warning 'This is plain-text progress, not a structured execution result. Do not infer an exit code or stdout.'
}
```

Example after `recovery-contoso` stages one file:

```json
{
  "code": "ComponentStatus/recovery-contoso_RecoveryAgentStatus/succeeded",
  "level": "Info",
  "message": "{\"executionMessage\":\"Successfully downloaded and staged remediation artifact for run command 'recovery-contoso'. ExtractedFileCount=1.\",\"exitCode\":0,\"executionState\":\"Succeeded\"}"
}
```

Parse `message` to read the command result. Here, `Succeeded` and `exitCode: 0` confirm staging. Match timestamps to your request.

For inline diagnostics, the same fields describe the script's execution, and the query displays its `output` and `error`.

The run-command resource's separate instance view can remain `Pending`. Don't resubmit solely because it differs from this report.

## Check remediation results

The plugin report shows detection and repair results for both pre-staged and on-demand remediations. The following expandable query parses the current report schema and displays each remediation's result.

<details>
<summary>Parse the plugin report with PowerShell</summary>

```powershell
$recovery = Get-WindowsRecoveryExtension -InstanceView $view
$pluginEntry = Get-RecoverySubstatus -Extension $recovery `
    -Names @('RecoveryPluginStatus')

$pluginEntry | Format-List code, level, message
if ([string]::IsNullOrWhiteSpace($pluginEntry.message) -or
    -not $pluginEntry.message.TrimStart().StartsWith('{')) {
    throw 'Structured plugin progress is unavailable. Read the message and refresh later; do not assume completion.'
}

$plugin = $pluginEntry.message | ConvertFrom-Json -ErrorAction Stop
$plugin | Select-Object timestamp, entryIntoWindowsRecovery,
    version, recoveryAgentVersion, truncated, droppedRemediations

if ($null -ne $plugin.phases) {
    $plugin.phases.PSObject.Properties | ForEach-Object {
        [pscustomobject]@{
            Phase = $_.Name
            Status = $_.Value.status
            Message = $_.Value.message
        }
    } | Format-Table -Wrap
}
else {
    Write-Warning 'No phase snapshot is available. Inspect the raw message.'
}

if ($plugin.PSObject.Properties.Name -contains 'remediations') {
    $plugin.remediations | ForEach-Object {
        [pscustomobject]@{
            Name = $_.displayName
            File = $_.fileName
            Source = $_.source
            DetectExitCode = $_.detect.exitCode
            RepairOutcome = $_.remediate.outcome
            RepairExitCode = $_.remediate.exitCode
            RepairStartTime = $_.remediate.startTime
            RepairDurationMs = $_.remediate.durationMs
        }
    } | Format-List
}
else {
    Write-Warning 'No per-remediation results are available.'
}

if ($plugin.truncated -eq $true -or $plugin.droppedRemediations -gt 0) {
    Write-Warning 'The report is incomplete. Missing records do not establish that a repair never ran.'
}
```

</details>

The following decoded `message` illustrates a successful repair in the `RecoveryPluginStatus` report:

```json
{
  "timestamp": "2026-06-27T02:34:22Z",
  "phases": {
    "RemediationExecution": {
      "status": "Success",
      "message": "Attempted 1 remediations: 1 succeeded, 0 failed"
    }
  },
  "remediations": [
    {
      "displayName": "Remediation.ps1",
      "fileName": "Remediation.ps1",
      "source": "WellKnown",
      "detect": {
        "exitCode": 0
      },
      "remediate": {
        "startTime": "2026-06-27T02:34:10Z",
        "durationMs": 12000,
        "exitCode": 0,
        "outcome": "Succeeded"
      }
    }
  ]
}
```

Match the name and timestamps to the intended attempt, not the positional `id` alone. Use the [results table](#interpret-results) to interpret detection and repair outcomes.

## Confirm return to Windows

After Windows boots, check for its normal OS name, a ready Guest Agent, and the extension's post-recovery summary.

```powershell
$view = az vm get-instance-view --subscription $subscriptionId `
    --resource-group $resourceGroup --name $vmName `
    --query instanceView --output json | ConvertFrom-Json
$recovery = Get-WindowsRecoveryExtension -InstanceView $view
$summaryEntry = Get-RecoverySubstatus -Extension $recovery `
    -Names @('WindowsRecoverySummary')

$view | Select-Object osName, osVersion
$view.vmAgent.statuses | Format-List code, displayStatus, message, time
$summaryEntry | Format-List code, level, message
```

The following example shows selected fields after Windows returns. The summary message is omitted from this example. Read your VM's actual message to determine what was reported.

```json
{
  "osName": "Windows Server 2025 Datacenter Azure Edition",
  "vmAgent": {
    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Ready"
      }
    ]
  },
  "extensions": [
    {
      "name": "WindowsRecoveryExtension",
      "substatuses": [
        {
          "code": "ComponentStatus/WindowsRecoverySummary/succeeded",
          "level": "Info"
        }
      ]
    }
  ]
}
```

Read the plain-text `WindowsRecoverySummary` message. Confirm Windows startup through [Boot Diagnostics](../../boot-diagnostics.md), guest access, and workload checks. Let Windows Startup Repair manage reboot.

## Handle missing or stale diagnostics

When reports are missing or incomplete, preserve evidence and avoid assuming an outcome:

- Refresh instance view periodically, and compare timestamps with the incident. A monitoring timeout doesn't cancel the operation.
- Preserve raw output for investigation. Missing, delayed, or truncated reports don't establish success or failure.
- Save required WinRE diagnostics before Windows returns. These reports can disappear after boot, and instance view isn't an audit log.

## Retain diagnostics safely

Redact customer data and credentials before sharing saved diagnostics. Retain them according to your incident policy. Preserve required evidence before cleaning up disposable test resources.

## Related content

- [Remote recovery overview and preview prerequisites](overview.md).
- [Deploy a VM with WinRE](deploy-vm.md).
- [Apply a custom remediation](apply-remediation.md).
- [VM instance-view REST API](/rest/api/compute/virtual-machines/instance-view?view=rest-compute-2025-11-01&preserve-view=true).
