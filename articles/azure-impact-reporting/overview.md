---
title: Azure Impact Reporting - Overview #Required; page title is displayed in search results. Include the brand.
description: An overview of Azure Impact Reporting - a service that enables you to send reports when you observe a negative change in performance of your Azure workloads. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 11/18/2022 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to utilize impact reporting tools to document performance issues in my Azure workloads, so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# What is Impact Reporting? (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Azure Impact Reporting enables you to leverage Azure Intelligence Systems to enhance your investigations through reporting suspected platform issues. In addition to providing you with insights if found, these impact reports are used in resource health modeling to determine service and resource health.

![End-to-end architecture diagram of Azumre Impact Reporting.](images/impact-reporting-end-to-end.png)

## What is an "Impact"?

In this context, an impact is any unexpected behavior or issue negatively affecting your workloads suspected to be caused by the platform.

Examples of impacts include:

* Unexpected VM reboot
* Disk IO failures
* High datapath latency 

## Next steps
<!-- Add a context sentence for the following links -->
* [File an impact report](report-vm-impact.md)
<!-- - [View previous impact reports](links-how-to.md) -->
