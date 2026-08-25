---
title: include file
description: include file
services: virtual-machines
author: msmbaldwin
ms.service: azure-virtual-machines
ms.topic: include
ms.date: 07/14/2026
ms.author: mbaldwin
ms.custom: include file, devx-track-azurepowershell
ai-usage: ai-assisted
# Customer intent: "As a systems administrator, I want to enable and configure Azure Disk Encryption using either the Azure CLI or PowerShell, so that I can ensure the security and compliance of my virtual machine data."
---
You can enable and manage Azure Disk Encryption through the [Azure CLI](/cli/azure) and [Azure PowerShell](/powershell/azure/new-azureps-module-az). To do so, install the tools locally and connect to your Azure subscription.

### Azure CLI

The [Azure CLI 2.0](/cli/azure) is a command-line tool for managing Azure resources. The CLI is designed to flexibly query data, support long-running operations as non-blocking processes, and make scripting easy. You can install it locally by following the steps in [Install the Azure CLI](/cli/azure/install-azure-cli).

To [sign in to your Azure account with the Azure CLI](/cli/azure/authenticate-azure-cli), use the [az login](/cli/azure/reference-index#az-login) command.

```azurecli
az login
```

To select a tenant to sign in under, use:

```azurecli
az login --tenant <tenant>
```

To specify one of multiple subscriptions, get your subscription list by using [az account list](/cli/azure/account#az-account-list), and specify the subscription by using [az account set](/cli/azure/account#az-account-set).

```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

For more information, see [Get started with Azure CLI](/cli/azure/get-started-with-azure-cli).

### Azure PowerShell

The [Azure PowerShell Az module](/powershell/azure/new-azureps-module-az) provides a set of cmdlets that use the [Azure Resource Manager](/azure/azure-resource-manager/management/overview) model for managing your Azure resources. You can use it in your browser with [Azure Cloud Shell](/azure/cloud-shell/overview), or you can install it on your local machine by using the instructions in [Install the Azure PowerShell module](/powershell/azure/install-azure-powershell).

If you already have it installed locally, use the latest Azure PowerShell version to configure Azure Disk Encryption. Download the latest version from [Azure PowerShell releases](https://github.com/Azure/azure-powershell/releases).

To [sign in to your Azure account with Azure PowerShell](/powershell/azure/authenticate-azureps), use the [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) cmdlet.

```powershell
Connect-AzAccount
```

To specify one of multiple subscriptions, use the [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) cmdlet to list them, followed by the [Set-AzContext](/powershell/module/az.accounts/set-azcontext) cmdlet:

```powershell
Set-AzContext -Subscription <SubscriptionId>
```

Run the [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) cmdlet to verify that the correct subscription is selected.

To confirm the Azure Disk Encryption cmdlets are installed, use the [Get-Command](/powershell/module/microsoft.powershell.core/get-command) cmdlet:

```powershell
Get-Command *diskencryption*
```

For more information, see [Getting started with Azure PowerShell](/powershell/azure/get-started-azureps).
