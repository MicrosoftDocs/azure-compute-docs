---
title: Deploy GPU-accelerated Linux virtual desktops with ThinLinc on Azure
description: Plan and deploy persistent, GPU-accelerated Linux virtual desktops on Azure by using ThinLinc and supported Azure GPU virtual machines.
ai-usage: ai-generated
author: padmalathas
ms.author: padmalathas
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.collection: linux
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 09/01/2026
# Customer intent: As an HPC administrator, I want to deploy GPU-accelerated Linux virtual desktops so that users can visualize data and use graphical engineering applications without moving data out of Azure.
---

# Deploy GPU-accelerated Linux virtual desktops with ThinLinc on Azure

**Applies to:** :heavy_check_mark: Linux VMs

Linux virtual desktop infrastructure (VDI) provides a persistent graphical desktop on a remote Linux virtual machine (VM). It can help researchers and engineers interact with graphical applications and visualize large datasets close to their Azure compute and storage resources.

ThinLinc is a Linux remote desktop product from Cendio. It supports persistent sessions, native clients for Windows, macOS, and Linux, and browser-based access. For graphics-intensive applications, ThinLinc can use VirtualGL to render OpenGL workloads on an Azure GPU and send the resulting images to the client.

> [!IMPORTANT]
> ThinLinc is a non-Microsoft product. This article describes Azure infrastructure considerations and links to Cendio documentation for product installation and configuration. Contact Cendio for support with ThinLinc software, licensing, and session configuration.

## Choose a deployment model

Use either a standalone VM or Azure CycleCloud Workspace for Slurm, depending on whether users need an individual workstation or access to a managed HPC environment.

| Requirement | Standalone VM | CycleCloud Workspace for Slurm |
| --- | --- | --- |
| Typical use | Individual workstation, small team, or proof of concept | Shared HPC environment with scheduled compute and interactive access |
| Scaling | Resize the VM or deploy more independently managed VMs | Add compute resources through Slurm and CycleCloud autoscaling |
| Storage | Managed disks or mounted shared storage | Shared home and project storage across access and compute nodes |
| User entry point | ThinLinc native client or web access | Open OnDemand with built-in ThinLinc integration |
| Operational effort | Lower infrastructure complexity; each VM requires lifecycle management | Centralized environment; requires CycleCloud and Slurm administration |

Choose a standalone VM when users need a persistent cloud workstation and don't need a scheduler. Choose CycleCloud Workspace for Slurm when interactive visualization is part of a larger workflow that submits jobs to a Slurm cluster.

> [!NOTE]
> ThinLinc licenses concurrent users within a ThinLinc instance or cluster. For deployments with multiple Marketplace images or CycleCloud nodes, don't assume that each VM or node provides a separate free license entitlement. Confirm the required license count with Cendio.

## Architecture

### Standalone VM

A standalone deployment uses one Linux VM for the ThinLinc master and agent components. The VM can also host VirtualGL and a supported NVIDIA GRID driver for GPU-accelerated visualization.

The deployment includes these components:

- A Linux VM on a [GPU-accelerated VM size](../sizes/overview.md#gpu-accelerated), such as an NVadsA10_v5 size.
- A virtual network and network security group (NSG) that restrict access to trusted source networks.
- ThinLinc server components and a supported Linux desktop environment.
- An NVIDIA GRID driver and VirtualGL for applications that require hardware-accelerated OpenGL rendering.
- Managed disks or shared storage for user and project data.
- A ThinLinc native client or supported web browser on the user device.

### CycleCloud Workspace for Slurm

[Azure CycleCloud Workspace for Slurm](/azure/cyclecloud/overview) provides a managed HPC environment with Slurm, autoscaling compute partitions, shared storage options, and Open OnDemand. ThinLinc integration with Open OnDemand is available in CycleCloud Workspace for Slurm version 2026.03.10 and later.

In this model, Open OnDemand provides the authenticated web entry point. Users start interactive sessions and submit compute jobs to Slurm. Use shared storage so that the interactive environment and compute nodes access the same home directories and workload data.

> [!NOTE]
> Use the built-in ThinLinc integration in a current CycleCloud Workspace for Slurm release. Don't use unvalidated third-party CycleCloud projects or cluster-init specifications as a substitute for the supported integration.

## Prerequisites

Before you deploy a standalone VM, verify that you have:

- An Azure subscription and permission to create VMs, networking resources, and disks.
- Sufficient regional quota for the selected GPU VM family. For more information, see [Check vCPU quotas](../quotas.md).
- A virtual network with private connectivity from user devices, such as a point-to-site VPN or ExpressRoute connection, for production deployments.
- A supported Linux distribution and NVIDIA GRID driver combination from [N-series GPU driver setup for Linux](n-series-driver-setup.md).
- A ThinLinc license appropriate for the number of concurrent users. The ThinLinc server bundle includes licenses for three concurrent users. You can download a Community License for up to 10 concurrent users in a ThinLinc instance or cluster. More than 10 concurrent users, or official production support, requires a paid subscription. See [ThinLinc free usage](https://www.cendio.com/thinlinc/buy-pricing/free-usage/).
- A supported ThinLinc client or web browser. See [Download the ThinLinc client](https://www.cendio.com/thinlinc/download/).

For CycleCloud Workspace for Slurm prerequisites, including network topology, identity, storage, and quota requirements, see [Plan your CycleCloud Workspace for Slurm deployment](/azure/cyclecloud/how-to/ccws/plan-your-deployment).

## Deploy a standalone Linux virtual desktop

### 1. Select a GPU VM size

For a new NVIDIA-based virtual workstation, consider an NVadsA10_v5 VM. It provides fractional NVIDIA A10 GPUs and supports Microsoft-redistributed NVIDIA GRID drivers on selected Linux distributions.

Select a size based on the graphics memory, CPU, memory, storage throughput, and number of concurrent sessions that your applications require. Confirm that the size is available in your target region and that your subscription has sufficient quota.

> [!NOTE]
> The ThinLinc image in Azure Marketplace supports NVIDIA GPUs only. It includes NVIDIA CUDA drivers, and VirtualGL is configured to detect CUDA devices. AMD CPU-based VMs can use the image, but AMD GPU acceleration isn't currently supported or validated for it. For a manual ThinLinc installation, confirm other GPU and driver combinations with Cendio.

> [!IMPORTANT]
> Don't use retired NV-series VMs for new deployments. NVv3 and NVv4 VMs retire on September 30, 2026. For current size and retirement information, see [GPU-accelerated VM sizes](../sizes/overview.md#gpu-accelerated).

### 2. Deploy and secure the Linux VM

Create the VM in a dedicated subnet. Use a Premium SSD for the operating system and add managed disks or shared storage based on application requirements.

For production environments:

- Use private IP access through a VPN or ExpressRoute connection.
- If you need a public IP for an evaluation, restrict inbound NSG rules to known source IP ranges and remove the public access when testing is complete.
- Open only the ports required for the ThinLinc access methods that you enable. Use the [ThinLinc network requirements](https://www.cendio.com/resources/docs/tag/network.html) and [TCP port reference](https://www.cendio.com/resources/docs/tag/tcp-ports.html) when you define NSG rules.
- Don't expose the ThinLinc web administration interface to the public internet.
- Use SSH public-key authentication for VM administration and apply least-privilege access.

For general deployment steps, see [Create a Linux VM in the Azure portal](quick-create-portal.md).

### 3. Install and verify the GPU driver

Install the GRID driver version that Microsoft lists for your VM size and Linux distribution. Follow [Install NVIDIA GPU drivers on N-series VMs running Linux](n-series-driver-setup.md) rather than downloading an unvalidated driver version.

After installation, restart the VM if the driver installation instructions require it, and then run:

```bash
nvidia-smi
```

Confirm that the command lists the expected GPU and doesn't report a driver communication error.

### 4. Install ThinLinc and VirtualGL

Install ThinLinc by following the [ThinLinc server installation documentation](https://www.cendio.com/resources/docs/tag/install_install.html). Configure a supported desktop environment and the authentication method required by your organization.

For GPU-accelerated OpenGL applications, follow Cendio's [3D acceleration guidance](https://www.cendio.com/resources/docs/tag/virtualgl.html) to install and configure VirtualGL. Restrict GPU device access to authorized users and groups.

Use a trusted certificate for browser-based access. For certificate configuration, see [ThinLinc Web Access certificates](https://www.cendio.com/resources/docs/tag/tlwebaccess_certificates.html).

### 5. Connect and validate the desktop

Connect by using the ThinLinc native client or Web Access. Verify the following behavior before onboarding users:

1. The user can authenticate and start a desktop session.
1. The session remains available after the client disconnects and reconnects.
1. The desktop can access the required project storage.
1. `nvidia-smi` identifies the expected GPU inside the session.
1. A representative OpenGL application uses the GPU when started through VirtualGL.
1. Clipboard, device redirection, and file-transfer settings follow your organization's data protection requirements.

Test with the actual engineering or visualization application. A simple graphics demonstration doesn't establish that an application is supported or performs adequately for a production workload.

## Use ThinLinc with CycleCloud Workspace for Slurm

CycleCloud Workspace for Slurm version 2026.03.10 introduced ThinLinc integration with Open OnDemand. Use this integration instead of adding an unverified ThinLinc CycleCloud project to a cluster template.

To prepare the environment:

1. Review the [CycleCloud Workspace for Slurm release notes](/azure/cyclecloud/release-notes/ccws/2026-03-10) and deploy a current supported release.
1. Follow the [CycleCloud Workspace for Slurm deployment quickstart](/azure/cyclecloud/qs-deploy-ccws). Enable Open OnDemand during deployment.
1. Configure shared home and project storage for interactive and compute nodes.
1. Register and configure the Microsoft Entra application required by Open OnDemand.
1. Follow [Configure Open OnDemand with CycleCloud](/azure/cyclecloud/how-to/ccws/configure-open-ondemand).
1. Validate that an authorized user can start an interactive desktop, access shared data, and submit a Slurm job.

Open OnDemand requires direct connectivity to the workspace virtual network. Azure Bastion tunneling isn't supported for the Open OnDemand access scenario. Use a point-to-site VPN or ExpressRoute connection as described in the CycleCloud Workspace for Slurm planning guidance.

## Security considerations

- Keep desktop and project data in Azure. Disable clipboard, drive, printer, or device redirection when those features conflict with data handling requirements.
- Restrict NSG rules to trusted networks and only the ThinLinc services that you use.
- Use a certificate issued by a trusted certificate authority for web access.
- Integrate authentication with your organization's identity controls. ThinLinc uses Linux authentication facilities, and Open OnDemand in CycleCloud Workspace for Slurm uses Microsoft Entra ID.
- Apply operating system, NVIDIA driver, ThinLinc, and application updates through a tested image lifecycle.
- Use Azure Compute Gallery image versions when you need repeatable deployment of multiple standalone VMs.
- Monitor VM availability, disk capacity, GPU utilization, sign-in activity, and ThinLinc license consumption.
- Review the [ThinLinc lockdown guidance](https://www.cendio.com/resources/docs/tag/lockdown.html) before enabling access for production users.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| The VM size isn't available | Confirm regional availability and quota for the exact GPU VM family. |
| `nvidia-smi` can't communicate with the driver | Confirm that the VM size, Linux distribution, and GRID driver version match the supported combinations. |
| 3D applications use software rendering | Confirm that VirtualGL is configured and that the application is started through VirtualGL. |
| The desktop is slow over a wide area network | Measure network latency and packet loss, reduce display resolution or quality for testing, and compare native-client and browser access. |
| A user can connect but can't access project data | Verify mount configuration, identity mapping, file permissions, and network access to shared storage. |
| Open OnDemand isn't reachable | Confirm private network connectivity and the Microsoft Entra application and OpenID Connect configuration. |

For product-specific logs and diagnostics, see [Troubleshooting ThinLinc](https://www.cendio.com/resources/docs/tag/troubleshoot.html).

## Next steps

- Review [N-series GPU driver setup for Linux](n-series-driver-setup.md).
- Review [Azure CycleCloud Workspace for Slurm](/azure/cyclecloud/overview).
- Plan repeatable images with [Azure Compute Gallery](../azure-compute-gallery.md).
- Review the [ThinLinc Administrator's Guide](https://www.cendio.com/resources/docs/tag/).