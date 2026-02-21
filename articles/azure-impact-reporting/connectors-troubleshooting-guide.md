---
title: 'Azure Impact Reporting: Connectors Troubleshooting Guide'
description: Azure Impact Reporting Connectors Troubleshooting Guide 
author: rolandnyamo
ms.author: kezaveri
ms.service: azure 
ms.topic: overview
ms.date: 06/25/2024
ms.custom: template-overview
# Customer intent: As an Azure user troubleshooting connector creation, I want guidance on solving common errors so that I can successfully create and manage Impact Reporting Connectors without delays or issues.
---

# Impact Reporting connectors (preview) troubleshooting guide

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

This article outlines solutions to common errors that you might encounter when you create an Azure Impact Reporting connector. For issues not covered here, contact the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

## Bash script fails immediately after starting

Ensure that the script has execution permissions. Use this command to make it executable:

`chmod +x create-impact-reporting-connector.sh`

## In the Bash script, Azure sign-in fails (az sign-in command not working)

Ensure that the [Azure CLI](/cli/azure) is installed and updated to the latest version. Try to sign in manually by using `az login` to check for any more prompts or errors.

## Error: "Subscription ID or file path with list of subscription IDs required"

- **Bash**: Make sure to provide either the `--subscription-id` or `--file-path` argument when you run the script. Don't provide both. <br>
- **PowerShell**: Make sure to provide either the `-SubscriptionId` parameter or the `-FilePath` parameter when you invoke the script. Don't provide both.

## Error: "Failed to find file: [file_path]"

- **Bash**: Verify that the file path provided with `--file-path` exists and is accessible. Make sure to use the correct path. <br>
- **PowerShell**: Verify that the file path provided with `-FilePath` exists and is accessible. Make sure to use the correct path, and confirm that the file isn't locked or in use by another process.

## Script fails to execute with permission errors

Ensure that you have Contributor permission to sign in to Azure, register resource providers, and create connectors in the Azure subscriptions. You also need to have User Access Administrator permission to create and assign custom roles.

## Script execution stops unexpectedly without completing

Check if the Azure PowerShell module is installed and up to date. Use `Update-Module -Name Az` to update the Azure PowerShell module. Ensure that `$ErrorActionPreference` is set to `Continue` to temporarily bypass noncritical errors.

## Namespace or feature registration takes too long or fails

These operations can take several minutes. Ensure that your Azure account has Contributor access on the subscriptions. Rerun the script after the required access is granted. If the issue persists, contact the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

## Custom role creation or assignment fails

1. Ensure that the Azure service principal `AzureImpactReportingConnector` exists by entering it into the Azure resource search box. If not, wait for a few minutes for it to get created. If it doesn't get created after an hour, contact the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

1. Verify that your account has User Access Administrator permission to create roles and assign them.

## Connector creation takes too long

Namespace registration that allows connector resource creation might take 15 to 20 minutes. If the script is still running after 30 minutes, cancel its execution and rerun it. If this second run also gets stuck, contact the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

## Connector creation fails

1. Ensure that the `Microsoft.Impact` resource provider is registered. You can register the subscription in two ways:

    - From the Azure portal, go to **Subscription** > **Resource Providers**.
    - Search for **Microsoft.Impact** and register:

       - **Bash**: Run `az provider show -n "Microsoft.Impact" -o json --query "registrationState"`.
       - **PowerShell**: Run `Get-AzResourceProvider -ProviderNamespace Microsoft.Impact`.

1. Ensure that the feature flags `AllowImpactReporting` and `AzureImpactReportingConnector` are registered against the feature `Microsoft.Impact`:

    - **Bash**
        - `az feature list -o json --query "[?contains(name, 'Microsoft.Impact/AllowImpactReporting')].{Name:name,State:properties.state}"`
        - `az feature list -o json --query "[?contains(name, 'Microsoft.Impact/AzureImpactReportingConnector')].{Name:name,State:properties.state}"` <br>
    - **PowerShell**
        - `Get-AzProviderFeature -ProviderNamespace "Microsoft.Impact" -FeatureName AzureImpactReportingConnector"`
        - `Get-AzProviderFeature -ProviderNamespace "Microsoft.Impact" -FeatureName AllowImpactReporting` <br>
1. Ensure that you have Contributor access to the subscriptions.
