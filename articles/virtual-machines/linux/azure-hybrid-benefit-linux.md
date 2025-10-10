---
title: Explore Azure Hybrid Benefit for Linux VMs
description: Learn how to use Azure Hybrid Benefit on Linux virtual machines and how it can help you reduce costs.
services: virtual-machines
author: MarckK
manager: clausw
ms.service: azure-virtual-machines
ms.subservice: billing
ms.collection: linux
ms.topic: concept-article
ms.date: 12/19/2024
ms.author: clausw
ms.reviewer: mattmcinnes
ms.custom: kr2b-contr-experiment, linux-related-content, devx-track-azurecli
# Customer intent: "As a cloud engineer managing Linux virtual machines, I want to utilize Azure Hybrid Benefit to optimize my subscription model, so that I can reduce costs and maintain flexibility without incurring downtime."
---

# Azure Hybrid Benefit for Linux virtual machines

You can use Azure Hybrid Benefit for Linux to easily switch the software subscription model for your Linux virtual machine (VM). Change your subscription model without redeploying your VM and with no risk of downtime. You gain flexibility and savings.

You can switch seamlessly between two subscription models on Azure by using Azure Hybrid Benefit:

* **Bring your own subscription (BYOS)**: In the BYOS model, you bring your own Red Hat Enterprise Linux (RHEL) or SUSE Linux Enterprise Server (SLES) subscription directly to Azure. You pay only for the infrastructure costs of your VM on Azure. The software fee is covered by your RHEL or SLES subscription you bought from the vendor.

* **Pay-as-you-go (PAYG)**: Use the pay-as-you-go subscription model in Azure to pay for RHEL and SLES subscriptions as you use them.

This article defines the BYOS and PAYG subscription models, compares the benefits of each model, and shows you how to use the Azure Hybrid Benefit to switch between the two subscription models for your Linux VMs on Azure.

This process applies to:

* Azure Virtual Machine Scale Sets
* Azure Spot Virtual Machines
* Custom images

Azure Hybrid Benefit gives you the option to make seamless bidirectional conversions between the two subscription models on eligible VM instances.

You might see combined savings estimated to up to 76% with Azure Hybrid Benefit for Linux and three-year Azure Reserved VM Instances. Savings estimates are based on one standard D2s v5 Azure VM, with an RHEL or SLES subscription, in the East US region, running at a pay-as-you-go rate versus a reduced rate for a three-year reserved instance plan. The savings estimates are based on Azure pricing as of September 2024. Prices are subject to change. Actual savings might vary based on location, instance type, or usage.

> [!TIP]
> Try the [Azure Pricing calculator](https://azure.microsoft.com/pricing/calculator/) to visualize the cost-saving benefits of this feature at a Virtual Machine level.

## PAYG vs. BYOS

Azure offers two main licensing pricing options: PAYG and BYOS. With PAYG, you pay only for the resources you use. You can scale up or scale down as needed.

With BYOS, you can use your existing license subscriptions. You don't pay the license fees in Azure.

:::image type="content" source="./media/ahb-linux/azure-hybrid-benefit-compare.png" alt-text="Diagram that shows the use of Azure Hybrid Benefit to switch Linux VMs between pay-as-you-go and bring your own subscription models.":::

VMs utilizing a PAYG subscription model, whether deployed from PAYG images or VMs converted to PAYG from BYOS, incur *both* an infrastructure fee and a software fee.

If you have an existing software subscription, use Azure Hybrid Benefit to convert from a PAYG model to a BYOS model.

You can use Azure Hybrid Benefit to switch between the two subscription options at any time.

## Linux VMs you can use with Azure Hybrid Benefit

### PAYG

The following PAYG RHEL and SLES Marketplace offers are eligible to use with Azure Hybrid Benefit:

##### [RHEL PAYG](#tab/ahbRhelPayg)

## Limitations

Only RHEL images *published by Red Hat, Inc.* are eligible for the PAYG to BYOS conversion using the Azure Hybrid Benefit. Images from other vendors aren't supported.

### Red Hat-published RHEL PAYG offers eligible for Azure Hybrid Benefit

The following Red Hat-published RHEL PAYG offers are eligible to use with the Azure Hybrid Benefit. Links to the offers in Azure Marketplace are included.

Within these offers, associated images are described as "Pay-As-You-Go".

* [Red Hat Enterprise Linux (RHEL)](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-20190605?tab=Overview)
* [Red Hat Enterprise Linux (RHEL)](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rh-rhel?tab=Overview)
* [Red Hat Enterprise Linux (RHEL) for SAP Apps](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-sap-apps?tab=Overview)
* [RHEL for SAP with HA and Update Services](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-sap-ha?tab=Overview)
* [Red Hat Enterprise Linux (RHEL) with High Availability (HA) Add-On](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-ha?tab=Overview)
* [Red Hat Enterprise Linux (RHEL) Arm64](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-arm64?tab=Overview)
* [Red Hat Enterprise Linux Confidential VM](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-cvm?tab=Overview)
* [Red Hat Enterprise Linux (RHEL) RAW](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/redhat.rhel-raw?tab=Overview)

##### [SLES PAYG](#tab/ahbSlesPayg)

## Limitations

Only SLES images *published by SUSE* are eligible to use for the PAYG to BYOS conversion with the Azure Hybrid Benefit. Images from other vendors aren't supported.

### SUSE-published SLES PAYG offers eligible for Azure Hybrid Benefit

The following SUSE-published SLES PAYG offers are eligible for Azure Hybrid Benefit. Links to the offers in Azure Marketplace are included.

Some of the offerings are end-of-support or end-of-life (EOL) (see [Product Support Lifecycle Dates](https://www.suse.com/lifecycle)) and only valid with additional Long Term Servicepack Support (LTSS) subscription, purchased directly from SUSE.

Within these offers, associated plans and images are described as a "Pay-As-You-Go" subscription of SLES.

**SUSE Linux Enterprise Server**

* [ SUSE Linux Enterprise Server 15 SP7 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp7?tab=Overview)
* [ SUSE Linux Enterprise Server 15 SP7 Arm64 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp7-arm64?tab=PlansAndPrice)
* [ SUSE Linux Enterprise Server 15 SP7 + Patching](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp7-basic?tab=Overview)

 _SLES 15 SP6 - End-of-Support: 31-Dec-2025 - LTSS 31-Dec-2028_
* [ SUSE Linux Enterprise Server 15 SP6 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp6?tab=Overview)
* [ SUSE Linux Enterprise Server 15 SP6 Arm64 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp6-arm64?tab=PlansAndPrice)
* [ SUSE Linux Enterprise Server 15 SP6 + Patching](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp6-basic?tab=Overview)

 _SLES 15 SP5 - End-of-Support: 31-Dec-2024 - LTSS 31-Dec-2027_
* [ SUSE Linux Enterprise Server 15 SP5 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp5?tab=Overview)
* [ SUSE Linux Enterprise Server 15 SP5 Arm64 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp5-arm64?tab=Overview)
* [ SUSE Linux Enterprise Server 15 SP5 + Patching](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-15-sp5-basic?tab=Overview)


**SUSE Linux Enterprise Server for SAP applications**

* [ SUSE Linux Enterprise Server for SAP applications 15 SP7 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp7?tab=Overview)
* [ SUSE Linux Enterprise Server for SAP applications 15 SP7 Hardened + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp7-hardened?tab=Overview)

* [ SUSE Linux Enterprise Server for SAP applications 15 SP6 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp6?tab=Overview)
* [ SUSE Linux Enterprise Server for SAP applications 15 SP6 Hardened + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp6-hardened?tab=Overview)

* [ SUSE Linux Enterprise Server for SAP applications 15 SP5 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp5?tab=Overview)
* [ SUSE Linux Enterprise Server for SAP applications 15 SP5 Hardened + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp5-hardened?tab=Overview)

* [ SUSE Linux Enterprise Server for SAP applications 15 SP4 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp4?tab=Overview)
* [ SUSE Linux Enterprise Server for SAP applications 15 SP4 Hardened + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp4-hardened?tab=Overview)

 _SLES for SAP applications 15 SP3 - End-of-Support: 31-Dec-2025_
* [ SUSE Linux Enterprise Server for SAP applications 15 SP3 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-15-sp3?tab=Overview)

 _SLES for SAP applications 12 SP5 - End-of-Support: 31-Oct-2024 - LTSS 31-Oct-2027_
* [ SUSE Linux Enterprise Server for SAP applications 12 SP5 + 24x7 Support](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/suse.sles-sap-12-sp5?tab=Overview)

---

### BYOS

Azure Hybrid Benefit is also available for RHEL and SLES BYOS Azure Marketplace images and images brought from on-premises or other cloud providers.

Currently, one RHEL BYOS offer is available. This offer is a private listing. To gain access to this private listing, you must join the Red Hat Cloud Access program.

You can identify SUSE BYOS marketplace offers by their name, which includes "BYOS". An example is the ** SUSE Linux Enterprise Server 15 SP7 - BYOS** offering.

Azure dedicated host instances and SQL hybrid benefits aren't eligible to use with Azure Hybrid Benefit if you already use Azure Hybrid Benefit with Linux VMs.

> [!NOTE]
> For Red Hat VMs, the Azure account must be part of the Red Hat Cloud Access program. Register with Red Hat Cloud Access before you try to enable Azure Hybrid Benefit on your VMs.

## Enable Azure Hybrid Benefit

You can enable Azure Hybrid Benefit on new VMs, on existing VMs, and on multiple VMs.

### New VM

You can invoke Azure Hybrid Benefit when you create a VM. The benefits of using this approach include:

* You can provision both PAYG and BYOS VMs by using the same image and process.
* You can change the licensing mode in the future.
* The PAYG VM is connected to Red Hat Update Infrastructure (RHUI) or SUSE Cloud Update Infrastructure (CUI) by default, to help keep it up to date and secure. You can change the update method after deployment.

#### [Azure portal](#tab/ahbNewPortal)

The SUSE workflow is the same as the RHEL example shown here.

To enable Azure Hybrid Benefit when you create a VM:

1. In the [Azure portal](https://portal.azure.com/), go to **Create a virtual machine**.

   ![Screenshot of the portal pane for creating a virtual machine.](./media/azure-hybrid-benefit/create-vm-ahb.png)
1. In the **Licensing** section, select the checkbox that asks if you want to use an existing RHEL subscription. Select the checkbox to confirm that your subscription is eligible.

   ![Screenshot of the Azure portal that shows checkboxes selected for licensing.](./media/azure-hybrid-benefit/create-vm-ahb-checkbox.png)
1. Create a virtual machine by following the steps that are presented.
1. On the VM service menu, select **Operating System**. Under **Licensing**, verify that the option is enabled.

   ![Screenshot of the Azure Hybrid Benefit configuration pane after you create a virtual machine.](./media/azure-hybrid-benefit/azure-hybrid-benefit.png)

#### [Azure CLI](#tab/ahbNewCli)

You can use the `az vm extension` and `az vm update` commands to update a new VM after you create them.

> [!NOTE]
> You do **not** need to install the extension for SUSE offerings.

1. Install the extension:

   ```azurecli
   az vm extension
   ```

1. Update the VM with the correct license type:

   ```azurecli
   az vm update
   ```

RHEL license types:

* RHEL_BASE
* RHEL_EUS
* RHEL_SAPAPPS
* RHEL_SAPHA
* RHEL_BASESAPAPPS
* RHEL_BASESAPHA​

SLES license types:

* SLES
* SLES_SAP
* SLES_HPC​

---

### Existing VM

You can enable Azure Hybrid Benefit on an existing VM.

#### [Azure portal](#tab/ahbExistingPortal)

To enable Azure Hybrid Benefit on an existing VM:

1. In the [Azure portal](https://portal.azure.com/), go to the overview pane of the VM that you want to convert.
1. Go to **Operating System** > **Licensing**. To enable the Azure Hybrid Benefit conversion, select **Yes**, and then select the confirmation checkbox.

![Screenshot of the Azure portal that shows the Licensing section of the configuration page for Azure Hybrid Benefit.](./media/azure-hybrid-benefit/azure-hybrid-benefit.png)

#### [Azure CLI](#tab/ahbExistingCli)

You can use the `az vm extension` and `az vm update` commands to update an existing VM.

> [!NOTE]
> You do **not** need to install the extension for SUSE offerings.

1. Install the extension:

   ```azurecli
   az vm extension
   ```

   > [!NOTE]
   > The complete `az vm extension` command depends on the particular distribution you're using. For complete information, see the next section.

1. Update the VM with the correct license type:

   ```azurecli
   az vm update
   ```

RHEL license types:

* RHEL_BASE
* RHEL_EUS
* RHEL_SAPAPPS
* RHEL_SAPHA
* RHEL_BASESAPAPPS
* RHEL_BASESAPHA​

SLES license types:

* SLES
* SLES_SAP
* SLES_HPC​
---

## Check the current licensing model of a VM that has Azure Hybrid Benefit enabled

With Red Hat, the Azure Hybrid Benefit extension must be installed on the VM to switch the licensing model from BYOS to PAYG or vice versa. You can install the Azure Hybrid Benefit (AHB) extension for RHEL VMs by using the following CLI command.

   ```azurecli
   az vm extension set -n AHBForRHEL --publisher Microsoft.Azure.AzureHybridBenefit --vm-name myVMName --resource-group myResourceGroup
   ```

    
For SUSE you do not need the extension, the functionality is covered by the SUSE tooling itself.

### [Azure CLI](#tab/licenseazcli)

1. Red Hat: You can use the `az vm get-instance-view` command to check whether the extension is installed. Look for the `AHBForRHEL` extension. If the corresponding extension is installed, the Azure Hybrid Benefit is enabled. Review the license type to determine which licensing model is applied to your VM.

   ```azurecli
   az vm get-instance-view -g MyResourceGroup -n myVm --query instanceView.extensions
   ```

1. Red Hat and SUSE: Use the following command to review the license type applied to the VM:

   ```azurecli
   az vm get-instance-view -g MyResourceGroup -n myVM --query licenseType
   ```

   The following license types correspond to a **PAYG** subscription model:

   For RHEL:

   * RHEL_BASE
   * RHEL_EUS
   * RHEL_SAPAPPS
   * RHEL_SAPHA
   * RHEL_BASESAPAPPS
   * RHEL_BASESAPHA

   For SLES:

   * SLES
   * SLES_SAP
   * SLES_HPC

   These license types correspond to a **BYOS** subscription model:

   For RHEL:

   * RHEL_BYOS

   For SLES:

   * SLES_BYOS

If the license type of the VM isn't modified, this command returns an empty string and the VM continues to use the billing model of the image that you used to deploy it.

### [Azure PowerShell](#tab/licensepowershell)

1. Red Hat: You can use the `az vm get-instance-view` command to check whether the extension is installed. Look for the `AHBForSLES` or `AHBForRHEL` extension. If the corresponding extension is installed, the Azure Hybrid Benefit is enabled. Review the license type to determine which licensing model is applied to your VM.

   ```azurepowershell
   Get-AzVM -ResourceGroupName MyResourceGroup -Name myVM -Status | Select-Object -ExpandProperty Extensions
   ```

1. Red Hat and SUSE: Use the following command to review the VM's license type:

   ```azurepowershell
   (Get-AzVM -ResourceGroupName MyResourceGroup -Name myVM).LicenseType
   ```

   The following license types correspond to a **PAYG** subscription model:

   For RHEL:

   * RHEL_BASE
   * RHEL_EUS
   * RHEL_SAPAPPS
   * RHEL_SAPHA
   * RHEL_BASESAPAPPS
   * RHEL_BASESAPHA

   For SLES:

   * SLES
   * SLES_SAP
   * SLES_HPC

   These license types correspond to a **BYOS** subscription model:

   For RHEL:

   * RHEL_BYOS

   For SLES:

   * SLES_BYOS

   **If the license type of the VM isn't modified, this command returns an empty string and the VM continues to use the billing model of the image that you used to deploy it.**

---

## Convert between PAYG and BYOS subscription models

Use the Azure CLI to convert a PAYG Azure Marketplace image to a BYOS subscription model, or to convert a BYOS or migrated VM to a PAYG subscription model.

---

### Convert a PAYG image to BYOS by using the Azure CLI

If you deployed an Azure Marketplace image by using a PAYG licensing model and want to convert licensing to BYOS, complete the following steps.

#### [Red Hat (RHEL)](#tab/rhelAzcliByosConv)

1. Apply the `RHEL_BYOS` license type to the VM:

    ```azurecli
    # This enables BYOS on a RHEL PAYG VM by using Azure Hybrid Benefit.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS
    ```

1. When the PAYG to BYOS conversion is finished, you must register the VM with Red Hat for system updates and usage compliance.

1. If you want to return the original subscription model, set `license-type` to `None`.

    ```azurecli
    # In order to revert back to the original licensing model, set license-type to None.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```

#### [SUSE (SLES)](#tab/slesAzcliByosConv)

1. Apply the `SLES_BYOS` license type to the VM:

    ```azurecli
    # This enables BYOS on a SLES virtual machine.
    az vm update -g myResourceGroup -n myVmName --license-type SLES_BYOS
    ```

1. When the conversion from PAYG to BYOS is finished, you must register the VM directly with SUSE for software updates and usage compliance.

1. If you want to return the original subscription model, set `license-type` to `None`.

   ```azurecli
    # In order to revert back to the original licensing model, set license-type to None.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
   ```
> [!NOTE]
> This is not possible for the SUSE offerings "+ Patching", they can only be moved back to "SLES".

---

### Convert BYOS to PAYG

Converting to a PAYG subscription model is supported for Azure Marketplace images labeled "BYOS" and for machines imported from on-premises or from a third-party cloud provider.

#### [Red Hat (RHEL)](#tab/rhelazclipaygconv)

1. Install the Azure Hybrid Benefit extension on a running VM. You can use the following command via the Azure CLI:

    ```azurecli
    az vm extension set -n AHBForRHEL --publisher Microsoft.Azure.AzureHybridBenefit --vm-name myVMName --resource-group myResourceGroup
    ```

1. After the extension is installed successfully, change the license type based on what you need:

    ```azurecli
    # This enables Azure Hybrid Benefit to fetch software updates for RHEL base/regular repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASE

    # This enables Azure Hybrid Benefit to fetch software updates for RHEL EUS repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_EUS

    # This enables Azure Hybrid Benefit to fetch software updates for RHEL SAP APPS repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_SAPAPPS

    # This enables Azure Hybrid Benefit to fetch software updates for RHEL SAP HA repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_SAPHA

    # This enables Azure Hybrid Benefit to fetch software updates for RHEL BASE SAP APPS repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASESAPAPPS

    # This enables Azure Hybrid Benefit to fetch software updates for RHEL BASE SAP HA repositories.
    az vm update -g myResourceGroup -n myVmName --license-type RHEL_BASESAPHA
   ```

1. Check to see if the "AHB for RHEL" feature flag is enabled:

    ```azurecli
    az feature list --namespace Microsoft.Compute | grep "AHBEnabledForRHEL" -A 3
    ```

1. If you want to return the original subscription model, set `license-type` to `None`.
    ```azurecli
    # In order to revert back to the original licensing model, set license-type to None.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```

#### [SUSE (SLES)](#tab/slesazclipaygconv)

> [!NOTE]
> You can NOT switch products, you can only switch between the billing types.

1. You do *not* need to install the Azure Hybrid Benefit extension on a running VM.

1. Simply change the license type based on what you need:

   ```azurecli
    # This enables Azure Hybrid Benefit to fetch software updates for SLES repositories.
    az vm update -g myResourceGroup -n myVmName --license-type SLES

    # This enables Azure Hybrid Benefit to fetch software updates for SLES SAP repositories.
    az vm update -g myResourceGroup -n myVmName --license-type SLES_SAP

    # This enables Azure Hybrid Benefit to fetch software updates for SLES HPC repositories.
    az vm update -g myResourceGroup -n myVmName --license-type SLES_HPC
   ```

1. If you want to return the original subscription model, set `license-type` to `None`.
    ```azurecli
    # In order to revert back to the original licensing model, set license-type to None.
    az vm update -g myResourceGroup -n myVmName --license-type NONE
    ```

---

### Multiple VMs

The following command converts the VMs that are specified in the argument to BYOS:

```azurecli
# This enables BYOS on a RHEL virtual machine. In this example, ids.txt is an
# existing text file that contains a delimited list of resource IDs corresponding
# to the virtual machines that use Azure Hybrid Benefit.
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS --ids $(cat ids.txt)
```

The following examples show two methods you can use to get a list of resource IDs. One method applies to a resource group, and one method applies to the subscription.

```azurecli
# To get a list of all the resource IDs in a resource group:
az vm list -g MyResourceGroup --query "[].id" -o tsv

# To get a list of all the resource IDs of virtual machines in a subscription:
az vm list -o json | jq '.[] | {VirtualMachineName: .name, ResourceID: .id}'
```

---

### Convert the license type in the VM operating system

#### [Red Hat (RHEL)](#tab/rhelpaygconversion)

To start using Azure Hybrid Benefit for Red Hat:

1. Install the `AHBForRHEL` extension on the VM on which you want to apply the Azure Hybrid Benefit BYOS benefit. You can install the extension by using the Azure CLI or Azure PowerShell.

1. Depending on the software updates that you want, change the license type to a relevant value. Here are the available license type values and the software updates associated with them:

    | License type  | Software updates  | Allowed VMs|
    |---|---|---|
    | RHEL_BASE  | Installs Red Hat regular/base repositories on your VM. | RHEL BYOS VMs, RHEL custom image VMs|
    | RHEL_EUS | Installs Red Hat Extended Update Support (EUS) repositories on your VM. | RHEL BYOS VMs, RHEL custom image VMs|
    | RHEL_SAPAPPS  | Installs RHEL for SAP Business Apps repositories on your VM. | RHEL BYOS VMs, RHEL custom image VMs|
    | RHEL_SAPHA | Installs RHEL for SAP with High Availability (HA) repositories on your VM. | RHEL BYOS VMs, RHEL custom image VMs|
    | RHEL_BASESAPAPPS | Installs RHEL regular/base SAP Business Apps repositories on your VM. | RHEL BYOS VMs, RHEL custom image VMs|
    | RHEL_BASESAPHA | Installs regular/base RHEL for SAP with HA repositories on your VM.| RHEL BYOS VMs, RHEL custom image VMs|

1. Wait one hour for the extension to read the license type value and install the repositories.

> [!NOTE]
> If the extension isn't running by itself, you can run it on demand.

   You should now be connected to Azure Red Hat Update. The relevant repositories are installed on your machine.

1. If you want to switch back to the BYOS model, set `license-type` to `None` and run the extension. This action removes all RHUI repositories from your VM and stops associated billing.

> [!NOTE]
> In the unlikely event that the extension can't install repositories or if there are any other issues, switch the license type back to empty and contact Microsoft support. Taking this step ensures that you aren't billed for software updates.

#### [SUSE (SLES)](#tab/slespaygconversion)

You can simply use the `az vm update` command to update the existing license type on your running VMs. For SLES VMs, run the command and set the `--license-type` parameter to one of the following license types: `SLES`, `SLES_SAP`, or `SLES_HPC`.

> [!NOTE]
> You can NOT switch products, you can only switch between the billing types.

To start using Azure Hybrid Benefit for SLES VMs:

1. Change the license type to the value that reflects the software updates you want. Here are the available license type values and the software updates associated with them:

    | License type  | Software updates  | Allowed VMs|
    |---|---|---|
    | SLES | Installs SLES Standard repositories on your VM. | SLES BYOS VMs, SLES custom image VMs|
    | SLES_SAP | Installs SLES SAP repositories on your VM. | SLES SAP BYOS VMs, SLES custom image VMs|
    | SLES_HPC | Installs SLES High Performance Computing repositories on your VM. | SLES HPC BYOS VMs, SLES custom image VMs|

1. Wait a little until the process reads the license type value and install the repositories.

   You should now be connected to the SUSE public cloud update infrastructure on Azure. The relevant repositories are installed on your machine.

1. If you want to switch back to the BYOS model, set `license-type` to `None`. This action removes all repositories from your VM and stops associated billing.

You can use the `az vm update` command to update the existing license type on your running VMs. For SLES VMs, run the command and set the `--license-type` parameter to one of the following license types: `SLES`, `SLES_SAP`, or `SLES_HPC`.

---

## Azure Hybrid Benefit for reserved instance VMs

[Azure reservations](/azure/cost-management-billing/reservations/save-compute-costs-reservations) (Azure Reserved Virtual Machine Instances) help you save money by committing to one-year or three-year plans for multiple products. Azure Hybrid Benefit for PAYG VMs is available for reserved instances.

If you purchased compute costs at a discounted rate by using reserved instances, you can apply Azure Hybrid Benefit on the licensing costs for RHEL and SUSE. The steps to apply Azure Hybrid Benefit for a reserved instance are exactly the same as for a regular VM.

![Screenshot of the interface for purchasing reservations for VMs.](./media/azure-hybrid-benefit/reserved-instances.png)

> [!NOTE]
> If you already purchased reservations for RHEL or SUSE PAYG software in Azure Marketplace, wait for the reservation tenure to finish before you use Azure Hybrid Benefit for PAYG VMs.

## Compliance

### [Red Hat compliance](#tab/rhelcompliance)

Customers who use Azure Hybrid Benefit for PAYG RHEL VMs agree to the standard [legal terms](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) and [privacy statement](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) associated with Azure Marketplace RHEL offers.

Customers who use Azure Hybrid Benefit for PAYG RHEL VMs have three options for providing software updates and patches to those VMs:

* [Red Hat Update Infrastructure](../workloads/redhat/redhat-rhui.md) (default option)
* Red Hat Satellite Server
* Red Hat Subscription Manager

Customers can use RHUI as the main update source for Azure Hybrid Benefit for PAYG RHEL VMs without attaching subscriptions. Customers who choose the RHUI option are responsible for ensuring RHEL subscription compliance.

Customers using either Red Hat Satellite Server or Red Hat Subscription Manager should remove the RHUI configuration and attach a cloud-access-enabled RHEL subscription to Azure Hybrid Benefit for PAYG RHEL VMs.

For more information about Red Hat subscription compliance, software updates, and sources for Azure Hybrid Benefit for PAYG RHEL VMs, see the [Red Hat article about using RHEL subscriptions with Azure Hybrid Benefit](https://access.redhat.com/articles/5419341).

Customers who use the Azure Hybrid Benefit BYOS to PAYG capability for RHEL agree to the standard [legal terms](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) and [privacy statement](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) associated with Azure Marketplace RHEL offerings.

### [SUSE compliance](#tab/slescompliance)

To use Azure Hybrid Benefit for PAYG SLES VMs, and to get information about moving from SLES PAYG to BYOS or moving from SLES BYOS to PAYG, see [Azure Hybrid Benefit](https://aka.ms/suse-ahb) knowledge base article.

Customers who use Azure Hybrid Benefit for PAYG SLES VMs need to move the cloud update infrastructure to one of three options that provide software updates and patches to those VMs:

* [SUSE Customer Center](https://scc.suse.com)
* SUSE Multi-Linux Manager (available in the Azure Marketplace)
* SUSE Repository Mirroring Tool (part of SLES)

If you use the Azure Hybrid Benefit BYOS to PAYG capability for SLES and want more information about moving from SLES PAYG to BYOS, or moving from SLES BYOS to PAYG, see [Azure Hybrid Benefit Support](https://aka.ms/suse-ahb) knowledge base article.

---

## Frequently asked questions

* **Q: Can I use a license type of `RHEL_BYOS` with a SLES image or vice versa?**

  * A: No, you can't. Trying to enter a license type that incorrectly matches the distribution running on your VM will not update any billing metadata. But if you accidentally enter the wrong license type, updating your VM again to the correct license type still enables Azure Hybrid Benefit.

* **Q: I've registered with Red Hat Cloud Access but still can't enable Azure Hybrid Benefit on my RHEL VMs. What should I do?**

  * A: It might take some time for your Red Hat Cloud Access subscription registration to propagate from Red Hat to Azure. If you still see the error after one business day, contact Microsoft support.

* **Q: I've deployed a VM by using a RHEL BYOS "golden image." Can I convert the billing on this image from BYOS to PAYG?**

  * A: Yes, you can use Azure Hybrid Benefit for BYOS VMs to convert this subscription model.

* **Q: I've uploaded my own RHEL or SLES image from on-premises (via Azure Migrate, Azure Site Recovery, or otherwise) to Azure. Can I convert the billing on these images from BYOS to PAYG?**

  * A: Yes, you can use Azure Hybrid Benefit for BYOS VMs to convert this subscription model.

* **Q: I've uploaded my own RHEL or SLES image from on-premises (via Azure Migrate, Azure Site Recovery, or otherwise) to Azure. Do I need to do anything to benefit from Azure Hybrid Benefit?**

  * A: No, you don't. RHEL or SLES images that you upload are already considered BYOS, and you're charged only for Azure infrastructure costs. You're responsible for RHEL or SLES subscription costs, just as you are for your on-premises environments.

* **Q: Can I use Azure Hybrid Benefit for PAYG VMs for Azure Marketplace RHEL and SLES SAP images?**

  * A: Yes. You can use the license type of RHEL_BYOS for RHEL VMs and SLES_BYOS for conversions of VMs deployed from Azure Marketplace RHEL and SLES SAP images.

* **Q: Can I use Azure Hybrid Benefit for PAYG VMs on virtual machine scale sets for RHEL and SLES?**

  * A: Yes. Azure Hybrid Benefit on virtual machine scale sets for RHEL and SLES is available to all users. You can learn more about this benefit and how to use it.

* **Q: Can I use Azure Hybrid Benefit for PAYG VMs on reserved instances for RHEL and SLES?**

  * A: Yes. Azure Hybrid Benefit for PAYG VMs on reserved instances for RHEL and SLES is available to all users.

* **Q: Can I use Azure Hybrid Benefit for PAYG VMs on a VM deployed for SQL Server on RHEL images?**

  * A: No, you can't. There's no plan to support these VMs.

* **Q: Can I use Azure Hybrid Benefit on my RHEL for Virtual Datacenters subscription?**

  * A: No. RHEL for Virtual Datacenters isn't supported on Azure at all, including for Azure Hybrid Benefit.

## Related content

* [Learn how to create and update VMs and add license types (RHEL_BYOS, SLES_BYOS) for Azure Hybrid Benefit by using the Azure CLI](/cli/azure/vm)
* [Learn about Azure Hybrid Benefit on virtual machine scale sets for RHEL and SLES and how to use it](../../virtual-machine-scale-sets/azure-hybrid-benefit-linux.md)
