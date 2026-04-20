---
title: Azure HPC Guest Health Reporting - Impact Categories 
description: View Guest Health Reporting impact categories.
author: rolandnyamo 
ms.author: ronyamo 
ms.service: azure 
ms.topic: overview 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# Impact categories for Guest Health Reporting (preview)

To properly report issues to Guest Health Reporting, you must use an impact category that starts with `Resource.HPC`.

There are three main types of impact categories for high-performance computing (HPC):

* `Reset`: Request a refresh of the node health state.
* `Reboot`: Request a node start.
* `Unhealthy`: Issues are observed on the node. Take the node out of production for further diagnostics and repair.

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Detailed HPC impact categories

| Category                                          | Description                                   | Mark as out for repair |
|--------------------------------------------------|-----------------------------------------------|----------|
| `Resource.Hpc.Reset`                               | Reset node health status.                      | No       |
| `Resource.Hpc.Reboot`                              | Restart the node.                              | No       |
| `Resource.Hpc.Unhealthy.HpcMissingGpu`             | Missing GPU.                                   | Yes      |
| `Resource.Hpc.Unhealthy.MissingIB`                 | Missing InfiniBand port.                       | Yes      |
| `Resource.Hpc.Unhealthy.IBPerformance`             | Degraded InfiniBand performance.               | Yes      |
| `Resource.Hpc.Unhealthy.IBPortDown`                | InfiniBand port is in a down state.            | Yes      |
| `Resource.Hpc.Unhealthy.IBPortFlapping`            | InfiniBand port flapping.                      | Yes      |
| `Resource.Hpc.Unhealthy.HpcGpuDcgmDiagFailure`     | Diagnostic failure of Data Center GPU Management Interface (DCGMI) for the GPU datacenter. | Yes      |
| `Resource.Hpc.Unhealthy.HpcRowRemapFailure`        | Failure of GPU row remapping.                         | Yes      |
| `Resource.Hpc.Unhealthy.HpcInforomCorruption`      | GPU infoROM corruption.                        | Yes      |
| `Resource.Hpc.Unhealthy.HpcGenericFailure`         | Issue doesn't fall into any other category.    | Yes      |
| `Resource.Hpc.Unhealthy.ManualInvestigation`       | Request further manual investigation by the HPC team. | Yes   |
| `Resource.Hpc.Unhealthy.XID95UncontainedECCError`  | GPU uncontained Error-Correcting Code (ECC) error (Xid 95). | Yes      |
| `Resource.Hpc.Unhealthy.XID94ContainedECCError`    | GPU contained ECC error (Xid 94).              | Yes      |
| `Resource.Hpc.Unhealthy.XID79FallenOffBus`         | GPU fell off the Peripheral Component Interconnect Express (PCIe) bus (Xid 79).            | Yes      |
| `Resource.Hpc.Unhealthy.XID48DoubleBitECC`         | GPU reports a double-bit ECC error (Xid 48).   | Yes      |
| `Resource.Hpc.Unhealthy.UnhealthyGPUNvidiasmi`     | NVIDIA System Management Interface (nvidia-smi) stops responding and might not recover. | Yes      |
| `Resource.Hpc.Unhealthy.NvLink`                    | NvLink is down.                                | Yes      |
| `Resource.Hpc.Unhealthy.HpcDcgmiThermalReport`     | DCGMI reports thermal violations.              | Yes      |
| `Resource.Hpc.Unhealthy.ECCPageRetirementTableFull`| Page retirements for double-bit ECC errors are over the threshold. | Yes |
| `Resource.Hpc.Unhealthy.DBEOverLimit`              | GPU has more than 10 retired pages for double-bit ECC errors in seven days. | Yes |
| `Resource.Hpc.Unhealthy.GpuXIDError`               | GPU reports a Xid error other than 48, 79, 94, or 95. | Yes      |
| `Resource.Hpc.Unhealthy.AmdGpuResetFailed`         | AMD GPU error for an unrecoverable reset failure. | Yes      |
| `Resource.Hpc.Unhealthy.EROTFailure`               | Failure of GPU memory External Root of Trust (eRoT).                       | Yes      |
| `Resource.Hpc.Unhealthy.GPUMemoryBWFailure`        | Failure of GPU memory bandwidth.                  | Yes      |
| `Resource.Hpc.Unhealthy.CPUPerformance`            | CPU performance issue.                         | Yes      |

## Related content

* [What is Guest Health Reporting?](guest-health-overview.md)
* [Report node health by using Guest Health Reporting](guest-health-impact-report.md)
