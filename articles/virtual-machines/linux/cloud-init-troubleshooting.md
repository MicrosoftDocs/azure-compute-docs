---
title: Troubleshoot using cloud-init
description: Troubleshoot provisioning an Azure VM using cloud-init.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.topic: troubleshooting
ms.date: 12/15/2025
ms.author: mattmcinnes
ms.reviewer: cynthn
ms.subservice: cloud-init
ms.custom: linux-related-content
# Customer intent: As a cloud administrator, I want to troubleshoot VM provisioning failures using cloud-init, so that I can ensure successful deployment and configuration of Linux VMs in the cloud.
---

# Troubleshooting VM provisioning with cloud-init

> [!CAUTION]
> This article references CentOS, a Linux distribution that is End Of Support (EOS) status. Consider your use and plan accordingly. For more information, see the [CentOS End Of Life guidance](~/articles/virtual-machines/workloads/centos/centos-end-of-life.md).

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

If you create generalized custom images and use cloud-init for provisioning, the VM might not build correctly. In that case, troubleshoot the image to find the issue.

Some examples, of issues with provisioning:

- The Compute Resource Provider API returns an error, and cloud-init reports the resulting failure.
- VM gets stuck at 'creating' for 40 minutes, and the VM creation is marked as failed.
- [Custom data](/azure/virtual-machines/custom-data) or [User data](/azure/virtual-machines/user-data) doesn't get processed.
- The ephemeral disk fails to mount (for VM skus that come with SCSI resource disks).
- Users don't get created, or there are user access issues.
- Networking isn't set up correctly.
- Swap file or partition failures.

This article steps you through how to troubleshoot cloud-init. For more in-depth details, see [cloud-init deep dive](./cloud-init-deep-dive.md).

## Troubleshooting failures reported by cloud-init and logged as error

Cloud-init emits structured errors when reporting failure to Azure during provisioning. These error messages include a reason and supporting data (such as timestamp, VM identifier, documentation URL, etc.) to help investigate the failure.

| Reason | Description | Action |
|:---|:---|:---|
| failure to find DHCP interface | No network interface was found. | Delete and re-create VM. If issue persists, ensure networking drivers or Azure-specific kernel is installed and check boot diagnostics to verify eth0 is enumerated. |
| failure to obtain DHCP lease | DHCP service fails to respond due to transient platform issue. | Redeploy the VM (delete + recreate) due to a transient Azure DHCP platform issue.  |
| failure to find primary DHCP interface | Primary DHCP interface wasn't found. | Check boot diagnostics to ensure primary network interface is named `eth0` and it isn't renamed. |
| connection timeout querying IMDS | Connections to IMDS may timeout due to transient platform issue, NSG, or OS firewall configuration. | Delete and re-create VM. If issue persists, validate that NSG or OS firewall isn't preventing access to IMDS.  |
| read timeout querying IMDS | Connections to IMDS may timeout due to transient platform issue or OS firewall configuration. | Delete and re-create VM. If issue persists, validate OS firewall isn't preventing access to IMDS. |
| unexpected metadata parsing ovf-env.xml | Malformed VM metadata in `ovf-env.xml`. | Submit the issue to the [cloud-init tracker.](https://github.com/canonical/cloud-init/issues/) |
| error waiting for host shutdown | Failure during host shutdown handling. | Submit the issue to the [cloud-init tracker.](https://github.com/canonical/cloud-init/issues/) |
| Azure-proxy-agent not found | The `azure-proxy-agent` binary is missing. | Ensure Azure proxy agent is installed in the image. For more troubleshooting, check out [MSP troubleshooting guide](/azure/virtual-machines/metadata-security-protocol/troubleshoot-guide). |
| Azure-proxy-agent status failure | Proxy agent reported a status error. | Review proxy agent logs and update if needed. For more troubleshooting, check out [MSP troubleshooting guide](https://learn.microsoft.com/azure/virtual-machines/metadata-security-protocol/troubleshoot-guide/). |
| unhandled exception | An unexpected error occurred inside cloud-init. | Submit the issue to the [cloud-init tracker.](https://github.com/canonical/cloud-init/issues/) |

For help with enabling and checking boot diagnostics, see [Boot Diagnostics](/azure/virtual-machines/boot-diagnostics).

If any of these issues persist on subsequent attempts at provisioning, it's due to a misconfiguration in the image. If there's reason to believe there's a cloud-init issue, report it to [cloud-init GitHub issue tracker](https://github.com/canonical/cloud-init/issues/).

## Troubleshooting other failures unreported by cloud-init

Depending on the failure, consider these steps.

### <a id="step1"></a> Step 1: Test the deployment without `customData`

Cloud-init can accept `customData` that is passed to it, when the VM is created. First, you should ensure this configuration isn't causing any issues with deployments. Some common issues can include:
- [The YAML is malformed](https://cloudinit.readthedocs.io/en/latest/howto/debug_user_data.html)
- Scripts inside customData fail or hang
- Override something cloud‑init relies on (for example, disk setup or networking)
- The data contains unsupported characters or encoding problems
Try to provisioning the VM without passing in any configuration. If the VM fails to provision, follow the recommended troubleshooting steps. If the configuration isn't applied, refer [step 4](#step4).

### <a id="step2"></a> Step 2: Review image requirements

The primary cause of VM provisioning failure is the OS image doesn't satisfy the prerequisites for running on Azure. Make sure your images are properly prepared before attempting to provision them in Azure.

The following articles illustrate the steps to prepare various linux distributions that are supported in Azure:

- [CentOS-based Distributions](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES & openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)
- [Others: Non-Endorsed Distributions](create-upload-generic.md)

For the [supported Azure cloud-init images](./using-cloud-init.md), the Linux distributions already have all the required packages and configurations in place to correctly provision the image in Azure. If you find your VM is failing to create from your own curated image, try a supported Azure Marketplace image that already is configured for cloud-init, with your optional `customData`. If the `customData` works correctly with an Azure Marketplace image, then there's probably an issue with your curated image.

### <a id="step3"></a> Step 3: Collect & review VM logs

When the VM fails to provision, Azure shows 'creating' status, for 20 minutes, and then reboot the VM, and wait another 20 minutes before finally marking the VM deployment as failed, before finally marking it with an `OSProvisioningTimedOut` error.

While the VM is running, you need the logs from the VM to understand why provisioning failed. To understand why VM provisioning failed, don't stop the VM. Keep the VM running. You need to keep the failed VM in a running state in order to collect logs. To collect the logs, use one of the following methods:

- [Enable Boot Diagnostics](/previous-versions/azure/virtual-machines/linux/tutorial-monitor#enable-boot-diagnostics) before creating the VM and then [View](/previous-versions/azure/virtual-machines/linux/tutorial-monitor#view-boot-diagnostics) them during the boot.

- [Serial Console](/troubleshoot/azure/virtual-machines/serial-console-grub-single-user-mode)
- [Run AZ VM Repair](/troubleshoot/azure/virtual-machines/repair-linux-vm-using-azure-virtual-machine-repair-commands) to attach and mount the OS disk ([*lvm*](/troubleshoot/azure/virtual-machines/linux/troubleshoot-recovery-disks-portal-linux#mount-the-attached-data-disk), [*no lvm*](/troubleshoot/azure/virtual-machines/linux/troubleshoot-recovery-disks-portal-linux#mount-the-attached-data-disk)), which will allow you to collect and examine these logs:

   ```bash
   /rescue/var/log/waagent*
   /rescue/var/log/syslog*
   /rescue/var/log/rsyslog*
   /rescue/var/log/messages*
   /rescue/var/log/kern*
   /rescue/var/log/dmesg*
   /rescue/var/log/boot*
   /rescue/ /var/log/cloud-init.log
   /rescue//var/log/cloud-init-output.log
   ```

> [!NOTE]
> Alternatively, you can create a rescue VM manually by using the Azure portal. For more information, see [Troubleshoot a Linux VM by attaching the OS disk to a recovery VM using the Azure portal](/troubleshoot/azure/virtual-machines/troubleshoot-recovery-disks-portal-linux).
To start initial troubleshooting, begin with the serial logs and cloud-init logs to understand where the failure occurred. Then use the other logs for a  deeper dive to help provide additional insights.

* /var/log/cloud-init.log
* /var/log/cloud-init-output.log
* [Serial/boot logs](/azure/virtual-machines/boot-diagnostics#boot-diagnostics-view)

In all logs, start searching for "Failed," "WARNING," "WARN," "err," "error," and "ERROR." Setting configuration to ignore case-sensitive searches is recommended.

Alternatively, use command `cloud‑init collect‑logs` to collect all necessary logs.
Azure’s latest cloud-init versions (≥ 18.2) include the collect‑logs command, which:

Gathers essential logs: /var/log/cloud-init*.log, instance metadata, system info.

Packages everything into a timestamped .tar.gz archive.

Saves the archive locally (for example, `/tmp/cloud-init-logs-timestamp.tar.gz`).

> [!TIP]
> If you're troubleshooting a custom image, you should consider adding a user during the image. If the provisioning fails to set the admin user, you can still log in to the OS.

#### Analyzing the logs

Here are more details about what to look for in each cloud-init log.

#### /var/log/cloud-init.log

By default, all cloud-init events with a priority of debug or higher, are written to `/var/log/cloud-init.log`. This log provides verbose logs of every event that occurred during cloud-init initialization.

For example:

```console
2019-10-10 04:51:25,321 - util.py[DEBUG]: Failed mount of '/dev/sr0' as 'auto': Unexpected error while running command.
Command: ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpLIrklc']
Exit code: 32
Reason: -
Stdout:
Stderr: mount: unknown filesystem type 'udf'
2020-01-31 00:21:53,352 - DataSourceAzure.py[WARNING]: /dev/sr0 was not mountable
```

When you find an error or warning, read backwards in the cloud-init log to understand what cloud-init was attempting before it hit the error or warning. In many cases, cloud-init runs OS commands or performs provisioning steps before the error occurs. These actions can help explain why the error appears in the logs. The following example shows that cloud-init attempted to mount a device right before it encountered the issue.

```output
2019-10-10 04:51:24,010 - util.py[DEBUG]: Running command ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpXXXXX'] with allowed return codes [0] (shell=False, capture=True)
```

If you have access to the [Serial Console](/troubleshoot/azure/virtual-machines/serial-console-grub-single-user-mode), you can try to rerun the command that cloud-init was trying to run.

The logging for `/var/log/cloud-init.log` can also be reconfigured within /etc/cloud/cloud.cfg.d/05_logging.cfg. For more information of cloud-init logging, see the [cloud-init documentation](https://cloudinit.readthedocs.io/en/latest/development/logging.html).

#### /var/log/cloud-init-output.log

You can get information from the `stdout` and `stderr` during the [stages of cloud-init](cloud-init-deep-dive.md). This data normally involves routing table information, networking information, ssh host key verification information, `stdout,` and `stderr` for each stage of cloud-init, along with the timestamps. If desired, `stderr` and `stdout` logging can be reconfigured from `/etc/cloud/cloud.cfg.d/05_logging.cfg`.


#### Serial/boot logs

Cloud-init has multiple dependencies. These dependencies are documented in the required prerequisites for images on Azure, such as networking, storage, the ability to mount an ISO, and to mount and format the temporary disk. Any of these dependencies may throw errors and cause cloud-init to fail. For example, If the VM can't get a DHCP lease, cloud-init fails.

If you still can't isolate why cloud-init failed to provision then you need to understand what cloud-init stages, and when modules run. For more information, see [Diving deeper into cloud-init](cloud-init-deep-dive.md).

### <a id="step4"></a> Step 4: Investigate why the configuration isn't being applied

Not every failure in cloud-init results in a fatal provisioning failure. For example, if you use the `runcmd` module in a cloud-init config, a nonzero exit code from the command causes the VM provisioning to fail. This behavior occurs because the module runs after the core provisioning steps in the first three stages of cloud-init. To troubleshoot why the configuration didn't apply, review the logs in Step 3, and cloud-init modules manually. For example:

- `runcmd` - do the scripts run without errors? To ensure they run as expected, run the configuration manually from the terminal.
- Installing packages - does the VM have access to package repositories?
- Check the `customData` configuration that was provided to the VM. This file is located in `/var/lib/cloud/instances/<unique-instance-identifier>/user-data.txt`.

## Next steps

If cloud-init skips the configuration, examine each cloud-init stage and the timing of module execution to identify the cause. For more information, see [Diving deeper into cloud-init configuration](./cloud-init-deep-dive.md).
