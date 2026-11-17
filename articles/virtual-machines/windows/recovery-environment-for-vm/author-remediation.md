---
title: Author a custom remediation for Azure VMs (Preview)
description: Learn the detection, repair, and packaging contract for custom remediations that run in WinRE on supported Azure virtual machines.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Author a custom remediation for Azure VMs (Preview)

**Applies to:** :heavy_check_mark: Windows VMs

A custom remediation for remote recovery runs in Windows Recovery Environment (WinRE) to detect and repair an issue in a VM's offline Windows installation. This article describes the invocation contract, an illustrative PowerShell script, and packaging requirements.

> [!IMPORTANT]
> Custom remediations through remote recovery for Azure VMs are in **preview**. Review the [preview access and scope](overview.md#preview-access-and-scope) and [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) before you begin. Don't develop or test a remediation on a production VM or its original OS disk.

## Prerequisites

- [Prepare a test VM](deploy-vm.md) in a [supported region](overview.md#supported-regions).
- Obtain [instance-view read permissions](overview.md#permissions) and administrator access to the test VM.
- [Identify the boot failure](monitor-recovery.md#read-boot-failure-data), target Windows version and architecture, and how to verify the repair.
- Use a disposable VM or disk copy with backups and a rollback plan.
- Verify that your dependencies run in WinRE. Normal Windows services, profiles, and networking might be unavailable.

## Remediation contract

The [recovery components](overview.md#how-it-works) use Windows Startup Repair (SRT) to coordinate plugins and supply boot-failure information:

- **Azure VM recovery plugin** (`AzureVMRecoveryPlugin`) scans the offline OS disk for custom remediations. The extension pre-stages files in `<extension-version>\Downloads\KnownRemediations` during normal Windows operation. The recovery agent stages on-demand files in `CustomRemediations` during WinRE. See [delivery options and file locations](apply-remediation.md).
- **Boot Critical Updates (BCU) plugin** downloads Microsoft remediations from Windows Update through [quick machine recovery](/windows/configuration/quick-machine-recovery/). It requires cloud remediation, network access, and applicable update policies.

Plugins run detection and report applicable remediations to SRT, which selects repairs. The recovery environment manages subsequent boot attempts. The following contract applies to `AzureVMRecoveryPlugin`, not BCU.

### Supported payloads and invocation

Provide a PowerShell script (`.ps1`), Windows executable (`.exe`), or ZIP file containing them and their dependencies. The detection process extracts ZIP files before scanning.

The plugin launches each file with only a mode switch. It doesn't pass a context object or call script functions directly.

| Payload | Detection argument | Repair argument |
| --- | --- | --- |
| PowerShell `.ps1` | `-Detect` | `-Remediate` |
| Windows `.exe` | `/Detect` | `/Remediate` |

Scripts run in noninteractive PowerShell. The working directory is the entry point's directory. Detection and repair use separate processes, so they don't share in-memory state.

The extension supplies the PowerShell runtime rather than using PowerShell from the WinRE partition. `WindowsRecoveryExtension` version 1.0.12 packages PowerShell 7.4.22. Test your scripts and dependencies against the runtime packaged with the extension version on your VM.

| Mode | Exit code `0` | Any nonzero exit code |
| --- | --- | --- |
| Detect | Applicable and eligible for selection. | Not selected from this result, including detection failures. |
| Remediate | The repair process reports success. | The repair process reports failure. |

Return a process exit code, not a printed result. See [Interpret results](monitor-recovery.md#interpret-results) for the distinction between detection, repair execution, and recovery.

### Execution time limits

The detection phase and individual repair invocations have different time limits:

| Phase or operation | Time limit | Scope |
| --- | --- | --- |
| Detection phase | 10 minutes. | Shared across the entire detection phase, not a separate limit for each remediation. |
| Repair invocation | Three minutes. | Applies to each invocation of a remediation, including startup and verification. |

These limits apply to one WinRE recovery loop. When the loop completes and the next loop starts, the limits reset.

### Detection requirements

Keep detection read-only, bounded, and noninteractive. Check the target OS, architecture, component, and actual fault. A matching failure code alone is insufficient. Reject missing or ambiguous evidence.

### Repair requirements

A repair invocation that exceeds its three-minute limit is terminated and reported as a timeout.

When the remediation runs:

- Recheck the offline target and fault.
- Make only the required changes, and handle repeated or interrupted execution safely.
- Wait for child tools, and check their results.
- Unload mounted registry hives, and close file handles.
- Return `0` only after verifying the fix, including when it was already applied. Otherwise, return nonzero.

Interpret child-tool results before returning them. A tool's success-with-reboot-required code is nonzero and would be treated as failure by the plugin. Let the recovery environment manage reboot and shutdown.

## Example remediation

The following script reads `pBootFailureData.txt`, matches a failure signature, and changes an offline registry value.

> [!CAUTION]
> This example is fictional and not a Windows fix or a production-ready remediation. Its detection checks a reported signature, not the actual component fault, Windows version, or architecture. Implement and test those checks for your real repair before publishing it.

### Example failure and repair

For this example, signature X has these values:

| Field | Value |
| --- | --- |
| `OSStarted` | `1` |
| `FatalErrorCode` | `424242` |
| `CrashModule` | `ContosoBootFilter.sys` |

The repair sets `RecoveryEnabled` to DWORD `1` under this key in the offline Windows installation, not WinRE's own registry:

```text
HKLM\SOFTWARE\Contoso\RecoveryDemo
```

### Locate boot-failure data

The plugin writes `pBootFailureData.txt` in the extension root on the offline OS disk. The following example shows that disk mounted as `D:`:

```text
D:\Packages\Plugins\Microsoft.CPlat.Core.WindowsRecoveryExtension\
  pBootFailureData.txt
  <version>\Downloads\KnownRemediations\<application>\Remediation.ps1
  CustomRemediations\<staged command>\Remediation.ps1
```

The helper searches upward from the script directory. Don't hard-code the drive or handler folder.

Don't modify or package this runtime file. The sample requires its `OfflineWindowsDirectory` field and rejects an invalid target. Other remediations need to read the file only if their detection logic requires it.

### Implement detection and repair

Save the following script as `Remediation.ps1`. The plugin invokes it inside WinRE. Don't run the repair mode on your management workstation or a production system.

`Invoke-Detection` checks the example signature. `Invoke-Remediation` applies and verifies the example change. The helpers read diagnostics and validate the offline hive path.

```powershell
param(
    [switch] $Detect,
    [switch] $Remediate
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

function Read-BootFailureData {
    $directory = [System.IO.DirectoryInfo]::new($PSScriptRoot)
    $bootFailurePath = $null

    while ($null -ne $directory) {
        $candidate = Join-Path $directory.FullName 'pBootFailureData.txt'
        if (Test-Path -LiteralPath $candidate -PathType Leaf) {
            $bootFailurePath = $candidate
            break
        }
        $directory = $directory.Parent
    }

    if ($null -eq $bootFailurePath) {
        throw 'Boot-failure data was not found above the script directory.'
    }

    $data = @{}
    foreach ($line in Get-Content -LiteralPath $bootFailurePath) {
        if ($line -match '^(OSStarted|FatalErrorCode|CrashModule|OfflineWindowsDirectory):[ \t]*(.*)$') {
            $name = $Matches[1]
            $value = $Matches[2].Trim()

            if ($data.ContainsKey($name)) {
                throw "Duplicate boot-failure field: $name."
            }
            $data[$name] = $value
        }
    }

    if ($data['OSStarted'] -cnotin @('0', '1') -or
        -not $data.ContainsKey('FatalErrorCode')) {
        throw 'Boot-failure data is missing required fields or is invalid.'
    }

    $code = [long]::Parse(
        $data['FatalErrorCode'],
        [System.Globalization.NumberStyles]::Integer,
        [System.Globalization.CultureInfo]::InvariantCulture)
    if ($code -lt [int]::MinValue -or $code -gt [uint32]::MaxValue) {
        throw 'FatalErrorCode is outside the 32-bit range.'
    }
    if ($code -lt 0) {
        $code += 4294967296
    }
    $data['FatalErrorCode'] = $code

    return $data
}

function Invoke-Detection {
    $failure = Read-BootFailureData

    if ($failure['OSStarted'] -eq '1' -and
        $failure['FatalErrorCode'] -eq 424242 -and
        $failure['CrashModule'] -ieq 'ContosoBootFilter.sys') {
        return 0
    }

    return 1
}

function Get-OfflineSoftwareHivePath {
    $failure = Read-BootFailureData
    $windowsPath = $failure['OfflineWindowsDirectory']

    if ([string]::IsNullOrWhiteSpace($windowsPath) -or
        $windowsPath -notmatch '^[A-Za-z]:\\') {
        throw 'A valid offline Windows directory was not supplied by SRT.'
    }

    $windowsDirectory = Get-Item -LiteralPath (
        [System.IO.Path]::GetFullPath($windowsPath)) -Force
    if ($windowsDirectory -isnot [System.IO.DirectoryInfo] -or
        $null -eq $windowsDirectory.Parent) {
        throw 'The offline Windows path must be a directory, not a file or volume root.'
    }
    $windowsPath = $windowsDirectory.FullName.TrimEnd('\')

    if ([string]::IsNullOrWhiteSpace($env:SystemRoot)) {
        throw 'The running Windows directory cannot be identified safely.'
    }

    $runningWindows = [System.IO.Path]::GetFullPath(
        $env:SystemRoot).TrimEnd('\')

    if ($windowsPath -ieq $runningWindows) {
        throw 'Refusing to modify the running Windows installation.'
    }

    $hivePath = Join-Path $windowsPath 'System32\Config\SOFTWARE'
    $hive = Get-Item -LiteralPath $hivePath -Force
    if ($hive -isnot [System.IO.FileInfo]) {
        throw 'The offline SOFTWARE hive is not a file.'
    }

    $item = $hive
    while ($null -ne $item) {
        if (($item.Attributes -band
            [System.IO.FileAttributes]::ReparsePoint) -ne 0) {
            throw 'The registry hive path must not traverse reparse points.'
        }

        $item = if ($item -is [System.IO.FileInfo]) {
            $item.Directory
        }
        else {
            $item.Parent
        }
    }

    return $hive.FullName
}

function Invoke-Remediation {
    if ((Invoke-Detection) -ne 0) {
        throw 'The failure does not match this remediation. No changes made.'
    }

    $hivePath = Get-OfflineSoftwareHivePath
    $mountName = 'WinREExample_' + [guid]::NewGuid().ToString('N')
    $loadOutput = & reg.exe load "HKLM\$mountName" $hivePath 2>&1
    if ($LASTEXITCODE -ne 0) {
        throw "Could not load the offline registry hive: $loadOutput"
    }

    $key = $null
    try {
        $key = [Microsoft.Win32.Registry]::LocalMachine.CreateSubKey(
            "$mountName\Contoso\RecoveryDemo")

        $key.SetValue(
            'RecoveryEnabled',
            1,
            [Microsoft.Win32.RegistryValueKind]::DWord)

        if ($key.GetValue('RecoveryEnabled') -ne 1) {
            throw 'The registry change could not be verified.'
        }
    }
    finally {
        if ($null -ne $key) {
            $key.Dispose()
        }

        $unloadOutput = & reg.exe unload "HKLM\$mountName" 2>&1
        if ($LASTEXITCODE -ne 0) {
            throw "Could not unload the offline registry hive: $unloadOutput"
        }
    }

    return 0
}

if ($Detect.IsPresent -eq $Remediate.IsPresent) {
    [Console]::Error.WriteLine('Specify exactly one mode: -Detect or -Remediate.')
    exit 2
}

try {
    $exitCode = if ($Detect) {
        Invoke-Detection
    }
    else {
        Invoke-Remediation
    }

    if ($exitCode -isnot [int]) {
        throw 'The function must return one integer exit code. Write logs separately.'
    }

    exit $exitCode
}
catch {
    [Console]::Error.WriteLine($_.Exception.Message)
    exit 1
}
```

### Verify expected behavior

Test your adapted remediation with the target WinRE runtime and architecture before publishing it. The example has the following behavior:

- **Detect:** Returns `0` for signature X, otherwise `1`, without changes.
- **Remediate:** Rechecks the signature, validates the target, writes and verifies the value, and unloads the hive.
- **Failure:** Returns nonzero for invalid input or registry-operation errors.
- **Repeat execution:** Sets DWORD `1` again safely. Detection still matches the unchanged failure report.

For a real repair, verify the component and on-disk fault as well as the reported signature. Store persistent logs on the offline disk without credentials or unnecessary customer data. Test this custom-remediation contract on your disposable VM separately from the [QMR simulation](test-quick-machine-recovery.md).

## Package the remediation

Choose a standalone file or a ZIP package based on the remediation's dependencies.

### Use a standalone file

Publish the `.ps1` or `.exe` directly if it needs no additional files. It must implement the [detection and repair contract](#remediation-contract).

### Use a ZIP package

Use a ZIP file for dependencies such as libraries, modules, or repair data. Put the entry point at the ZIP root:

```text
Remediation.ps1
repair-data.json
```

Alternatively, put it inside one top-level folder:

```text
Package\
  Remediation.ps1
  repair-data.json
```

Deeper entry points aren't discovered. Load supporting files explicitly from their subdirectories.

Every `.ps1` and `.exe` file at these depths is treated as a remediation. No special file name is required. Keep helpers outside these locations unless they implement the same contract. Verify compatibility with the VM architecture and WinRE runtime.

## Clean up test artifacts

Exclude fixtures, logs, credentials, and runtime boot-failure data from the published package. Release hive and disk handles. Preserve required diagnostics, and remove unneeded test resources without deleting shared resources.

## Next steps

- [Publish the tested artifact](publish-remediation.md).
- [Pre-stage or apply a published remediation](apply-remediation.md).
- [Monitor recovery mode and results](monitor-recovery.md).

## Related content

- [Windows RE technical reference](/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference).
- [PowerShell command-line and exit-code behavior](/powershell/module/microsoft.powershell.core/about/about_pwsh).
