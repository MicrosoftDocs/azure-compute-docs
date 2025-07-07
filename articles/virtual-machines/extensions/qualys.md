---
title: Qualys Cloud Agent Extension for Azure VMs
description: Install Qualys Agent on Linux or Windows VMs using the VM Extension.
ms.topic: concept-article
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.date: 05/23/2025
# Customer intent: As a cloud administrator, I want to install and configure the Qualys Cloud Agent on Azure VMs using various deployment methods, so that I can ensure continuous security monitoring and data analysis for my virtual machines.
---

# Qualys Agent on Linux or Windows VMs

The Qualys Cloud Agent is a lightweight, cloud-based security and monitoring tool installed on servers, virtual machines (VMs), and workstations. It continuously collects system data and sends it to Qualys Enterprise TruRisk Platform (ETP) for analysis and reporting.

Qualys supports Cloud Agent deployment on Windows and Linux Azure VMs via Microsoft’s Azure portal. The [Cloud Agent Platform Availability Matrix](https://success.qualys.com/customersupport/s/cloud-agent-pam) provides more information about the supported platforms.

> [!IMPORTANT]
> Qualys has removed support for the Windows Cloud Agent extension version 3.1.3.34.  This version will become inoperable starting January 31, 2026. We recommend that you uninstall it and install the latest Windows Cloud Agent extension version (1.6.x.x) on your Azure portal. To learn more about the uninstallation steps, refer to [Uninstalling and Reinstalling Qualys Extension on Azure VMs](https://success.qualys.com/support/s/article/000007473).

## Installation Methods
The following are the different deployment methods to deploy Cloud Agent extensions on Azure VMs.

### Deploying Qualys Cloud Agent for Single Azure VM
The Microsoft Azure platform has the option to deploy a Cloud Agent extension for a single Azure VM. To learn more about the deployment steps, refer to [Deploy Cloud Agent for Single Azure VM](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/deploy_ca_single_vm.htm).

### Embedding Qualys Cloud Agent via Golden Machine Image
The Qualys Cloud Agent supports configuration and deployment into cloned images in cloud environments like Microsoft Azure.

To learn more about embedding Qualys Cloud Agent as a part of Golden Machine Image, refer to the following links:
- **For Windows VMs:** [Steps to Install Qualys Cloud Agent for Windows in Gold Image](https://docs.qualys.com/en/ca/install-guide/windows/gold_image/gold_image_install.htm)
- **For Linux VMs:** [Steps to Install Qualys Cloud Agent for Linux in Gold Image](https://docs.qualys.com/en/ca/install-guide/linux/gold_image/Cloud_Agent_Installation_in_Gold_Image.htm)

### Deploy Qualys Cloud Agent via Azure Resource Manager Template
Qualys supports the Azure Resource Manager (ARM) templates to deploy and install Qualys Cloud Agent as a virtual machine extension on a list of Azure Windows and Linux VMs. VM extensions can be used whenever a virtual machine requires software installation, anti-virus protection, or running a script inside of it.

To learn more about deployment steps, refer to [Cloud Agent Deployment via ARM Template](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/deploy_ca_resource_manager.htm).

### Deploy Qualys Cloud Agent via Azure Deployment Policy
Qualys supports the Azure Deployment Policy to schedule auto-deployment of Cloud Agent Extensions on Windows and Linux VMs.

To learn more about deployment steps, refer to:
- **For Windows VM:** [Deploy Cloud Agent for Windows VMs via Azure Deployment Policy](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/ca_deployment_windows_vm_policy.htm)
- **For Linux VM:** [Deploy Cloud Agent for Linux VMs via Azure Deployment Policy](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/ca_deployment_linux_vm_policy.htm)

### Deploy Cloud Agent via PowerShell
Qualys supports the PowerShell command-line tool to deploy Cloud Agent Extensions on Windows and Linux VMs. To learn more about deployment steps, refer to:
- **For Windows VM:** [Deploy Cloud Agent For Windows via PowerShell](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/deploy_ca_powershell.htm)
- **For Linux VM:** [Deploy Cloud Agent For Linux via PowerShell](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/deploy_linux_ca_powershell.htm)

### Deploy Qualys Cloud Agent via Other Tool Sets
Qualys Cloud Agent can be deployed via automation, orchestration, or configuration management tools in your environment, such as Ansible, Chef, and Puppet. Qualys provides a template for deploying Qualys Cloud Agent via Ansible. You can use this to deploy and configure Qualys Cloud Agent in their Azure environment.

To learn more, refer to [Cloud Agent Deployment via Other Tool Sets](https://docs.qualys.com/en/integration/securing-azure/deploying_sensor/deploy_cat_tool_sets.htm).

## Support
Contact [Qualys Support](https://success.qualys.com/customersupport/s/) if you encounter an issue when deploying Cloud Agent on Azure VMs.
