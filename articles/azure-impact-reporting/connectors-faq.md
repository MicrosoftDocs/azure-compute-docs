---
title: Azure Impact Connectors for Azure Monitor - FAQ #Required; page title is displayed in search results. Include the brand.
description: Frequently asked questions for Azure Monitor Connectors. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.topic: faq #Required; leave this attribute/value as-is.
ms.service: azure #Required; use either service or product per approved list. 
ms.date: 06/25/2024 #Required; mm/dd/yyyy format
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As an Azure administrator, I want to troubleshoot and configure Azure Impact Reporting Connectors, so that I can ensure successful integration with Azure Monitor and manage permissions effectively.
---

# Azure Impact Reporting Connectors for Azure Monitor FAQ (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Here are answers to common questions about Azure Impact Reporting Connectors.

## How do I enable debug mode?

* **Bash**: Uncomment the `set -x` at the beginning of the script.
* **Powershell**: Change `Set-PSDebug -Trace 0 to Set-PSDebug -Trace 1` at the beginning

## What should I do if I encounter a permission error?

Verify your Azure role and permissions. You may need the help of your Azure administrator to grant you the necessary permissions or roles as defined in the permissions section.

## How can I verify if the connector is successfully created?
### Option 1
* **Bash**:
    * Step 1: Run this command: 
`az rest --method get --url https://management.azure.com/subscriptions/<subscription-id>/providers/Microsoft.Impact/connectors?api-version=2024-05-01-preview`.
    * Step 2: You should see a resource with the name `AzureMonitorConnector`.
* **Powershell**:
    * Step 1: Run this command: 
`(Invoke-AzRestMethod -Method Get -Path subscriptions/<Subscription Id>/providers/Microsoft.Impact/connectors?api-version=2024-05-01-preview).Content`.
    * Step 2: You should see a resource with the name `AzureMonitorConnector`.

### Option 2
* Step 1: From the Azure portal, navigate to [Azure Resource Graph Explorer](https://portal.azure.com/#view/HubsExtension/ArgQueryBlade).
* Step 2: Run this query: 
```kql
impactreportresources  | where name == "AzureMonitorConnector"  and type == "microsoft.impact/connectors"
```

## Next steps

* [File an impact report](report-impact.md).
<!-- Add Impact Connectors TSG -->
<!-- - [File an impact report](report-impact.md) -->