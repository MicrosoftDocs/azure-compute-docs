---
title: Test quick machine recovery on an Azure VM (Preview)
description: Test the quick machine recovery simulation on an eligible Windows client Azure VM, verify the result, collect diagnostics, and clean up safely.
author: GabstaMSFT
ms.author: gabsta
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.date: 11/09/2026
ai-usage: ai-assisted
---

# Test quick machine recovery on an Azure VM (Preview)

**Applies to:** :heavy_check_mark: Windows client VMs

Quick machine recovery (QMR) uses Windows Recovery Environment (WinRE) to find Microsoft remediations through Windows Update. Test mode simulates a boot failure without damaging Windows. Use this procedure to test QMR on an eligible Windows client VM. Test [custom remediations through remote recovery for Azure VMs](apply-remediation.md) separately.

> [!WARNING]
> Use a disposable, nonproduction VM with backups, a rollback plan, guest access, and [Boot Diagnostics](../../boot-diagnostics.md). This test changes recovery settings and reboots the guest into WinRE. Don't force a real boot failure.

## Prerequisites

- Use a Windows client VM enrolled in the Windows Insider **Experimental channel**, as required by [QMR test mode](/windows/configuration/quick-machine-recovery/#test-mode). Don't apply these enrollment instructions to Windows Server.
- Confirm that the VM's Windows edition, build, and recovery environment support the simulation before testing. Use a separate Windows client VM from the Windows Server 2025 image supported by the [remote recovery onboarding procedure](deploy-vm.md).
- Obtain administrator access inside the test VM and permission to view its Boot Diagnostics.
- Verify Windows Update connectivity from WinRE, not just normal Windows.
- Record the current recovery configuration and your rollback method. Configuration exports can contain network credentials. Store them securely.

QMR itself doesn't require Windows Insider enrollment. Only this test-mode procedure requires an eligible Windows client device in the Experimental channel. Test mode isn't currently available on Windows Server 2025, so you can't use this simulation on the image supported by the remote recovery preview. For general QMR support and configuration, see the [Windows QMR documentation](/windows/configuration/quick-machine-recovery/).

## Prepare the client VM

1. Follow [Windows Insider enrollment](https://www.microsoft.com/windowsinsider/getting-started). Don't use registry overrides to bypass enrollment requirements.
1. [Configure QMR](/windows/configuration/quick-machine-recovery/#configuration) through policy or **Settings** > **System** > **Recovery** > **Quick machine recovery**. Enterprise policies and Windows Autopatch approvals can affect remediation delivery.
1. Inside the test VM, open an elevated command prompt and check the recovery settings:

   ```cmd
   reagentc.exe /info
   reagentc.exe /getrecoverysettings
   ```

1. Confirm that WinRE is enabled with a recovery location and that the intended QMR settings are present. Stop if a command fails or you can't verify the settings.

## Run the simulation

Run the following commands inside the test VM, not on your management workstation. Continue only after each command succeeds.

1. In an elevated command prompt, enable test mode:

   ```cmd
   reagentc.exe /SetRecoveryTestMode
   ```

1. Configure the next boot to enter WinRE:

   ```cmd
   reagentc.exe /BootToRe
   ```

1. Restart the guest after both commands succeed.
1. Watch Boot Diagnostics while QMR runs. Resolve any network-access issue before retrying. The following screenshot illustrates simulation progress.

   :::image type="content" source="media/quick-machine-recovery-boot-diagnostics.png" alt-text="Screenshot of Azure Boot diagnostics showing Hyper-V and the QMR message [Test Mode] Diagnosing your PC." lightbox="media/quick-machine-recovery-boot-diagnostics.png":::

1. Let recovery return the VM to Windows. Completion time varies.

If the VM enters ordinary WinRE or doesn't recover within your test window, preserve diagnostics and follow the [QMR test-mode troubleshooting guidance](/windows/configuration/quick-machine-recovery/#test-mode). Don't repeatedly restart it. Use your rollback plan if needed.

## Verify the result

Confirm Windows startup, Guest Agent health, guest access, and workload health. In **Settings** > **Windows Update** > **Update history** > **Quality updates**, find the **Quick machine recovery update** for this test.

Match the update entry to this test's time. For the meaning of platform and recovery signals, see [Interpret results](monitor-recovery.md#interpret-results).

## Collect diagnostics and feedback

For troubleshooting, retain Boot Diagnostics, Windows image and build details, Insider channel details, test times, command results, and the following logs:

```text
C:\Windows\System32\LogFiles\Srt\*
```

For offline collection, snapshot the OS disk, create a disk from the snapshot, and attach the copy as a data disk to a repair VM. Use its mapped drive letter, and leave the original disk unchanged.

Redact customer data and credentials before sharing diagnostics with support. To provide QMR feedback, use **Feedback Hub** > **Recovery and Uninstall** > **Quick Machine Recovery**.

## Clean up

Restore the baseline or recreate the test VM. Retain needed diagnostics, and then remove unneeded test VMs, disks, snapshots, and storage to avoid ongoing charges. Don't delete shared resources.

## Related content

- [Compare Microsoft and custom remediation sources](overview.md#remediation-sources).
- [Azure VM Boot Diagnostics](../../boot-diagnostics.md).
- [Windows QMR documentation](/windows/configuration/quick-machine-recovery/).
