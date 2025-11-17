---
title: 'Azure Impact Reporting: Overview'
description: Azure Impact Reporting is a service that you can use to report observed performance and availability regressions with your Azure workloads. . 
author: rolandnyamo
ms.author: ronyamo
ms.service: azure 
ms.topic: overview
ms.date: 09/17/2025
ms.custom: template-overview
# Customer intent: As a cloud administrator, I want to use impact reporting tools to document performance issues in my Azure workloads so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# What is Azure Impact Reporting Preview?

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

To enhance your investigations into observed workload regressions, Azure Impact Reporting provides an Azure platform relevant to the resource and the time of the regression. When you report an issue, we investigate changes, outages, and other platform events in the context of provided resources and provide you with findings.

[![Diagram that shows the architecture of impact connectors for Azure Monitor.](images/impact-reporting-end-to-end.png)](images/impact-reporting-end-to-end.png#lightbox)

## What is an impact?

In this context, an *impact* is any observed regression, unexpected behavior, or issue that negatively affects your workloads, and you suspect the platform is the cause.

Examples of impacts include:

* Unexpected virtual machine reboots.
* Disk I/O failures.
* High data-path latency.

## Related content

* [File an impact report](report-impact.md)
* [View impact insights](view-impact-insights.md)
