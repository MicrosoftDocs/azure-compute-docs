---
title: 'Azure Impact Reporting: Create an Azure Monitor Alert Connector'
description: Learn how to create an Azure Impact Reporting connector for Azure Monitor alerts. 
author: rolandnyamo
ms.author: ronyamo
ms.service: azure 
ms.topic: overview
ms.date: 06/19/2024
ms.custom: template-overview
# Customer intent: As an IT administrator, I want to create an Impact Reporting connector for Azure Monitor Alerts so that I can efficiently manage and report on alerts across my Azure subscriptions.
---

# Create an impact connector for Azure Monitor alerts (preview)

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

You can create impact connectors through [AzCLI](/cli/azure), [Azure PowerShell](/powershell/azure), or the Azure portal. This article outlines the connector creation process and requirements.

## Prerequisites

| Type     | Details      |
| ------------- | ------------- |
| Azure role-based access control permissions | [Owner](/azure/role-based-access-control/built-in-roles#owner) or [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) to give the Connector Alerts Reader role. Contributor access to the subscription for provider registration and to enable connector features and create connector resources. |
| Command-line tools | [Bash](/cli/azure) or [PowerShell](/powershell/azure). (*Not needed if you use Azure Cloud Shell.*)|
| Subscription ID| A subscription ID or a file that contains a list of subscription IDs whose alerts are of interest.|

## Create an Azure Impact Reporting connector

### [Command line](#tab/cli/)

Here's what deployment scripts do:

* Register your subscriptions for Azure Impact Reporting Preview (prerequisite for using connectors).
* Create a connector resource (`microsoft.impact/connector`).
* Report an impact whenever an alert from those subscriptions is triggered.

#### Step 1: Get the script

Go to the [Azure Impact Reporting samples](https://github.com/Azure/impact-reporting-samples/tree/main/Onboarding/Connector/Azure%20Monitor%20Connector/Scripts) GitHub repo, select your script, and choose either the Bash or PowerShell script.

#### Step 2: Run in your environment

You need to run this script in your Azure environment.

#### PowerShell

* **Single subscription**: `./CreateImpactReportingConnector.ps1 -SubscriptionId <subid>`
* **Multiple subscriptions from file**: `./CreateImpactReportingConnector.ps1 -FilePath './subscription_ids'`

#### Bash

* **Single subscription**: `./create-impact-reporting-connector.sh --subscription-id <subid>`
* **Multiple subscriptions from file**: `./create-impact-reporting-connector.sh --file_path './subscription_ids'`

### [Portal](#tab/portal/)

1. Search for **Impact Reporting Connectors** in the Azure portal search box.

1. On the left pane, select **Create** to open the **Create Impact Reporting Connector** page.

    ![Screenshot that shows the Create Impact Reporting Connector page.](images/create-connector.png)

1. In the **Subscription** dropdown list, select the subscription for the connector.

1. Under **Instance details**, for **Connector name**, enter **AzureMonitorConnector**. For **Connector type**, select **Azure Monitor**.

1. Select **Review + create**.

1. After validation and the **Review + create** tab shows no error, create the connector.

1. The deployment can take several minutes to complete. Feature flags need registration, which can take some time to propagate. Meanwhile, continue to the next section to enable alert reading from your subscription.

---

## Assign the Azure Alerts Reader role to the connector

1. Go to your subscription. On the service menu, select **Access control (IAM)**.

1. Select **Add**, and then select **Add role assignment**.

1. On the **Add role assignment** page, on the **Role** tab, enter **Azure Alerts Reader role** in the search bar. If this role doesn't exist, skip to [Create the Azure Alerts Reader role](#create-the-azure-alerts-reader-role) to create this role. After the role is created, return to this step.

1. Select **Azure Alerts Reader role** and select **Next**.

1. On the **Members** tab, for **Assign access to**, select **User, group, or service principal**.

1. Choose **Select members** to open the **Select Members** pane on the right side.

1. Enter **AzureImpactReportingConnector** in the search bar, and select the **AzureImpactReportingConnector** application.

1. Select **Review + assign**.

1. On the **Review + assign** tab, select **Review + assign**.

## Create the Azure Alerts Reader role

1. Go to your subscription. On the service menu, select **Access control (IAM)**.

1. Select **Add**, and then select **Add custom role** to open the **Create a custom role** page.

1. On the **Basics** tab, for **Custom role name**, enter **Azure Alerts Reader role**. Leave others as defaults, and select **Next**.

1. On the **Permissions** tab, select **Add permissions**.

1. Enter **Microsoft.AlertsManagement/alerts/read** in the search bar.

1. Select the tile **Microsoft.AlertsManagement** to access the **Microsoft.AlertsManagement permissions** pane. Select the permission **Read: Read alerts**, and then select **Add**.

1. Select **Review + create**.

1. On the **Review + create** tab, select **Create**.

## Troubleshooting

### Connector deployment fails because of permission errors

Ensure that you have Contributor permission to sign in to Azure, register resource providers, and create connectors in Azure subscriptions.

### Custom role assignment fails because of permission errors

You also need the [Owner](/azure/role-based-access-control/built-in-roles#owner) or [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) role to create and assign custom roles.

### Connector creation takes too long

Namespace registration that allows connector resource creation might take 15 to 20 minutes. If the script execution isn't finished after 30 minutes, cancel the execution and rerun it. If this issue persists, contact the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

## Related content

* [What is the impact connector for Azure Monitor alerts?](azure-monitor-connector.md)
* [Impact Reporting connectors: Troubleshooting guide](connectors-troubleshooting-guide.md)
* [View reported impacts and insights](view-impact-insights.md)
