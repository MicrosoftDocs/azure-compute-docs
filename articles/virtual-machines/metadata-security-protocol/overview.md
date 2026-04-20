---
title: Metadata Security Protocol (MSP)
description: Get an overview of the Metadata Security Protocol feature.
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 09/04/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: As a cloud administrator, I want to implement the Metadata Security Protocol so that I can enhance the security of instance metadata services against attacks and protect sensitive VM credentials from potential threats.
---

# Metadata Security Protocol (MSP)

Metadata Security Protocol (MSP) is a feature in the Azure Virtual Machines and Azure Virtual Machine Scale Sets services. It enhances the security of the [Azure Instance Metadata Service](https://aka.ms/azureimds) and [WireServer](https://aka.ms/azureWireserver) services. These services are available in Azure infrastructure as a service (IaaS) virtual machines (VMs) or virtual machine scale sets at 169.254.169.254 and 168.63.129.16, respectively.

Organizations use Instance Metadata Service and WireServer for providing metadata and bootstrapping VM credentials. As a result, threat actors frequently target these services. Common vectors include confused deputy attacks against in-guest workloads and sandbox escapes. These vectors are of particular concern for hosted-on-behalf-of workloads where untrusted code loads into the VM.

With metadata services, the trust boundary is the VM itself. Any software within the guest is authorized to request secrets from Instance Metadata Service and WireServer. VM owners are responsible for carefully sandboxing any software that they run inside the VM and ensuring that external actors can't exfiltrate data. In practice, the complexity of the problem leads to mistakes at scale, which in turn lead to exploits.

Although numerous defense-in-depth strategies exist, providing secrets over an unauthenticated HTTP API carries inherent risk. Across the industry, this class of vulnerabilities has affected hundreds of companies, affected millions of individuals, and caused financial losses in the hundreds of millions of dollars.

MSP closes the most common vulnerabilities by:

- Addressing the root cause of these attacks.
- Introducing strong authentication and authorization concepts to cloud metadata services.

## Compatibility

MSP is supported on Azure IaaS VMs and virtual machine scale sets that run these operating systems:

- Windows 10 or later (x64)
- Windows Server 2019 (x64)
- Windows Server 2022 (x64)
- Windows Server 2025 (x64)
- Mariner 2.0 (x64, ARM64)
- Azure Linux (Mariner 3.0) (x64, ARM64)
- Ubuntu 20.04+ (x64, ARM64)

The following items are currently not supported:

- Red Hat Enterprise Linux 9+
- Rocky Linux 9+
- SUSE Linux Enterprise Server 15 SP4+
- Ephemeral disks
- Compatibility with Azure Backup
- ARM64

## Enhanced security

The Guest Proxy Agent (GPA) hardens against these types of attacks by:

- Limiting metadata access to a subset of the VM (applying the principle of least-privileged access).
- Switching from a *default-open* model to a *default-closed* model.

  For instance, with nested virtualization, a misconfigured L2 VM that has access to the L1 VM's vNIC can communicate with a metadata service as the L1. With the GPA, a misconfigured L2 could no longer gain access, because it would be unable to authenticate with the service.

At provisioning time, the metadata service establishes a trusted delegate within the guest (the GPA). A long-lived secret is negotiated to authenticate with the trusted delegate. The delegate must endorse all requests to the metadata service by using a [hash-based message authentication code (HMAC)](https://en.wikipedia.org/wiki/HMAC). The HMAC establishes a point-to-point trust relationship with strong authorization.

The GPA uses [eBPF](https://ebpf.io/what-is-ebpf/) to intercept HTTP requests to the metadata service. eBPF enables the GPA to verify the identity of the in-guest software that made the request. The GPA uses eBPF to intercept requests without requiring an extra kernel module. GPA uses this information to compare the identity of the client against an allowlist defined as a part of the VM model in Azure Resource Manager (ARM). The GPA then endorses requests by adding a signature header. So, you can enable the MSP feature on existing workloads without breaking changes.

By default, the existing authorization levels are enforced:

- Instance Metadata Service is open to all users.
- WireServer is root/admin only.

This restriction is currently accomplished with firewall rules in the guest. This is still a default-open mechanism. If that rule can be disabled or bypassed for any reason, the metadata service still accepts the request. The authorization mechanism enabled here is default-closed. Bypassing interception maliciously or by error doesn't grant access to the metadata service.

You can set up an advanced authorization configuration (that is, authorize specific in-guest processes and users to access only specific endpoints) by defining a custom allowlist with role-based access control (RBAC) semantics.

> [!WARNING]
> Please note that on Windows, the users can enable GuestProxyAgent (GPA) from an ARM template. However, in Linux it's a two-step process. The VM/VMSS is provisioned first and only then can the GPA be configured. 

## Related content

- [MSP feature configuration](./configuration.md)
- [Deploy a VM or virtual machine scale set with MSP](./greenfield.md)
- [Enable MSP on an existing VM or virtual machine scale set](./brownfield.md)
