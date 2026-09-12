---
title: Linux VDI Options
description: Linux VDI on Azure supports remote desktop, GPU visualization, and brokered sessions. Compare architectures and choose an option for your users.
ai-usage: ai-assisted
author: padmalathas
ms.author: padmalathas
ms.date: 09/10/2026
ms.topic: concept-article
ms.service: azure-virtual-machines
ms.collection: linux
ms.custom: linux-related-content
# Customer intent: As a cloud architect, I want to compare graphical Linux desktop options on Azure so that I can select an architecture that meets user, performance, and management requirements.
---

# Linux remote desktop and VDI options on Azure

Linux virtual desktop infrastructure (VDI) on Azure is a set of architectures for delivering graphical Linux desktops from Azure virtual machines (VMs). Depending on the architecture, users can connect to a specific VM, use GPU-accelerated applications, or receive a brokered session from a shared environment.

Azure doesn't offer a first-party managed VDI service with Linux session hosts. Azure Virtual Desktop and Windows 365 provide Windows desktops and applications. For Linux desktops, you deploy and manage remote desktop software on Azure VMs or use a partner solution.

This article compares the main Linux remote desktop and VDI approaches on Azure. It focuses on architecture and selection criteria rather than product installation.

## Choose a Linux desktop approach

The following table maps common scenarios to an appropriate starting point.

| Scenario | Starting approach | Key consideration |
| --- | --- | --- |
| Administer or troubleshoot a Linux VM with graphical tools | A Linux VM with a remote desktop server | Each user connects to a specific VM. |
| Provide persistent desktops to a small development team | One or more Linux VMs with remote desktop software | You manage VM assignment, availability, and scaling. |
| Run 3D, computer-aided design (CAD), or scientific visualization applications | A graphics-capable GPU VM and an accelerated remote display protocol | The desktop session must route rendering through the GPU. |
| Provide centrally managed sessions to many users | A partner Linux VDI platform | Evaluate brokering, load balancing, identity, and client support. |
| Combine interactive Linux desktops with scheduled Slurm jobs | CycleCloud Workspace for Slurm with Open OnDemand | Users need private connectivity to the workspace and shared access to workload data. |
| Deliver Windows applications or desktops | [Azure Virtual Desktop](/azure/virtual-desktop/overview) | Azure Virtual Desktop session hosts run Windows, not Linux. |

## Single-VM remote desktop

The simplest architecture is a Linux VM that runs a desktop environment and a remote desktop server. Users connect from a native client or a browser, depending on the software. This approach works well for graphical administration, troubleshooting, proof-of-concept environments, and small teams that can each use an assigned VM.

A single-VM approach doesn't provide a connection broker, automatic host assignment, or load balancing. Session persistence, device redirection, and browser access also depend on the remote desktop product. As the number of users grows, independently managing each VM can become an operational burden.

For a basic Remote Desktop Protocol (RDP) configuration, see [Use xrdp with Linux](use-remote-desktop.md). xrdp is suitable for general desktop access, but it doesn't provide a complete multi-user VDI control plane or hardware-accelerated 3D rendering by itself.

## GPU-accelerated remote visualization

Graphics-intensive applications need a GPU in the rendering path. Examples include CAD, computer-aided engineering, molecular visualization, seismic interpretation, and medical imaging. Without GPU acceleration, these applications might render through the CPU and provide correct images at frame rates that aren't practical for interactive work.

An accelerated Linux desktop has three main requirements:

- **A graphics-capable GPU VM size.** The [NV family](../sizes/gpu-accelerated/nv-family.md) is designed for remote visualization and other graphics workloads. Compute-oriented NC and ND sizes target different workload profiles, so validate application and driver requirements before selecting a size.
- **A supported graphics driver.** NVIDIA-based NV sizes use an NVIDIA GRID driver for graphics workloads. For installation options, see [NVIDIA GPU Driver Extension for Linux](../extensions/hpccompute-gpu-linux.md) and [Install NVIDIA GPU drivers on N-series VMs running Linux](n-series-driver-setup.md). For AMD-based sizes, use the supported AMD graphics driver for the selected VM size and operating system.
- **A remote display path that uses the GPU.** A working desktop session doesn't prove that applications use hardware acceleration. The remote desktop product must integrate with the graphics stack, or applications must use a component such as VirtualGL to redirect rendering to the GPU.

> [!IMPORTANT]
> Verify acceleration with the applications that users run. Confirm that the OpenGL renderer reports the expected GPU instead of a software rasterizer such as `llvmpipe`.

Some Azure Marketplace images provide a preconfigured path for accelerated visualization. For example, the default ThinLinc Marketplace path is focused on NVIDIA GPU workloads. That image focus doesn't mean that ThinLinc itself requires NVIDIA GPUs. ThinLinc can also support CPU-only Linux desktops and other deployment designs when you install and configure the product separately.

For a standalone NVIDIA GPU deployment, see [Deploy GPU-accelerated Linux virtual desktops with ThinLinc on Azure](thinlinc-linux-vdi.md). For more information about accelerated rendering mechanisms, see [Remote visualization for HPC workloads on Azure](/azure/high-performance-computing/remote-visualization-overview).

## Multi-user and brokered Linux VDI

A brokered VDI architecture separates the user entry point from a specific VM address. A connection broker assigns sessions to available hosts, while management components can provide pooling, load balancing, session reconnection, monitoring, and policy controls.

Partner Linux VDI products are available through Azure Marketplace. Product capabilities vary, so compare the following factors with your own applications and network conditions:

- Native-client and browser support.
- Connection brokering and host-pool management.
- Session persistence after disconnects or network changes.
- Graphics acceleration and supported GPU vendors.
- Identity integration and multifactor authentication.
- Clipboard, local drive, printer, and other device-redirection controls.
- Licensing, vendor support, and operational ownership.
- Protocol performance over the expected latency and packet-loss range.

Benchmark candidate solutions by using representative applications, datasets, display resolutions, and user locations. Synthetic graphics benchmarks alone don't predict interactive user experience.

ThinLinc is one partner remote desktop product that can provide persistent Linux sessions through native clients or a browser. It isn't limited to NVIDIA-based deployments. However, the companion Azure deployment article focuses on a standalone NVIDIA GPU design because that configuration provides a documented path for accelerated visualization.

## Linux desktops with Slurm workloads

When interactive visualization is part of a high-performance computing (HPC) workflow, users often need a desktop close to the same storage and network used by scheduled jobs. [Azure CycleCloud Workspace for Slurm](/azure/cyclecloud/overview?view=cyclecloud-8) provides Slurm, autoscaling compute partitions, shared storage options, and Open OnDemand.

Open OnDemand is the browser-based entry point for interactive applications and job submission. Current CycleCloud Workspace for Slurm releases include ThinLinc integration for graphical desktop sessions. This path differs from a standalone ThinLinc VM: Open OnDemand handles the authenticated user entry point, and the workspace connects interactive sessions to the wider Slurm environment.

Use shared storage so that interactive sessions and compute nodes can access the same home directories and workload data. Open OnDemand also requires direct private connectivity to the workspace virtual network. For the supported configuration path, see [Configure Open OnDemand with CycleCloud](/azure/cyclecloud/how-to/ccws/configure-open-ondemand?view=cyclecloud-8).

## Access and security considerations

A graphical desktop is an interactive entry point to the VM and its data. Apply these controls to any Linux remote desktop or VDI architecture:

- Use private connectivity through a point-to-site or site-to-site VPN, or ExpressRoute. Use Azure Bastion tunneling only when the selected client and protocol support that connection model.
- Avoid public IP addresses for production session hosts. If temporary public access is required, restrict network security group rules to trusted source ranges and remove the rules after evaluation.
- Expose only the ports required by the selected product. Keep management interfaces private.
- Replace self-signed web certificates with certificates issued by a trusted certificate authority.
- Integrate with organizational identity controls and require multifactor authentication when the product supports it.
- Restrict clipboard, file transfer, drive mapping, printing, and other redirection features according to data handling requirements.
- Patch the operating system, desktop software, remote access components, and graphics drivers through a tested image lifecycle.

## Cost and operational considerations

Linux desktop costs depend on how long VMs run, how many users share each host, and whether the workload needs a GPU. Consider the following cost and management factors:

- Deallocate idle VMs so that compute billing stops. A stopped but allocated VM continues to incur compute charges.
- Use schedules or autoscaling when usage follows predictable hours.
- Size GPU VMs according to graphics memory and application requirements instead of selecting the largest available size.
- Compare persistent per-user VMs with shared hosts. Shared hosts can improve utilization but require brokering, capacity planning, and workload isolation.
- Include software licensing, support, image maintenance, monitoring, storage, and network egress in the total cost.

## Next steps

- Configure a basic desktop by following [Use xrdp with Linux](use-remote-desktop.md).
- Deploy a standalone accelerated desktop by following [Deploy GPU-accelerated Linux virtual desktops with ThinLinc on Azure](thinlinc-linux-vdi.md).
- Review the [CycleCloud Workspace for Slurm deployment quickstart](/azure/cyclecloud/qs-deploy-ccws?view=cyclecloud-8).
- Learn how accelerated Linux sessions work in [Remote visualization for HPC workloads on Azure](/azure/high-performance-computing/remote-visualization-overview).
