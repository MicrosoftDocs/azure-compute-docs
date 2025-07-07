---
title: Azure Impact Reporting - Connectors Troubleshooting Guide #Required; page title is displayed in search results. Include the brand.
description: Azure Impact Reporting Connectors Troubleshooting Guide #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: kezaveri #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 06/25/2024 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As an Azure user troubleshooting connector creation, I want guidance on solving common errors, so that I can successfully create and manage Impact Reporting Connectors without delays or issues.
---

# Impact Reporting Connectors Troubleshooting Guide (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

This guide outlines solutions to common errors faced when creating an Impact Reporting Connector.

## The bash script fails immediately after starting
Ensure that the script has execution permissions. Use this command to make it executable.
`chmod +x create-impact-reporting-connector.sh`

## In the bash script, Azure sign-in fails (az sign-in command not working)
Ensure [Azure CLI](/cli/azure) is installed and updated to the latest version. Try manually logging in using `az login` to check for any more prompts or errors.

## Error "**Subscription ID or file path with list of subscription IDs required**"
- **Bash**: Make sure you're providing either `--subscription-id` or `--file-path` argument when executing the script. Don't provide both. <br>
- **Powershell**: Make sure to provide either the `-SubscriptionId` parameter or the  `-FilePath` parameter when invoking the script. Don't provide both.

## Error "**Failed to find file: [file_path]**"
- **Bash**: Verify the file path provided with `--file-path` exists and is accessible. Ensure the correct path is used. <br>
- **Powershell**: Verify the file path provided with `-FilePath` exists and is accessible. Ensure to use the correct path and the file isn't locked or in use by another process.

## Script fails to execute with permission errors
Ensure you have Contributor permission to log into Azure, register resource providers, and create connectors in the Azure subscriptions. You also need to have `User Access Administrator` permission to create and assign custom roles.

## Script execution stops unexpectedly without completing
Check if the Azure PowerShell module is installed and up to date. Use `Update-Module -Name Az` to update the Azure PowerShell module. Ensure `$ErrorActionPreference` is set to `Continue` temporarily to bypass noncritical errors.

## Namespace or feature registration takes too long or fails
These operations can take several minutes. Ensure your Azure account has the Contributor access on the subscriptions. Rerun the script once the required access is granted. Reach out to the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com) if the issue persists.

## Custom role creation or assignment fails
1.	Ensure the Azure Service Principal `AzureImpactReportingConnector` exists by typing it into the Azure resource search box as shown in the image, if not wait for a few minutes for it to get created. If it doesn't get created even after an hour, reach out to the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

2.	Verify your account has `User Access Administrator` permission to create roles and assign them.
## Connector creation takes too long
It can take about 15-20 minutes for the namespace registration to allow the connector resource creation to take place. After 30 minutes if the script is still running, cancel its execution and rerun it. If this second run also gets stuck, reach out to the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).

## Connector creation fails

1. Ensure that the `Microsoft.Impact` resource provider is registered. You can register the subscription in two ways -
    - From the Azure portal, navigate to your `Subscription -> Resource Providers`.
    - Search for `Microsoft.Impact` and register.
    - **Bash**: run `az provider show -n "Microsoft.Impact" -o json --query "registrationState"`.
    - **PowerShell**: run `Get-AzResourceProvider -ProviderNamespace Microsoft.Impact`.
2.	Ensure that the feature flags: AllowImpactReporting and `AzureImpactReportingConnector` are registered against the feature:` Microsoft.Impact`.

    - **Bash**
        - `az feature list -o json --query "[?contains(name, 'Microsoft.Impact/AllowImpactReporting')].{Name:name,State:properties.state}"`
        - `az feature list -o json --query "[?contains(name, 'Microsoft.Impact/AzureImpactReportingConnector')].{Name:name,State:properties.state}"` <br>
    - **PowerShell**
        - `Get-AzProviderFeature -ProviderNamespace "Microsoft.Impact" -FeatureName AzureImpactReportingConnector"`
        - `Get-AzProviderFeature -ProviderNamespace "Microsoft.Impact" -FeatureName AllowImpactReporting` <br>
3.	Ensure that you have Contributor access to the subscriptions

This covers the common scenarios encountered while onboarding the connector. For issues not covered here, reach out to the [Impact Reporting connectors team](mailto:impactrp-preview@microsoft.com).
