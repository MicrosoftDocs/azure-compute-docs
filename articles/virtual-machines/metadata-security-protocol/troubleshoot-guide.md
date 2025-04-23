---
title: Troubleshooting guide
description: Troubleshooting guide
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Troubleshooting guide
This page provides some basic troubleshooting guide. If you're unable to resolve the issues encountered while using the Metadata Security Protocol (MSP) feature, contact our Support team. 

## New Virtual Machine (VM) or Virtual Machine Scale Sets deployment failures

### Windows

In Windows, the platform automatically installs the necessary components - GPA and eBPF for Windows. If the installation fails:

1. Ensure you're using a [supported OS version]() <- link to compatibility matrix here.
1. Ensure the failure code is [MSP related]() <- Link to possible errors
1. Retry the deployment. Transient failures are a part of cloud computing.

If the issue persists, contact support.

### Linux

Ensure that you're using a [valid image]()

- The GPA must be baked into your image in order to deploy with MSP enabled.
- Your cloud-init version must be a newer, MSP aware version. If it's not, a race condition occurs between it and the GPA.

The exact failure depends on if/how your image is misconfigured or if a platform failure occurred:

| GPA Installed | Cloud-init Version | Expected Failure | Cause | 
|--|--|--|--|
| No | `< 24.3`| `Linux.OSProvisioningInternalError`<br>Linux.cloud-init successfully reported ready for provisioning but the platform failed to record success | VM may fail to provision even though cloud-init reports ready.|
| No | `>= 24.3`| Linux.Cloud-Init failed due to azure-proxy-agent not found | Cloud-init reports failure to platform after detecting GPA not installed. |
| Yes | `< 24.3`| `Linux.OSProvisioningInternalError` | Cloud-init may report ready before GPA is configured as it is GPA-unaware. May fail up to 100% of the time depending on scenario. | 
| Yes | `>= 24.3`| Cloud-init reports GPA is unhealthy | Any of: eBPF setup failure, Cgroups v2 not enabled, generic startup failure, failure to acquire key |

## MSP enabled, but not applied in existing VM or Virtual Machine Scale Sets

To avoid service disruptions when enabling MSP on an existing VM or Virtual Machine Scale Sets, protections aren't applied until the VM indicates it successfully sets up and acquires the long-lived key. This delay means the VM model can show that MSP is enabled. However, GPA service may still be unhealthy and indicate that protections aren't activated.

### Confirming the issue still exists

Sometimes the delay is higher than anticipated. To begin, check the status report of the GPA service:

#### 1. Get status.json file: 

Windows VM:  
ProxyAgent logs from inside the VM as they have more detailed logs, the logs folder is `C:\WindowsAzure\ProxyAgent\Logs`

Status.json
ProxyAgent service captured its overall status into `C:\WindowsAzure\ProxyAgent\Logs\status.json`

Linux VM:
ProxyAgent logs from inside the VMs as they have more details, the logs folder is `/var/log/azure-proxy-agent/`

status.json
azure-proxy-agent service captures its overall status into `/var/log/azure-proxy-agent/status.json`

#### 2. Check `proxyAgentStatus` in  the json file and ensure it's "SUCCESS." If it's not "SUCCESS", check these other statuses:

|Status| Expected value|
|--|--|
|`keyLatchStatus`| "RUNNING" |
| `ebpfProgramStatus`| "RUNNING"|
|`proxyListenerStatus`|"RUNNING"|

### GPA service missing or not running
#### Windows VM
Control Resource Plane (CRP) implicitly installs these Windows services using Guest extensions:

- Windows Guest Proxy Agent (GPA) service (**GuestProxyAgent**)
- ebpf-for-windows kernel drivers: **eBPFCore**, **NetEbpfExt**

#### Linux VM
For Linux VMs, Linux Guest Proxy Agent (GPA) must be baked into its base image, or the user explicitly add GPA VM Extension `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` before enable MSP feature.

Prerequisites
- Linux Kernel 5.15 or later, which has all our required eBPF features;
- cgroup2 mounted by default, GPA service hooks up cgroup/connect4 eBPF event.

When enabling MSP, CRP installs one service `azure-proxy-agent` to the VM.
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
### GPA service is running, but unable to set up

#### eBPF setup failed

Windows VM

eBPFCore and NetEbpfExt service must be in `RUNNING` state,
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

Linux VM:

#### Failed to load eBPF program

GPA uses some eBPF features which requires Linux kernel 5.15 or later. If the customer is trying to enable the MSP feature on an unsupported Linux distro version, you may see the message similar to:

```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to load program 'connect4' with error: the BPF_PROG_LOAD syscall failed. Verifier output: ; int connect4(struct bpf_sock_addr *ctx)\n0: (bf) r6 = r1\n; __u64 cookie = bpf_get_socket_cookie(ctx);\n1: (85) call bpf_get_socket_cookie#46\n2: (b7) r1 = 0\n; destination_entry entry = {0};\n3: (63) *(u32 *)(r10 -44) = r1\nlast_idx 3 first_idx 0\nregs=2 stack=0 before 2: (b7) r1 = 0\n4: (63) *(u32 *)(r10 -48) = r1\n5: (63) *(u32 *)(r10 -52) = r1\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n6: (61) r1 = *(u32 *)(r6 +4)\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n7: (63) *(u32 *)(r10 -56) = r1\n; entry.destination_port = ctx->user_port;\n8: (61) r1 = *(u32 *)(r6 +24)\n; entry.destination_port = ctx->user_port;\n9: (63) *(u32 *)(r10 -40) = r1\n; entry.protocol = ctx->protocol;\n10: (61) r1 = *(u32 *)(r6 +36)\n; entry.protocol = ctx->protocol;\n11: (63) *(u32 *)(r10 -36) = r1\n12: (bf) r2 = r10\n; \n13: (07) r2 += -56\n; destination_entry *policy = bpf_map_lookup_elem(&policy_map, &entry);\n14: (18) r1 = 0xffff8baa17403400\n16: (85) cal..."
    },
```
Try get the kernel version using `uname -r`, if it displays lower than 5.15,  opt-out MSP feature, upgrade your OS image to latest version and enable MSP feature.

#### Failed to attach cgroup program for redirection

Guest ProxyAgent requires cgroup2 mounted by default to hooks up cgroup/connect4 eBPF event. If the customer trying to enable the MSP feature on an unsupported Linux distro version, you may see the message like:
- status.json
```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to attach program 'connect4' with error: `bpf_link_create` failed"
    },
```
- Check if current Linux VM supports CGroup2 using `grep cgroup /proc/filesystems`,
```
grep cgroup /proc/filesystems
``` 
If `nodev cgroup2` is listed, it means cgroup v2 is supported by this OS; if not listed, opt-out MSP feature, update to this latest OS version and then enable MSP feature.
- Check if CGroup2 is mounted by default using `mount | grep cgroup2`
```
mount | grep cgroup2
```
If there's no record displayed, ask the VM owner to set it up with command below and then reboot the VM.
```
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

#### Failed to acquire key / key rejected

The GPA wasn't able to acquire a key, because the platform is no longer offering it. Actions like migrating a disk to a new VM or deleting the VMs OS disk and replacing it can cause this failure. 

Follow the Key recovery / key reset instructions. Resetting the key allows the platform to offer a new one. The GPA periodically attempts to recover. Once the reset is completed, the GPA automatically acquires a new key and go back to a healthy state.

## Key recovery / key reset

If your VM's long-lived key is lost, it no longer communicates with Instance Metadata Service (IMDS) or Wireserver. Without the key, a new one can't be safely issued to the VM automatically. Additionally, if the key is compromised in some way it must be reset to ensure security is maintained.

### Resetting a Key

1. Refer to [Key Reset](other-examples/key-reset.md) for complete documentation on the `KeyIncarnationId` filed of the `ProxyAgentSettings` section of the VM model.
1. Select a new `KeyIncarnationId` value, and apply it to the VM model with a `PUT` operation

> [!Note]
> If sending multiple requests or otherwise using automation you must ensure that the values you send are strictly monotonically increasing or else no change may be applied. 
