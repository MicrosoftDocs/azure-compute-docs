---
title: Metadata Security Protocol (MSP)
description: Overview of metadata security protocol.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/20/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Metadata Security Protocol (MSP)

MSP enhances the security of the [Azure Instance Metadata Service](https://aka.ms/azureimds) and [Azure WireServer](https://aka.ms/azureWireserver) services (available in Azure IaaS Virtual Machine (VM) or Virtual Machine Scale Sets at 169.254.169.254 and 168.63.129.16 respectively). These services are used for providing metadata and bootstrapping VM credentials. As a result, threat actors frequently target these services. Common vectors include confused deputy attacks against in-guest workloads and sandbox escapes. These vectors are of particular concern for hosted-on-behalf-of workloads where untrusted code loads into the VM.

With metadata services, the trust boundary is the VM itself. Any software within the guest is authorized to request secrets from Instance Metadata Service (IMDS) + WireServer. VM owners are responsible for carefully sandboxing any software they run inside the VM and ensuring that external actors can't exfiltrate data. In practice, the complexity of the problem leads to mistakes at scale which in turn lead to exploits.

While numerous defense-in-depth strategies exist, providing secrets over an unauthenticated HTTP API carries inherent risk. Across the industry, this class of vulnerabilities impacted hundreds of companies, millions of individuals, and caused financial losses in the hundreds of millions of dollars. MSP closes most common vulnerabilities by addressing the root cause of these attacks and by introducing strong Authentication (AuthN)/Authorization (AuthZ) concepts to cloud metadata services.

## Compatibility

MSP is supported on Azure IaaS VMs + Virtual Machine Scale Sets running OS based on:

- Windows 10 or later (x64)
- Windows Server 2019 (x64)
- Windows Server 2022 (x64)
- Windows Server 2025 (x64)
- Mariner 2.0 (x64, ARM64)
- Azure Linux(Mariner 3.0) (x64, ARM64)
- Ubuntu 20.04+ (x64, ARM64)

> Once MSP is Generally Available, the list will be expanded to include:
> - Redhat 9+
> - Rocky-Linux9+
> - SUSE 15 SP4+

The following are not supported yet:
> - Ephemeral Disks 
> - Compatability with Azure Backup 
> - ARM64 

## How to Configure MSP

### Register the feature flags

To use MSP in preview, register the following flag using the `az feature register` command.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name ProxyAgentPreview
```

Verify the registration status by using the `az feature show` command. It takes a few minutes for the status to show *Registered*:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name ProxyAgentPreview
```

When the status reflects *Registered*, refresh the registration of the *Microsoft.Compute* resource provider by using the `az provider register` command:

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```
Once you are registered in the feature flag you can configure MSP via: 

- [ARM Templates](./other-examples/arm-templates.md)
- REST API 
- PowerShell
- [Azure portal](./other-examples/portal.md)

For details on how to configure MSP, visit  [MSP Configuration](./configuration.md) page. 

## Enhanced Security

### Implementation

The GPA hardens against these types of attacks by:

- Limiting metadata access to a subset of the VM (applying the principle of least privileged access).
- Switching from a "default-open" to "default-closed" model. For instance, with nested virtualization, a misconfigured L2
  VM that has access to the L1 VM's vNIC can communicate with a metadata service as the L1. With the GPA, a misconfigured
  L2 would no longer be able to gain access, as it would be unable to authenticate with the service.

At provisioning time, the metadata service establishes a trusted delegate within the guest (the GPA). A long-lived
secret is negotiated to authenticate with the trusted delegate, and the delegate must endorse all requests to the metadata service using a hash-based message authentication code (HMAC) ([HMAC](https://en.wikipedia.org/wiki/HMAC)). The HMAC establishes a point-to-point trust
relationship with strong AuthN.

The GPA uses [eBPF](https://ebpf.io/what-is-ebpf/) to intercept HTTP requests to the metadata services. eBPF
enables the GPA to verify the identity of the in-guest software that made the request. The GPA uses eBPF to intercept requests without requiring an extra kernel module. GPA uses this information to compare the identity of the client against an allowlist defined as a part of the VM model in the Azure Resource Manager (ARM). The GPA then endorses requests  by adding a signature header. Therefore, the MSP feature can be enabled on existing workloads without breaking changes.

- By default, the existing authorization levels are enforced: IMDS is open to all users and Wireserver is root / admin only.
  - Today this restriction is accomplished with firewall rules in the guest. This is still a default-open mechanism, because if that rule can be disabled or bypassed for any reason the metadata service still accepts the request. The AuthN mechanism enabled here default-closed. Bypassing interception maliciously or by error doesn't grant access to the metadata service.
- Advanced AuthZ configuration to authorize specific in-guest processes and users to access only specific endpoints is supported by defining a custom allowlist with Role Based Access Control (RBAC) semantics.

## Getting Audit Logs

In `Audit` and `Enforce` modes, audit logs are generated on the local disk.

| OS Family | Audit Log Location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |


## Related Content

- [MSP Configuration](./configuration.md)
- [Deploy a VM or Virtual Machine Scale Sets With MSP](./greenfield.md)
- [Enable MSP on Existing VM or Virtual Machine Scale Sets](./brownfield.md)
