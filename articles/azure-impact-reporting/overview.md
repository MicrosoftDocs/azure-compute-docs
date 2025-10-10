---
title: Azure Impact Reporting - Overview #Required; page title is displayed in search results. Include the brand.
description: An overview of Azure Impact Reporting - a service that enables you to report observed performance and availability regressions with your Azure workloads. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/17/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to utilize impact reporting tools to document performance issues in my Azure workloads, so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# What is Impact Reporting? (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Our goal with Azure Impact Reporting is to enhance customer investigations into observed workload regressions by providing Azure platform relevant to the resource and time of the regression. When you report an issue, we investigate changes, outages, and other platform events in the context of provided resources and provide you with findings.

[ ![Architecture diagram of impact connectors for azure monitor.](images/impact-reporting-end-to-end.png) ](images/impact-reporting-end-to-end.png#lightbox)

## What is an "Impact"?

In this context, an impact is any observed regression, unexpected behavior, or issue negatively affecting your workloads and suspected to be caused by the platform.

Examples of impacts include:

* Unexpected virtual machine reboot
* Disk IO failures
* High datapath latency 

## Next steps
<!-- Add a context sentence for the following links -->
* [File an impact report](report-impact.md)
<!-- - [View previous impact reports](links-how-to.md) -->
