---
title: Troubleshoot MSP
description: Get troubleshooting guidance for using Metadata Security Protocol (MSP).
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: troubleshooting-general
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Troubleshoot MSP

This article provides some basic troubleshooting guidance for using the Metadata Security Protocol (MSP) feature with virtual machines (VMs) or virtual machine scale sets. If you can't resolve your problem by using this guidance, contact the support team.

## Deployment failed for a new VM or virtual machine scale set

### Windows

In Windows, the platform automatically installs the necessary components: Guest Proxy Agent (GPA) and eBPF. If the installation fails:

1. Ensure that you're using a supported OS version.
1. Ensure that the failure code is MSP related.
1. Retry the deployment. Transient failures are a part of cloud computing.

### Linux

Ensure that you're using a valid image:

- The GPA must be included in your image. That's a requirement for deploying with MSP enabled.
- Your cloud-init version must be a newer, MSP-aware version. If it's not, a race condition occurs between it and the GPA.

The exact failure depends on whether your image is misconfigured or a platform failure occurred:

| GPA installed | Cloud-init version | Expected failure | Cause |
|--|--|--|--|
| No | Earlier than 24.3| `Linux.OSProvisioningInternalError`<br><br>`Linux.Cloud-Init` successfully reported ready for provisioning, but the platform failed to record success. | VM might fail provisioning even though cloud-init reports ready.|
| No | 24.3 or later| `Linux.Cloud-Init` failed because `azure-proxy-agent` is not found. | Cloud-init reports failure to the platform after detecting that the GPA is not installed. |
| Yes | Earlier than 24.3| `Linux.OSProvisioningInternalError` | Cloud-init might report ready before the GPA is configured, because it's GPA-unaware. Failures might happen up to 100% of the time, depending on the scenario. |
| Yes | 24.3 or later| Cloud-init reports that the GPA is unhealthy. | Any of these items could be the cause: eBPF setup failure, failure to enable cgroup v2, generic startup failure, failure to acquire a key. |

## MSP is enabled but not applied

To avoid service disruptions when you're enabling MSP on an existing VM or virtual machine scale set, protections aren't applied until the VM indicates that it successfully set up and acquired the long-lived key. This delay means the VM model can show that MSP is enabled. However, the GPA might still be unhealthy and indicate that protections aren't activated.

### Confirm that the problem still exists

Sometimes the delay is longer than anticipated. To begin, check the status report of the GPA:

1. Get the `status.json` file:

   - For a Windows VM, the ProxyAgent service logs from inside the VMs because they have more details. The log folder is `C:\WindowsAzure\ProxyAgent\Logs`. ProxyAgent captures its overall status in `C:\WindowsAzure\ProxyAgent\Logs\status.json`.

   - For a Linux VM, the ProxyAgent service (`azure-proxy-agent`) logs from inside the VMs because they have more details. The log folder is `/var/log/azure-proxy-agent/`. ProxyAgent captures its overall status in `/var/log/azure-proxy-agent/status.json`.

2. Check `proxyAgentStatus` in the `status.json` file and ensure that the value is `SUCCESS`. If it's not `SUCCESS`, check these other statuses:

   |Status| Expected value|
   |--|--|
   |`keyLatchStatus`| `RUNNING` |
   |`ebpfProgramStatus`| `RUNNING`|
   |`proxyListenerStatus`|`RUNNING`|

### GPA is missing or not running

#### Windows VM

Control Resource Plane (CRP) implicitly installs these Windows services by using guest extensions:

- GPA: `GuestProxyAgent`
- Kernel drivers for eBPF: `eBPFCore`, `NetEbpfExt`

#### Linux VM

For Linux VMs, the GPA must be included in the base image. Or you can explicitly add the GPA VM extension `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` before you enable the MSP feature.

Prerequisites are:

- Linux kernel 5.15 or later, which has all the required eBPF features.
- The cgroup v2 feature mounted by default. The GPA hooks up the cgroup/connect4 eBPF event.

When you enable MSP, CRP installs one service (`azure-proxy-agent`) in the VM.

```
# systemctl status azure-proxy-agent.service 
● azure-proxy-agent.service - Microsoft Azure GuestProxyAgent
     Loaded: loaded (/lib/systemd/system/azure-proxy-agent.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-09-13 18:52:00 UTC; 1 week 5 days ago
   Main PID: 4040671 (azure-proxy-age)
      Tasks: 10 (limit: 19169)
     Memory: 491.2M
        CPU: 1d 8h 13min 2.321s
     CGroup: /system.slice/azure-proxy-agent.service
             └─4040671 /usr/sbin/azure-proxy-agent
```

### GPA is running, but you can't set it up

#### eBPF setup failure

For eBPF setup to be successful, `eBPFCore` and `NetEbpfExt` must be in a `RUNNING` state.

Here's the code that shows the `RUNNING` state for a Windows VM:

```
c:\>sc query eBPFCore

SERVICE_NAME: eBPFCore
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Here's the code that shows the `RUNNING` state for a Linux VM:

```
c:\>sc query NetEbpfExt

SERVICE_NAME: NetEbpfExt
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

#### Failure to load eBPF

The GPA uses some eBPF features that require Linux kernel 5.15 or later. If you're trying to enable the MSP feature on an unsupported Linux distribution, you might see a message similar to this example:

```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to load program 'connect4' with error: the BPF_PROG_LOAD syscall failed. Verifier output: ; int connect4(struct bpf_sock_addr *ctx)\n0: (bf) r6 = r1\n; __u64 cookie = bpf_get_socket_cookie(ctx);\n1: (85) call bpf_get_socket_cookie#46\n2: (b7) r1 = 0\n; destination_entry entry = {0};\n3: (63) *(u32 *)(r10 -44) = r1\nlast_idx 3 first_idx 0\nregs=2 stack=0 before 2: (b7) r1 = 0\n4: (63) *(u32 *)(r10 -48) = r1\n5: (63) *(u32 *)(r10 -52) = r1\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n6: (61) r1 = *(u32 *)(r6 +4)\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n7: (63) *(u32 *)(r10 -56) = r1\n; entry.destination_port = ctx->user_port;\n8: (61) r1 = *(u32 *)(r6 +24)\n; entry.destination_port = ctx->user_port;\n9: (63) *(u32 *)(r10 -40) = r1\n; entry.protocol = ctx->protocol;\n10: (61) r1 = *(u32 *)(r6 +36)\n; entry.protocol = ctx->protocol;\n11: (63) *(u32 *)(r10 -36) = r1\n12: (bf) r2 = r10\n; \n13: (07) r2 += -56\n; destination_entry *policy = bpf_map_lookup_elem(&policy_map, &entry);\n14: (18) r1 = 0xffff8baa17403400\n16: (85) cal..."
    },
```

Try get the kernel version by using `uname -r`. If the version is earlier than 5.15, opt out of the MSP feature, upgrade your OS image to the latest version, and then enable MSP.

#### Failure to attach cgroup for redirection

`GuestProxyAgent` requires cgroup v2 mounted by default to hook up cgroup/connect4 eBPF events. If you try to enable the MSP feature on an unsupported Linux distribution, you might see a message like the following `status.json` example:

```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to attach program 'connect4' with error: `bpf_link_create` failed"
    },
```

To resolve this problem, try these methods:

- Check if the current Linux VM supports cgroup v2 by using this command:

  ```
  grep cgroup /proc/filesystems
  ```

  If `nodev cgroup2` is listed, this OS supports cgroup v2. If `nodev cgroup2` isn't listed, opt out of the MSP feature, update to the latest OS version, and then enable MSP.

- Check if cgroup v2 is mounted by default. Use the following command:

  ```
  mount | grep cgroup2
  ```

  If no record appears, ask the VM owner to set it up by using the following command and then restart the VM:

  ```
  sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
  ```

#### Failure to acquire a key or rejected key

The GPA can't acquire a key if the platform no longer offers it. Actions like migrating a disk to a new VM, or deleting the VM's OS disk and replacing it, can cause this failure.

Follow the [instructions for resetting a key](#you-need-to-reset-a-key) in this article. Resetting the key allows the platform to offer a new one. The GPA periodically attempts to recover. After you complete the reset, the GPA automatically acquires a new key and goes back to a healthy state.

## You need to reset a key

If your VM's long-lived key is lost, it no longer communicates with Azure Instance Metadata Service or WireServer. Without the key, a new one can't be safely issued to the VM automatically. Additionally, if the key is compromised in some way, it must be reset to maintain security.

To reset a key:

1. Refer to [Reset a latched key](other-examples/key-reset.md) for complete documentation on the `KeyIncarnationId` field of the `ProxyAgentSettings` section of the VM model.
1. Select a new `KeyIncarnationId` value, and apply it to the VM model by using a `PUT` operation.

> [!NOTE]
> If you're sending multiple requests or otherwise using automation, you must ensure that the values you send are strictly monotonically increasing. Otherwise, the change might not be applied.

## You got an error code

|Error code | Error message | Action |
|--|--|--|
| `ProxyAgentNotSupportedInRegion` |"Creation of VMs or Virtual Machine Scale Sets with ProxyAgent feature isn't supported in this region." |Choose a supported region. |
| `SubscriptionNotEnabledForProxyAgentFeature` | "The subscription is not registered for the private preview of ProxyAgent feature." | [Register the feature flag](/azure/virtual-machines/metadata-security-protocol/overview#register-the-feature-flags). |
| `BadRequest` | "The property `securityProfile.proxyAgentSettings.wireServer.inVMAccessControlProfileReferenceId` can't be used together with property `securityProfile.proxyAgentSettings.wireServer.mode'`" <br><br>"The property `securityProfile.proxyAgentSettings.imds.inVMAccessControlProfileReferenceId` can't be used together with property `securityProfile.proxyAgentSettings.imds.mode`." | Fix the parameter. |
| `BadRequest` | "The value `securityProfile.proxyAgentSettings.keyIncarnationId` can only be incremented." | Fix the parameter. |
| `BadRequest` | "The value of parameter `securityProfile.proxyAgentSettings.wireServer.mode` is invalid." <br><br>"The value of parameter `securityProfile.proxyAgentSettings.imds.mode` is invalid." | Provide a valid value: `Audit`, `Enable`, `Disabled`. |
| `InvalidParameter` | "The resource id '{0}' isn't a valid gallery `inVMAccessControlProfile` reference. A gallery `inVMAccessControlProfile` reference should be a valid resource identifier, of the format: '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profileName}/versions/{version}'." | Fix the parameter value. |
| `BadRequest`/`GalleryInVMAccessControlProfileNotMatchOSDisk` | "Current Gallery `InVMAccessControlProfile` Version {0} supports OS {1}, while current OSDisk's OS is {2}." | Fix the parameter value. |
| `BadRequest`/`GalleryInVMAccessControlProfileNotMatchHostEndpointType` | "Current Gallery `InVMAccessControlProfile` Version {0} is for host endpoint {1}, while current referred by {2}." | Fix the parameter value. |
| `InVMAccessControlProfileNotFound` | "The gallery `InVMAccessControlProfile` '{0}' isn't available. Verify that the `InVMAccessControlProfileReferenceId` passed in is correct." | Check that the profile exists and is replicated in the regions where the VM or virtual machine scale set exists. |
| `InVMAccessControlProfileNotFound` | "Failed to prepare the `InVMAccessControlProfile` '{0}' metadata for one or more resources due to an error: '{1}'." | Create a new profile and start over. |
