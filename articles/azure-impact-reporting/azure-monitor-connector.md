---
title: Azure Impact Reporting - Azure Monitor Alert Connector #Required; page title is displayed in search results. Include the brand.
description: Seamlessly report impact through Azure Monitor alerts. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 06/19/2024 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: "As a systems administrator, I want to utilize the Impact Connector for Azure Monitor alerts, so that I can efficiently report impact events and enhance change event correlation within my organization's AIOps workflows."
---

# What is the Impact Connector for Azure Monitor Alerts? (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The impact Connector for Azure Monitor alerts enables you to seamlessly report impact from an alert into Microsoft AIOps for change event correlation.

## How Connectors work

![Architecture diagram of impact connectors for azure monitor.](images/azure-monitor-connector.png)

When you create a connector, it gets associated with a subscription. When alerts whose target resources reside in the specified subscription get fired, an Impact report is created through Azure Impact Reporting and sent to Microsoft Intelligence Systems.

## Next steps
* [Create a Connector for Azure Monitor Alerts](create-azure-monitor-connector.md).
* [Impact Reporting Connectors - Troubleshooting Guide](connectors-troubleshooting-guide.md).
* [View Reported Impacts and Insights](view-impact-insights.md).