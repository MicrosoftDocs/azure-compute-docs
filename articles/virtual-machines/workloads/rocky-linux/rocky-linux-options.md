---
title: Rocky Linux options on Azure
description: Understand the Rocky Linux options available on Azure, including Community Edition, CIQ Rocky Linux Plus, and CIQ Rocky Linux Pro.
author: karlabbott
ms.author: abbottkarl
ms.service: azure-virtual-machines
ms.subservice: vm-linux-setup-configuration
ms.custom: linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.date: 07/07/2026
ai-usage: ai-assisted
# Customer intent: "As a Rocky Linux user on Azure, I want to understand the differences between Community Edition, CIQ Rocky Linux Plus, and CIQ Rocky Linux Pro, so that I can choose the right variant for my workload requirements."
---

# Rocky Linux options on Azure

CIQ is the endorsed distribution publisher of Rocky Linux on Azure. As the founding sponsor of the Rocky Enterprise Software Foundation (RESF), CIQ publishes and supports Rocky Linux from CIQ in the Azure Marketplace, from a no-cost build to a paid enterprise product. The RESF publishes the community build independently but isn't part of the Azure endorsed distribution program. For background on the endorsed distribution program and the partners in it, see [Linux distributions endorsed on Azure](../../linux/endorsed-distros.md).

Three Rocky Linux options are commonly available on Azure Marketplace, each serving different operational needs. This article explains what each variant provides, which scenarios each is designed for, and how to move between them.

## Rocky Linux Community Edition (RESF)

Rocky Linux Community Edition is published to the Azure Marketplace by the Rocky Enterprise Software Foundation (RESF). It's the community-maintained distribution, freely available with no paid subscription required.

| Property | Value |
|---|---|
| Marketplace offer | [Rocky Linux 9 (RESF)](https://azuremarketplace.microsoft.com/marketplace/apps/resf.rockylinux-x86_64) |
| URN (x64) | `resf:rockylinux-x86_64:9-base:latest` |
| URN (Arm64) | `resf:rockylinux-aarch64:rockylinux-aarch64-9:latest` |

### What you get

- Community-built and community-maintained packages from RESF repositories
- Security updates and bug fixes delivered through community release cadences
- Support through community channels: forums, mailing lists, and chat platforms

### What it doesn't include

- Commercial support, SLAs, or guaranteed response times
- Vendor indemnification or compliance certifications
- Pre-integrated GPU drivers (NVIDIA and AMD drivers require manual installation and validation)
- A vendor escalation path

### When to use Rocky Linux Community Edition

- Development, testing, and proof-of-concept workloads
- Production workloads where your organization is self-supporting and doesn't require commercial SLAs
- Environments where you maintain in-house Linux expertise and don't need escalation paths to a vendor

### Considerations

When you run Rocky Linux Community Edition in production, your organization assumes full responsibility for issue resolution, security response, and lifecycle management. There's no contractual relationship with a vendor for updates, escalation, or legal protection. If your workloads require compliance certifications (such as FIPS 140-3), guaranteed patching timelines, or vendor-backed support, consider CIQ Rocky Linux Plus or CIQ Rocky Linux Pro.

Because Community Edition packages come from upstream sources and are rebuilt through community processes, Azure-specific kernel tunings - including enablement for accelerated networking, specific hardware generations, and new VM features - can take longer to appear in community repositories compared to CIQ offerings, which maintain direct integration with Azure platform requirements.

## CIQ Rocky Linux Plus (RLC+)

CIQ Rocky Linux Plus is a free Rocky Linux offering from CIQ, endorsed by Azure. It delivers a validated Rocky Linux experience with CIQ-built kernels and dedicated infrastructure.

| Property | Value |
|---|---|
| Marketplace offer | [CIQ Rocky Linux Plus](https://azuremarketplace.microsoft.com/marketplace/apps/ciq.rlc-plus) |
| URN (x64) | `ciq:rlc-plus:rocky9:latest` |
| URN (Arm64) | `ciq:rlc-plus:rocky9-arm64:latest` |
| URN (x64, NVIDIA) | `ciq:rlc-plus:rocky9-nvidia:latest` |
| URN (Arm64, NVIDIA) | `ciq:rlc-plus:rocky9-nvidia-arm64:latest` |

### What you get

- Azure endorsed distribution
- CIQ-built kernel validated for Azure, with timely integration of Azure-specific patches, platform tunings, and hardware enablement
- CIQ Core repositories with package signing and integrity verification for all packages
- Dedicated, US-based repository infrastructure
- Pre-integrated NVIDIA and AMD GPU drivers supporting enterprise-grade GPUs on Azure
- NVIDIA CUDA and DOCA-OFED redistributed and validated for shipped versions
- ROCm signed and validated for Azure AMD GPU machine shapes
- Subscription management through CIQ Depot CLI and the CIQ Portal
- Azure Marketplace deployment with no reinstall required from Community Edition

### When to use CIQ Rocky Linux Plus

- Production workloads that require a commercially validated distribution but don't need advanced compliance certifications or enterprise SLAs
- Workloads that depend on current Azure networking features, accelerated networking, or new VM hardware generations where community kernel cadence is insufficient
- Organizations moving from Community Edition that want a vendor relationship without the full enterprise tier
- GPU-accelerated workloads (AI/ML, HPC) that benefit from pre-integrated driver delivery
- Teams that want a clear upgrade path to CIQ Rocky Linux Pro as requirements evolve

## CIQ Rocky Linux Pro (RLC Pro)

CIQ Rocky Linux Pro is the enterprise offering. It includes everything in CIQ Rocky Linux Plus and adds long-term support, compliance certifications, indemnification, and SLA-backed professional support.

| Property | Value |
|---|---|
| Marketplace offer | [CIQ Rocky Linux Pro](https://azuremarketplace.microsoft.com/marketplace/apps/ciq.rlc-pro) |
| URN (x64) | `ciq:rlc-pro:rlc-pro-9-latest:latest` |
| URN (Arm64) | `ciq:rlc-pro:rlc-pro-9-latest-arm64:latest` |
| URN (x64, NVIDIA) | `ciq:rlc-pro:rlc-pro-9-latest-nvidia:latest` |
| URN (Arm64, NVIDIA) | `ciq:rlc-pro:rlc-pro-9-latest-nvidia-arm64:latest` |

### What you get (in addition to everything in CIQ Rocky Linux Plus)

- Long-term support (LTS): Security updates and fixes for a pinned minor version for up to three to five years, independent of upstream community release cadence
- Version pinning: Lock deployments to a specific Rocky Linux minor release (for example, 9.6) without forced upgrades
- FIPS 140-3 validated cryptographic modules for regulated environments
- Legal indemnification: Contractual protection for CIQ-packaged Rocky Linux components
- Priority bug and security fixes: Customer-driven issue resolution with guaranteed timelines
- Professional support with SLAs: Standard (business-day, one-hour Sev1 response) and Premium (24x7, 30-minute Sev1 response, named CSM)
- CIQ Rocky Linux Pro repositories: Additional performance, security, and compliance enhancements beyond Core

### When to use CIQ Rocky Linux Pro

- Mission-critical production workloads requiring vendor-backed SLAs and guaranteed response times
- Regulated environments (government, financial services, healthcare) that require FIPS 140-3 validated cryptography
- Organizations that need legal indemnification for their Linux layer
- Deployments requiring version stability and long-term support beyond community lifecycle timelines
- Enterprises that expect commercial-grade support guarantees for their Linux platform on Azure

## Feature comparison

| Capability | Community Edition (RESF) | CIQ Rocky Linux Plus | CIQ Rocky Linux Pro |
|---|---|---|---|
| Subscription cost | None | None | Paid |
| Azure Marketplace availability | Yes | Yes | Yes |
| Azure endorsed distribution | No | Yes | Yes |
| Azure platform kernel tunings | Community cadence | Current | Current |
| CIQ-built kernel | No | Yes | Yes |
| Dedicated repository infrastructure | No | Yes | Yes |
| GPU driver integration | Manual install | Pre-integrated | Pre-integrated |
| Long-term support (LTS) | No | No | Yes |
| Version pinning | No | No | Yes |
| FIPS 140-3 validated modules | No | No | Yes |
| Legal indemnification | No | No | Yes |
| Professional support with SLA | No | No | Yes |

## Moving between Rocky Linux variants

### From Community Edition to CIQ Rocky Linux Plus

You can convert an existing Rocky Linux Community Edition deployment to CIQ Rocky Linux Plus without reinstalling the operating system. The process registers the system with CIQ's subscription management and transitions package sources from RESF community repositories to CIQ Core repositories.

#### Prerequisites

- An active CIQ Rocky Linux Plus subscription (obtain through the Azure Marketplace or directly from CIQ)
- An account on the [CIQ Portal](https://portal.ciq.com)
- Root or sudo access on the target system

#### Steps

1. Sign in to the CIQ Portal and generate your subscription credentials.

1. Install the CIQ Depot CLI:

   ```bash
   curl -fsSL https://depot.ciq.com/install.sh | sudo bash
   ```

1. Authenticate the Depot CLI with your subscription credentials:

   ```bash
   sudo depot auth login
   ```

1. Enable CIQ Core repositories:

   ```bash
   sudo depot repo enable core
   ```

1. Run a full system update.

   ```bash
   sudo dnf update -y
   ```

No data migration or workload downtime is required for the conversion itself. After the update, your system runs the CIQ-built kernel with current Azure platform tunings.

### From CIQ Rocky Linux Plus to CIQ Rocky Linux Pro

Upgrading from CIQ Rocky Linux Plus to CIQ Rocky Linux Pro is a subscription-level change. Because CIQ Rocky Linux Pro builds on the same CIQ Core foundation, the transition adds access to CIQ Rocky Linux Pro repositories and support entitlements without reinstallation.

1. Upgrade your subscription to CIQ Rocky Linux Pro through the Azure Marketplace or your CIQ account.

1. Enable CIQ Rocky Linux Pro repositories:

   ```bash
   sudo depot repo enable pro
   ```

1. Apply any Pro-specific packages:

   ```bash
   sudo dnf update -y
   ```

1. Confirm SLA and support entitlements are active in the CIQ Portal.

### From CIQ Rocky Linux Plus or CIQ Rocky Linux Pro to Community Edition

You can move from a CIQ offering back to Community Edition. This change removes access to CIQ repositories and ends any associated support entitlements.

> [!NOTE]
> After reverting, the system gets updates from community repositories on the upstream release cadence. Azure-specific kernel tunings might take longer to appear. Make sure your organization is ready to self-support before making this change.

1. Cancel your CIQ subscription through the Azure Marketplace or CIQ Portal.

1. Remove CIQ repository configuration and Depot CLI tooling.

   ```bash
   sudo depot repo disable --all
   sudo dnf remove depot-cli -y
   ```

1. Re-enable RESF community repositories.

   ```bash
   sudo dnf install rocky-repos -y
   ```

1. Run a full system update.

   ```bash
   sudo dnf update -y
   ```

## Next steps

- [Deploy CIQ Rocky Linux Plus on Azure (Azure Marketplace)](https://azuremarketplace.microsoft.com/marketplace/apps/ciq.rlc-plus)
- [Deploy CIQ Rocky Linux Pro on Azure (Azure Marketplace)](https://azuremarketplace.microsoft.com/marketplace/apps/ciq.rlc-pro)
- [CIQ Portal — Getting Started](https://portal.ciq.com)
- [CIQ Depot CLI installation](https://docs.ciq.com/depot/depot-client/get-started/installation)
- [CIQ documentation](https://docs.ciq.com/rlc)
- [Rocky Linux community resources](https://rockylinux.org)
- [Linux distributions endorsed on Azure](../../linux/endorsed-distros.md)
