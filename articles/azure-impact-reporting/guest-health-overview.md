---
title: Azure HPC Guest Health Reporting - Overview 
description: Report Azure supercomputing VM device health status to Microsoft. 
author: rolandnyamo 
ms.author: ronyamo 
ms.service: azure 
ms.topic: overview 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# What is Guest Health Reporting (preview)?

Guest Health Reporting is a feature of the Azure Impact Reporting service that allows Azure supercomputing customers to provide virtual machine (VM) device health statuses to Azure. Based on these status updates, Azure high-performance computing (HPC) can make decisions to remove problematic nodes from production and send them for repair.

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Onboarding process

To use Guest Health Reporting to report the health of a node, you need to onboard the subscription that hosts the resources to the Impact Reporting service:

1. In the Azure portal, select **Subscriptions** and go to the relevant subscription.

1. On the left menu, select **Resource providers**.

   :::image type="content" source="images/guest-health-onboarding-subscription.png" alt-text="Screenshot that shows subscription settings with the link for resource providers.":::

1. Search for and select the **Microsoft.Impact** resource provider, and then select **Register**.

   :::image type="content" source="images/guest-health-registration.png" alt-text="Screenshot that shows the button for registering a resource provider.":::

1. On the left menu, go to **Settings** > **Preview features**.

1. Search for and select **Allow Impact Reporting**, and then select **Register**.

   :::image type="content" source="images/guest-health-preview-feature-selection.png" alt-text="Screenshot that shows selections for allowing Impact Reporting.":::

1. On the left pane, go to **Settings** > **Overview**. Retrieve your subscription ID and send it to the Azure team member who's helping you complete the onboarding process.

1. Wait for confirmation that the onboarding process is complete before you proceed with submitting Guest Health Reporting requests.

## Access management and role assignment

To submit Guest Health Reporting requests from a resource within Azure, you must assign the appropriate access management roles:

1. Create a user-assigned or system-assigned managed identity.

2. On the left menu, go to **Access control (IAM)**. Then select **Add role assignment**.

   :::image type="content" source="images/guest-health-add-role.png" alt-text="Screenshot that shows selections for adding a role assignment.":::

3. Search for and select the **Impact reporter** role.

   :::image type="content" source="images/guest-health-impact-reporter-role.png" alt-text="Screenshot that shows search results for the Impact Reporter role on the pane for adding a role assignment.":::

4. Go to the **Members** tab. Search for and select the user identity, app ID, or  service principal. Then select **Members**. The app ID is the service principal for the app to be used for reporting.

5. Review and assign the app ID or the managed identity.

## Related content

* [Report node health by using Guest Health Reporting](guest-health-impact-report.md)
* [Impact categories for Guest Health Reporting](guest-health-impact-categories.md)
