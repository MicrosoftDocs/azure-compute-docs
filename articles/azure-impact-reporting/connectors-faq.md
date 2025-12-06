---
title: 'Azure Impact Reporting Connectors for Azure Monitor: FAQ'
description: Frequently asked questions about Azure Monitor connectors. 
author: rolandnyamo
ms.author: ronyamo
ms.topic: faq
ms.service: azure 
ms.date: 06/25/2024
ms.custom: template-overview
# Customer intent: As an Azure administrator, I want to troubleshoot and configure Azure Impact Reporting connectors so that I can ensure successful integration with Azure Monitor and manage permissions effectively.
---

# Azure Impact Reporting connectors for Azure Monitor (preview) FAQ

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Here are answers to common questions about Azure Impact Reporting connectors.

## How do I enable debug mode?

* **Bash**: Uncomment the `set -x` at the beginning of the script.
* **PowerShell**: Change `Set-PSDebug -Trace 0` to `Set-PSDebug -Trace 1` at the beginning.

## What should I do if I encounter a permission error?

Verify your Azure role and permissions. You might need the help of your Azure administrator to grant you the necessary permissions or roles as defined in the permissions section.

## How can I verify if the connector is successfully created?

### Option 1

* **Bash**:
    * Run the command `az rest --method get --url https://management.azure.com/subscriptions/<subscription-id>/providers/Microsoft.Impact/connectors?api-version=2024-05-01-preview`.
    * You should see a resource with the name `AzureMonitorConnector`.
* **PowerShell**:
    * Run the command `(Invoke-AzRestMethod -Method Get -Path subscriptions/<Subscription Id>/providers/Microsoft.Impact/connectors?api-version=2024-05-01-preview).Content`.
    * You should see a resource with the name `AzureMonitorConnector`.

### Option 2

1. In the Azure portal, go to [Azure Resource Graph Explorer](https://portal.azure.com/#view/HubsExtension/ArgQueryBlade).

1. Run this query:

    ```kql
    impactreportresources  | where name == "AzureMonitorConnector"  and type == "microsoft.impact/connectors"
    ```

## Related content

* [File an impact report](report-impact.md)
* [Create an impact connector for Azure Monitor alerts](create-azure-monitor-connector.md)
