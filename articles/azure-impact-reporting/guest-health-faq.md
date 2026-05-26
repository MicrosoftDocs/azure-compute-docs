---
title: Azure HPC Guest Health Reporting - FAQ 
description: Frequently asked questions for Guest Health Reporting.
author: rolandnyamo 
ms.author: ronyamo 
ms.topic: faq 
ms.service: azure 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# FAQ for Guest Health Reporting (preview)

Here are answers to common questions about Guest Health Reporting.

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## What happens after sending the request to Guest Health Reporting?

Reports with unhealthy impact categories are evaluated to determine the best way to recover. In some cases, recovery can be done while the VM is running; in other cases, interruption is needed. Progress updates are provided as Insights, which belong to the original Impact — see [list impacts for a workload impact](guest-health-impact-report.md#query-workload-impact-insights). The terminal insight advises whether VM deallocation is required. Similarly, the VM availability through [Project Flash](/azure/virtual-machines/flash-overview) changes. Once these signals appear, typically within minutes, the platform has a repair planned and is waiting for the virtual machine (VM) to be deallocated. If, after 30 days, the VM isn't deallocated, the platform tries to move the VM to an alternate node. If an alternate node isn't available, the platform continues waiting for a VM deallocate/delete call.

## Related content

* [What is Guest Health Reporting?](guest-health-overview.md)
* [Report node health by using Guest Health Reporting](guest-health-impact-report.md)
* [How to provide a diagnostic log file with Guest Health Reporting](guest-health-log-upload.md)

