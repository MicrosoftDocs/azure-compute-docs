---
title: Create install, uninstall, and update scripts for VM Applications
description: Learn how to create reliable install, uninstall, and update scripts for common Windows and Linux VM Application package formats.
author: tanmaygore
ms.author: tagore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 10/23/2026
ai-usage: ai-assisted
# Customer intent: As a VM Application publisher, I want to create install, uninstall, and update scripts for an application package so that I can publish and deploy it successfully.
---

# Create install, uninstall, and update scripts for VM Applications

Each VM Application version needs an install script and an uninstall script, and can have an update script. In the Azure portal, these scripts are the **Install script**, **Uninstall script**, and **Update script** fields. In the API, they're the `install`, `remove`, and `update` properties.

- For a common application, copy the tested scripts from [Scripts for common Windows applications](#scripts-for-common-windows-applications) or [Scripts for common Linux applications](#scripts-for-common-linux-applications).
- For any other application, see [Create scripts for other applications](#create-scripts-for-other-applications).

## Prerequisites

- An Azure Compute Gallery and a VM Application. See [Create and deploy a VM Application](vm-applications-how-to.md).
- A storage account for the package, with a SAS URL or a [managed identity attached to the gallery](vm-applications-publish-with-managed-identity.md).
- A test VM with the same operating system as your target VMs.

The scripts expect the package to be named `appInstall.<extension>`, for example `appInstall.msi`. Set `packageFileName` to that name when you [create the VM Application version](vm-applications-how-to.md#create-the-vm-application). If you don't set this name, replace `appInstall.<extension>` in the scripts with the VM Application name. See [File naming for deployed VM applications](vm-applications.md#file-naming-for-deployed-vm-application).

## Scripts for common Windows applications

| Application | Format | packageFileName | Install script | Uninstall values | Update script |
| --- | --- | --- | --- | --- | --- |
| 7-Zip | .exe | appInstall.exe | appInstall.exe /S | $displayNamePattern='7-Zip\*'; $fallbackArguments='/S' | appInstall.exe /S |
| 7-Zip | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='7-Zip\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart |
| Google Chrome | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='\*Chrome\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart |
| Python 3.13 | .exe | appInstall.exe | appInstall.exe /quiet /norestart InstallAllUsers=1 PrependPath=1 | $displayNamePattern='Python 3.13\*'; $fallbackArguments='/quiet /norestart' | appInstall.exe /quiet /norestart InstallAllUsers=1 PrependPath=1 |
| PowerShell 7 | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='\*Power\*Shell\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart |
| PuTTY | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='\*PuTTY release 0.85 (64-bit)\*'; $fallbackArguments='' | Leave blank. |
| Azure CLI | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='\*Azure\*CLI\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart |
| Visual Studio Code | .exe (Inno Setup) | appInstall.exe | appInstall.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /mergetasks=!runcode | $displayNamePattern='\*Visual\*Studio\*Code\*'; $fallbackArguments='/VERYSILENT /SUPPRESSMSGBOXES /NORESTART' | appInstall.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /mergetasks=!runcode |
| Azure Storage Explorer | .exe (Inno Setup) | appInstall.exe | appInstall.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /ALLUSERS | $displayNamePattern='\*Azure Storage\*Explorer\*'; $fallbackArguments='/VERYSILENT /SUPPRESSMSGBOXES /NORESTART' | appInstall.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /ALLUSERS |
| Adobe Acrobat Reader | .exe | appInstall.exe | appInstall.exe -sfx\_nu /sAll /rs /msi | $displayNamePattern='\*Adobe Acrobat (64-bit)\*'; $fallbackArguments='/S' | appInstall.exe -sfx\_nu /sAll /rs /msi |
| SQL Server Management Studio 22 | .exe | appInstall.exe | appInstall.exe --quiet --wait --campaign winget | $displayNamePattern='\*SQL\*Server\*Management\*Studio 22\*'; $fallbackArguments='/S' | appInstall.exe update --quiet --wait --campaign winget |
| Cinc Client | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart | $displayNamePattern='Cinc Client\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart |
| Automox Agent | .msi | appInstall.msi | msiexec /i appInstall.msi /qn /norestart ACCESSKEY=&lt;automox-access-key&gt; | $displayNamePattern='Automox Agent\*'; $fallbackArguments='' | msiexec /i appInstall.msi /qn /norestart ACCESSKEY=&lt;automox-access-key&gt; |

To publish an application from the preceding table:

1. Download the installer in the listed format. Use a specific release, not a link to the latest release.
1. Upload the installer to your storage account.
1. Create the VM Application version with the listed `packageFileName`.
1. Set the install and update scripts to the commands in the table. If the update script is **Leave blank.**, leave it empty.
1. Set the uninstall script to the [Windows uninstall script](#windows-uninstall-script), and replace `$displayNamePattern='<DISPLAY-NAME-PATTERN>'; $fallbackArguments=''` with the **Uninstall values** from the table.
1. Test install, update, and uninstall on your test VM. See [Validate before publishing](#validate-before-publishing).

Notes:

- **Automox Agent:** Replace `<automox-access-key>` with your access key from the Automox console. Without a valid key, the agent installs but doesn't appear in the console.
- **Cinc Client and Automox Agent:** These agents can update themselves through the vendor console. To control the version from Azure, publish a new application version.

> [!NOTE]
> Anyone who can read the gallery can see the scripts, including access keys. Restrict gallery read access if a value is sensitive.

> [!IMPORTANT]
> Installer formats, silent arguments, and return codes can change between releases. Verify them with the publisher for the release you download.

### Windows uninstall script

This script works for any application that registers in the Windows uninstall registry. It runs the registered uninstall command, succeeds if the application is already absent, and verifies removal. Set the two values at the start:

- `$displayNamePattern`: A pattern that matches only this application. See [Find the display name of a Windows application](#find-the-display-name-of-a-windows-application).
- `$fallbackArguments`: Silent uninstall arguments, used only when the registry entry has no `QuietUninstallString`.

If you don't replace `<DISPLAY-NAME-PATTERN>`, the script exits with code `87`.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ErrorActionPreference='SilentlyContinue'; $displayNamePattern='<DISPLAY-NAME-PATTERN>'; $fallbackArguments=''; if($displayNamePattern -like '*DISPLAY-NAME-PATTERN*'){ Write-Output 'UNINSTALL: set displayNamePattern before publishing'; exit 87 }; $keys=@('HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*','HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*'); $apps=@(Get-ItemProperty $keys | Where-Object { $_.DisplayName -like $displayNamePattern -and $_.SystemComponent -ne 1 }); if($apps.Count -eq 0){ Write-Output 'UNINSTALL: no matching registry entry; nothing to do'; exit 0 }; $code=0; foreach($a in $apps){ if(-not (Get-ItemProperty $a.PSPath)){ continue }; $c=$a.QuietUninstallString; if(-not $c){ $c=$a.UninstallString; if(-not $c){ continue }; $g=[regex]::Match($c,'\{[0-9A-Fa-f-]+\}').Value; if($c -match 'msiexec' -and $g){ $c='msiexec.exe /x '+$g+' /qn /norestart' } elseif($fallbackArguments){ $c=$c+' '+$fallbackArguments } else { Write-Output ('UNINSTALL: set fallbackArguments for '+$a.DisplayName); exit 87 } }; $p=Start-Process cmd.exe -ArgumentList '/c',$c -NoNewWindow -PassThru; $null=$p.Handle; if($p.WaitForExit(600000)){ $rc=$p.ExitCode } else { try{ $p.Kill() }catch{}; $rc=1460 }; $gone=-not (Get-ItemProperty $a.PSPath); Write-Output ('UNINSTALL: '+$a.DisplayName+' exit='+$rc+' removed='+$gone); if(-not $gone){ if($rc -eq 0){ $code=1 } else { $code=$rc } } }; exit $code"
```

## Scripts for common Linux applications

Find your application and distribution. Combinations that aren't listed weren't tested.

| Application | Format | Distributions | packageFileName | Install script | Uninstall script | Update script | Dependencies |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cinc Client | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' cinc 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 cinc; else echo 'cinc not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| Cinc Client | .rpm | RHEL, SLES | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm | if rpm -q cinc &gt;/dev/null 2&gt;&1; then rpm -e cinc; else echo 'cinc not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| PowerShell 7 | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' powershell 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 powershell; else echo 'powershell not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| PowerShell 7 | .rpm | RHEL, Azure Linux | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm | if rpm -q powershell &gt;/dev/null 2&gt;&1; then rpm -e powershell; else echo 'powershell not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| PowerShell 7 | .tar.gz (deb bundle) | Ubuntu, Debian | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.deb; for f in \*.deb; do \[ "$f" = app.deb \] && continue; i=$(dpkg-query -W -f='${db:Status-Status} ${Version}' "$(dpkg-deb -f "$f" Package)" 2&gt;/dev/null); if \[ "${i%% \*}" != installed \] \|\| dpkg --compare-versions "${i#\* }" lt "$(dpkg-deb -f "$f" Version)"; then n="$n $f"; fi; done; DEBIAN\_FRONTEND=noninteractive dpkg -i $n; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' powershell 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 powershell; else echo 'powershell not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| PowerShell 7 | .tar.gz (rpm bundle) | RHEL, Azure Linux | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.rpm; for f in \*.rpm; do \[ "$f" = app.rpm \] && continue; if rpm -U --test --nodeps "$f" &gt;/dev/null 2&gt;&1; then n="$n $f"; fi; done; rpm -Uvh --replacepkgs $n; } | if rpm -q powershell &gt;/dev/null 2&gt;&1; then rpm -e powershell; else echo 'powershell not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Azure CLI | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' azure-cli 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 azure-cli; else echo 'azure-cli not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| Azure CLI | .rpm | RHEL, Azure Linux | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm | if rpm -q azure-cli &gt;/dev/null 2&gt;&1; then rpm -e azure-cli; else echo 'azure-cli not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| Azure CLI | .tar.gz (rpm bundle) | RHEL | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.rpm; for f in \*.rpm; do \[ "$f" = app.rpm \] && continue; if rpm -U --test --nodeps "$f" &gt;/dev/null 2&gt;&1; then n="$n $f"; fi; done; rpm -Uvh --replacepkgs $n; } | if rpm -q azure-cli &gt;/dev/null 2&gt;&1; then rpm -e azure-cli; else echo 'azure-cli not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Automox Agent | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } && /opt/amagent/amagent --setkey &lt;automox-access-key&gt; && systemctl restart amagent | if \[ "$(dpkg-query -W -f='${db:Status-Status}' amagent 2&gt;/dev/null)" = installed \]; then /opt/amagent/amagent --deregister; DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 amagent; else echo 'amagent not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| Automox Agent | .rpm | RHEL, SLES | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm && /opt/amagent/amagent --setkey &lt;automox-access-key&gt; && systemctl restart amagent | if rpm -q amagent &gt;/dev/null 2&gt;&1; then /opt/amagent/amagent --deregister; rpm -e amagent; else echo 'amagent not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| Automox Agent | .tar.gz (deb bundle) | Ubuntu, Debian | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.deb; for f in \*.deb; do \[ "$f" = app.deb \] && continue; i=$(dpkg-query -W -f='${db:Status-Status} ${Version}' "$(dpkg-deb -f "$f" Package)" 2&gt;/dev/null); if \[ "${i%% \*}" != installed \] \|\| dpkg --compare-versions "${i#\* }" lt "$(dpkg-deb -f "$f" Version)"; then n="$n $f"; fi; done; DEBIAN\_FRONTEND=noninteractive dpkg -i $n && /opt/amagent/amagent --setkey &lt;automox-access-key&gt; && systemctl restart amagent; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' amagent 2&gt;/dev/null)" = installed \]; then /opt/amagent/amagent --deregister; DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 amagent; else echo 'amagent not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Automox Agent | .tar.gz (rpm bundle) | RHEL, SLES | appInstall.tar.gz | \[ -L /etc/init.d \] \|\| rmdir /etc/init.d 2&gt;/dev/null; mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.rpm; for f in \*.rpm; do \[ "$f" = app.rpm \] && continue; if rpm -U --test --nodeps "$f" &gt;/dev/null 2&gt;&1; then n="$n $f"; fi; done; rpm -Uvh --replacepkgs $n && /opt/amagent/amagent --setkey &lt;automox-access-key&gt; && systemctl restart amagent; } | if rpm -q amagent &gt;/dev/null 2&gt;&1; then /opt/amagent/amagent --deregister; rpm -e amagent; else echo 'amagent not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Google Chrome | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' google-chrome-stable 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 google-chrome-stable; else echo 'google-chrome-stable not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| Google Chrome | .rpm | RHEL, SLES | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm | if rpm -q google-chrome-stable &gt;/dev/null 2&gt;&1; then rpm -e google-chrome-stable; else echo 'google-chrome-stable not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| Google Chrome | .tar.gz (deb bundle) | Ubuntu, Debian | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.deb; for f in \*.deb; do \[ "$f" = app.deb \] && continue; i=$(dpkg-query -W -f='${db:Status-Status} ${Version}' "$(dpkg-deb -f "$f" Package)" 2&gt;/dev/null); if \[ "${i%% \*}" != installed \] \|\| dpkg --compare-versions "${i#\* }" lt "$(dpkg-deb -f "$f" Version)"; then n="$n $f"; fi; done; DEBIAN\_FRONTEND=noninteractive dpkg -i $n; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' google-chrome-stable 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 google-chrome-stable; else echo 'google-chrome-stable not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Google Chrome | .tar.gz (rpm bundle) | RHEL, SLES | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.rpm; for f in \*.rpm; do \[ "$f" = app.rpm \] && continue; if rpm -U --test --nodeps "$f" &gt;/dev/null 2&gt;&1; then n="$n $f"; fi; done; rpm -Uvh --replacepkgs $n; } | if rpm -q google-chrome-stable &gt;/dev/null 2&gt;&1; then rpm -e google-chrome-stable; else echo 'google-chrome-stable not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Visual Studio Code | .deb | Ubuntu, Debian | appInstall.deb | export DEBIAN\_FRONTEND=noninteractive; apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb \|\| { apt-get update -o DPkg::Lock::Timeout=600 && apt-get install -y -o DPkg::Lock::Timeout=600 ./appInstall.deb; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' code 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 code; else echo 'code not installed'; fi | Same as install. | Downloaded by `apt-get`, so the VM needs repository access. For offline installs, use .tar.gz. |
| Visual Studio Code | .rpm | RHEL, SLES | appInstall.rpm | rpm -Uvh --replacepkgs appInstall.rpm | if rpm -q code &gt;/dev/null 2&gt;&1; then rpm -e code; else echo 'code not installed'; fi | Same as install. | Must already be on the VM. Otherwise, use .tar.gz. |
| Visual Studio Code | .tar.gz (deb bundle) | Ubuntu, Debian | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.deb; for f in \*.deb; do \[ "$f" = app.deb \] && continue; i=$(dpkg-query -W -f='${db:Status-Status} ${Version}' "$(dpkg-deb -f "$f" Package)" 2&gt;/dev/null); if \[ "${i%% \*}" != installed \] \|\| dpkg --compare-versions "${i#\* }" lt "$(dpkg-deb -f "$f" Version)"; then n="$n $f"; fi; done; DEBIAN\_FRONTEND=noninteractive dpkg -i $n; } | if \[ "$(dpkg-query -W -f='${db:Status-Status}' code 2&gt;/dev/null)" = installed \]; then DEBIAN\_FRONTEND=noninteractive apt-get purge -y -o DPkg::Lock::Timeout=600 code; else echo 'code not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |
| Visual Studio Code | .tar.gz (rpm bundle) | RHEL, SLES | appInstall.tar.gz | mkdir -p pkgs && tar -xzf appInstall.tar.gz -C pkgs && cd pkgs && { n=app.rpm; for f in \*.rpm; do \[ "$f" = app.rpm \] && continue; if rpm -U --test --nodeps "$f" &gt;/dev/null 2&gt;&1; then n="$n $f"; fi; done; rpm -Uvh --replacepkgs $n; } | if rpm -q code &gt;/dev/null 2&gt;&1; then rpm -e code; else echo 'code not installed'; fi | Same as install. | Bundled. The VM doesn't need repository access. See [Build a dependency bundle](#build-a-dependency-bundle). |

To publish an application from the table:

1. Choose a row. For VMs without repository access, or when .rpm dependencies aren't on the VM, use a .tar.gz row.
1. Download the vendor package for your distribution. Use a specific release, not a link to the latest release.
1. For a .tar.gz row, [build a dependency bundle](#build-a-dependency-bundle).
1. Upload the package to your storage account.
1. Create the VM Application version with the listed `packageFileName`.
1. Set the install, uninstall, and update scripts to the commands in the table.
1. Test install, update, and uninstall on your test VM. See [Validate before publishing](#validate-before-publishing).

The scripts run as root in the package's download directory.

Notes:

- **Automox Agent:** Replace `<automox-access-key>` with your access key from the Automox console. The key is a UUID. The uninstall script deregisters the agent first.
- **Cinc Client:** Azure Linux 3 isn't supported, because it doesn't provide `libcrypt.so.2`. Uninstall leaves configuration in `/etc/cinc`.
- **Google Chrome and Visual Studio Code:** Azure Marketplace RHEL images have a small root volume. Confirm free space for `/opt` and `/usr/share`.
- **All applications:** Uninstall succeeds if the application is absent, and leaves dependencies in place. Tested on Azure Marketplace images of Ubuntu 24.04 LTS, Debian 12, RHEL 9, SLES 15 SP6, and Azure Linux 3.

> [!NOTE]
> Anyone who can read the gallery can see the scripts, including access keys. Restrict gallery read access if a value is sensitive.

> [!IMPORTANT]
> Package names, dependencies, and supported distributions can change between releases. Verify them with the publisher for the release you download.

### Build a dependency bundle

A bundle is a .tar.gz file that contains the vendor package and all its dependencies, so the VM doesn't need repository access. The .tar.gz install script installs only the bundled dependencies that are missing or older on the VM.

1. Create a VM from a clean image of the target distribution and release, such as RHEL 9. A clean image ensures that the bundle includes every dependency. The VM needs repository access.
1. Copy the vendor package and the build script to the VM, and run the script as root:
   - Ubuntu and Debian: `sh build-deb-bundle.sh <vendor-package.deb>`
   - RHEL, SLES, and Azure Linux: `sh build-rpm-bundle.sh <vendor-package.rpm>`
1. Upload the resulting `appInstall.tar.gz` as the application package.

`build-deb-bundle.sh`:

```bash
#!/bin/sh
# build-deb-bundle.sh <vendor-package.deb>
# Run on a clean VM of the target distribution. Writes appInstall.tar.gz to the
# current directory: the vendor package as app.deb plus every dependency.
set -e
app=$(readlink -f "$1")
pkg=$(dpkg-deb -f "$app" Package)
work=$(mktemp -d)
apt-get update
apt-get clean
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --download-only "$app"
for f in /var/cache/apt/archives/*.deb; do
  [ -e "$f" ] || continue
  if [ "$(dpkg-deb -f "$f" Package)" != "$pkg" ]; then cp "$f" "$work/"; fi
done
cp "$app" "$work/app.deb"
tar -czf appInstall.tar.gz -C "$work" .
rm -rf "$work"
tar -tzf appInstall.tar.gz
```

`build-rpm-bundle.sh`:

```bash
#!/bin/sh
# build-rpm-bundle.sh <vendor-package.rpm>
# Run on a clean VM of the target distribution. Writes appInstall.tar.gz to the
# current directory: the vendor package as app.rpm plus every dependency.
set -e
app=$(readlink -f "$1")
pkg=$(rpm -qp --qf '%{NAME}' "$app" 2>/dev/null)
work=$(mktemp -d)
if command -v dnf >/dev/null 2>&1; then
  # Fresh RHEL images ship with no imported keys, and dnf's automatic import fails.
  # Boot-time jobs can hold the RPM lock briefly, so retry the import.
  k=/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
  if [ -f "$k" ]; then i=0; until rpm --import "$k"; do i=$((i+1)); [ "$i" -lt 30 ] || exit 1; sleep 10; done; fi
  dnf install -y --downloadonly --setopt=install_weak_deps=False --downloaddir="$work" "$app"
elif command -v tdnf >/dev/null 2>&1; then
  tdnf install -y --downloadonly --downloaddir="$work" "$app"
elif command -v zypper >/dev/null 2>&1; then
  mkdir -p "$work/cache"
  zypper --non-interactive --no-gpg-checks --pkg-cache-dir "$work/cache" install --download-only --no-recommends --allow-unsigned-rpm "$app"
  find "$work/cache" -name '*.rpm' -exec mv {} "$work/" \;
  rm -rf "$work/cache"
else
  echo "dnf, tdnf, or zypper is required" >&2
  exit 1
fi
for f in "$work"/*.rpm; do
  [ -e "$f" ] || continue
  if [ "$(rpm -qp --qf '%{NAME}' "$f" 2>/dev/null)" = "$pkg" ]; then rm -f "$f"; fi
done
cp "$app" "$work/app.rpm"
tar -czf appInstall.tar.gz -C "$work" .
rm -rf "$work"
tar -tzf appInstall.tar.gz
```

## Create scripts for other applications

If your application isn't listed, create its scripts by using this guidance.

### Script requirements

Every script must:

- Install from the downloaded package. To include dependencies, [build a dependency bundle](#build-a-dependency-bundle).
- Run silently, without prompts.
- Wait for the installer and its child processes to finish.
- Not restart the VM. Use `scriptBehaviorAfterReboot` to control post-restart behavior.
- Return `0` on success and nonzero on failure. Treat other codes as success only if the publisher documents them.
- Stay within 4,096 characters. Put longer logic in a script inside a ZIP or TAR package.
- Be safe to run again.

Azure runs each script in the package's download directory. Windows uses `cmd.exe` and Linux uses `/bin/bash`.

### Create an install script

Use the publisher's unattended install arguments.

#### Install an EXE on Windows

EXE installers such as InstallShield, Inno Setup, NSIS, and Burn use different silent arguments, so `/S`, `/silent`, and `/quiet` aren't interchangeable. Remove any placeholder that the installer doesn't need. PowerShell waits for the installer and returns its exit code.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "$process=Start-Process -FilePath '.\appInstall.exe' -ArgumentList '<silent-install-arguments>','<no-restart-argument>' -Wait -PassThru; exit $process.ExitCode"
```

#### Install an MSI on Windows

`/qn` hides the UI, `/norestart` prevents a restart, and `start /wait` waits for Windows Installer. Add MSI properties after `/norestart`.

```cmd
start /wait msiexec.exe /i ".\appInstall.msi" /qn /norestart
```

#### Install a ZIP or PowerShell package on Windows

Put the application files and an unattended `install.ps1` in the ZIP, with the same layout in every version. `install.ps1` must copy the application out of the download directory, because Azure can delete that directory.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "$ErrorActionPreference='Stop'; Remove-Item '.\payload' -Recurse -Force -ErrorAction SilentlyContinue; Expand-Archive -LiteralPath '.\appInstall.zip' -DestinationPath '.\payload' -Force; & '.\payload\install.ps1'; exit $LASTEXITCODE"
```

For a .ps1 package, use an `Action` parameter so that one script handles all three operations.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\appInstall.ps1" -Action Install
```

#### Install a DEB package on Debian or Ubuntu

`dpkg` doesn't download dependencies, so the VM must already have them. To include them, [build a dependency bundle](#build-a-dependency-bundle).

```bash
sudo env DEBIAN_FRONTEND=noninteractive dpkg --install "./appInstall.deb"
```

#### Install an RPM package on RHEL, CentOS, or SUSE

`--upgrade` installs or upgrades the package, and `--replacepkgs` makes retries safe. The VM must already have dependencies. To include them, [build a dependency bundle](#build-a-dependency-bundle).

```bash
sudo rpm --upgrade --replacepkgs "./appInstall.rpm"
```

#### Install a TAR or shell script package on Linux

Include the application files and an unattended `install.sh`, which must copy the application out of the download directory. Use `-xzf` for .tar.gz and `-xf` for .tar.

```bash
rm -rf ./payload && mkdir ./payload && tar -xzf "./appInstall.tar.gz" -C ./payload && chmod +x ./payload/install.sh && sudo ./payload/install.sh
```

For a .sh package, accept an action argument. Use `sudo` only for system-wide installs.

```bash
chmod +x "./appInstall.sh" && sudo "./appInstall.sh" install
```

### Create an uninstall script

Along with the [script requirements](#script-requirements), an uninstall script must:

- Remove only the target application, including its files outside the download directory.
- Return `0` if the application is already absent.
- Keep shared dependencies and customer data, unless the publisher's removal policy says otherwise.

#### Uninstall an EXE on Windows

Use the [Windows uninstall script](#windows-uninstall-script). To find its values, see [Find the display name of a Windows application](#find-the-display-name-of-a-windows-application).

#### Uninstall an MSI on Windows

Use the [Windows uninstall script](#windows-uninstall-script). It's safe to retry, handles product codes that change between versions, and verifies removal.

If only VM Applications installs and removes the MSI, at the same version, you can use the package file or the installed product code instead:

```cmd
start /wait msiexec.exe /x ".\appInstall.msi" /qn /norestart
```

```cmd
start /wait msiexec.exe /x "{PRODUCT-CODE}" /qn /norestart
```

> [!IMPORTANT]
> `msiexec.exe` returns `1605` if the product is already absent. Check first, or treat `1605` as success after you confirm removal.

#### Uninstall a ZIP or PowerShell package on Windows

Include an idempotent `uninstall.ps1` that removes only the application's files.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "$ErrorActionPreference='Stop'; Remove-Item '.\payload' -Recurse -Force -ErrorAction SilentlyContinue; Expand-Archive -LiteralPath '.\appInstall.zip' -DestinationPath '.\payload' -Force; & '.\payload\uninstall.ps1'; exit $LASTEXITCODE"
```

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\appInstall.ps1" -Action Uninstall
```

#### Uninstall a DEB package on Debian or Ubuntu

This script reads the package name from the .deb file and removes the package if it's installed.

```bash
packageName="$(dpkg-deb --field "./appInstall.deb" Package)" && status="$(dpkg-query -W -f='${db:Status-Status}' "$packageName" 2>/dev/null || true)" && if [ "$status" = "installed" ]; then sudo dpkg --remove "$packageName"; fi
```

#### Uninstall an RPM package on RHEL, CentOS, or SUSE

This script reads the package name from the .rpm file and removes the package if it's installed.

```bash
packageName="$(rpm -qp --queryformat '%{NAME}' "./appInstall.rpm")" && if rpm -q "$packageName" >/dev/null 2>&1; then sudo rpm --erase "$packageName"; fi
```

#### Uninstall a TAR or shell script package on Linux

Include an idempotent `uninstall.sh` that returns `0` if the application is already absent.

```bash
rm -rf ./payload && mkdir ./payload && tar -xzf "./appInstall.tar.gz" -C ./payload && chmod +x ./payload/uninstall.sh && sudo ./payload/uninstall.sh
```

```bash
chmod +x "./appInstall.sh" && sudo "./appInstall.sh" uninstall
```

### Find the display name of a Windows application

The [Windows uninstall script](#windows-uninstall-script) needs a `$displayNamePattern` that matches only your application. Applications in the Windows table already have one.

Install the exact package that you plan to publish on a test VM, and then run:

#### [Azure CLI](#tab/cli)

```azurecli
az vm run-command invoke \
  --resource-group <resource-group> \
  --name <vm-name> \
  --command-id RunPowerShellScript \
  --scripts 'Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -and $_.SystemComponent -ne 1 } | Sort-Object DisplayName | Format-List DisplayName, QuietUninstallString, UninstallString'
```

#### [Azure PowerShell](#tab/powershell)

```azurepowershell
Invoke-AzVMRunCommand `
  -ResourceGroupName <resource-group> `
  -VMName <vm-name> `
  -CommandId 'RunPowerShellScript' `
  -ScriptString 'Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*","HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -and $_.SystemComponent -ne 1 } | Sort-Object DisplayName | Format-List DisplayName, QuietUninstallString, UninstallString'
```

---

Keep the outer quotation marks single so that your local shell doesn't expand `$_`.

Find your application in the output. If you aren't sure which entry it is, compare the output before and after you install the package. Keep the product name in the pattern, and replace the parts that change between releases with a wildcard:

| Registered display name | Pattern to use |
| --- | --- |
| `7-Zip 26.03 (x64 edition)` | `7-Zip*` |
| `Python 3.13.15 (64-bit)` | `Python 3.13*` |
| `Contoso Agent 4.2` | `Contoso Agent*` |

Start the pattern with the product name. For example, `Python 3.13*` excludes `Python Launcher`, but `*Python*` matches both. To confirm the pattern, add `-and $_.DisplayName -like '<your-pattern>'` to the `Where-Object` filter, and check that only your application is returned.

### Create an update script

The update script is optional. If it's blank, VM Applications runs the old version's uninstall script and then the new version's install script. Leave it blank if the installer has no tested in-place upgrade.

An update script must keep configuration and customer data, remove obsolete files, and upgrade from every older version that you support.

#### Update an EXE on Windows

Use the publisher's update arguments. Some installers upgrade by using their normal install arguments.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "$process=Start-Process -FilePath '.\appInstall.exe' -ArgumentList '<silent-update-arguments>','<no-restart-argument>' -Wait -PassThru; exit $process.ExitCode"
```

#### Update an MSI on Windows

Use this command only if the new MSI's upgrade metadata covers the installed versions. Test major and minor upgrades.

```cmd
start /wait msiexec.exe /i ".\appInstall.msi" /qn /norestart
```

#### Update a ZIP or PowerShell package on Windows

Include an `update.ps1` that keeps configuration, replaces changed files, and removes obsolete ones.

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "$ErrorActionPreference='Stop'; Remove-Item '.\payload' -Recurse -Force -ErrorAction SilentlyContinue; Expand-Archive -LiteralPath '.\appInstall.zip' -DestinationPath '.\payload' -Force; & '.\payload\update.ps1'; exit $LASTEXITCODE"
```

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\appInstall.ps1" -Action Update
```

#### Update a DEB package on Debian or Ubuntu

`dpkg --install` upgrades to a newer version. Dependencies must already be on the VM.

```bash
sudo env DEBIAN_FRONTEND=noninteractive dpkg --install "./appInstall.deb"
```

#### Update an RPM package on RHEL, CentOS, or SUSE

```bash
sudo rpm --upgrade --replacepkgs "./appInstall.rpm"
```

#### Update a TAR or shell script package on Linux

Include an `update.sh` that keeps required data.

```bash
rm -rf ./payload && mkdir ./payload && tar -xzf "./appInstall.tar.gz" -C ./payload && chmod +x ./payload/update.sh && sudo ./payload/update.sh
```

```bash
chmod +x "./appInstall.sh" && sudo "./appInstall.sh" update
```

## Validate before publishing

You can't change the scripts or `packageFileName` after you create a version. On a test VM, confirm:

1. **Environment:** The VM matches the target operating system and architecture. For self-contained packages, block outbound internet access.
1. **Package name:** The downloaded file name matches the scripts.
1. **Install:** Installation finishes without prompts or restarts, and creates the expected files.
1. **Repeat install:** Running the install script again succeeds.
1. **Update:** Updating from each supported version succeeds and keeps configuration and data.
1. **Uninstall:** Removal deletes only the application's files, and returns `0` when run twice.
1. **Exit codes:** Success returns `0`, failure returns nonzero, and each script waits for child processes.
1. **Logs:** Review `stdout` and `stderr` for each action. See [View logs of application installation using Run Command](vm-applications-manage.md#view-logs-of-application-installation-using-run-command).
1. **Failure reporting:** Set `treatFailureAsDeploymentFailure` to `true` in the application profile so that failures report as deployment failures.
