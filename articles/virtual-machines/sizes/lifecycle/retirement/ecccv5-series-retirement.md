---
title: ECccv5-series-retirement
description: Retirement information for the ECas_cc_v5, and ECads_cc_v5 nested confidential VM sizes. Before retirement, migrate your workloads to recommended options.
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: how-to
ms.date: 8/1/2026
author: angarg05
ms.author: ananyagarg
ms.custom: references_regions
---

<!-- DRAFT — cc_v5 nested confidential VM retirement migration guide. Modeled on the DCsv2-series retirement page. Owner/front-matter aliases and the items flagged below need confirmation before publishing. -->

# DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 series retirement - Azure Virtual Machines | Microsoft Learn

This migration guide is for users of the DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 nested confidential virtual machine (VM) sizes, which retire on **August 1, 2026**. To ensure minimal disruption and to continue optimizing cost and performance, this guide helps you transition to the latest series VMs.

This document covers:

- Recommended options for migration
- Detailed migration steps
- Frequently Asked Questions

By migrating to newer VM series, you gain access to improved price-performance ratios, broader regional availability, and the latest hardware capabilities.

## Recommended options for migration

**Before August 1, 2026**, migrate your workloads to one of the following options that best aligns with your business needs:

- **Running a general-purpose workload?** Migrate to a standard general-purpose VM series such as [Dasv5/Dadsv5](../../general-purpose/d-family.md) or [Easv5/Eadsv5](../../memory-optimized/e-family.md). These series offer improved price-performance and broad regional availability.
- **Using nested virtualization for non-confidential VMs?** Migrate to a general-purpose VM size that [supports nested virtualization](../../../../virtual-machines/sizes/resize-vm.md).
- **Running confidential container or confidential VM workloads?** To lift and shift into a VM-based programming model, consider [DCasv6/ECasv6](../../general-purpose/dcasv6-series.md) series (currently in preview), or [DCesv6](../../general-purpose/dcesv6-series.md) (currently in preview) confidential VMs (CVMs). If you already have or plan to transition to containerized workloads, consider using [Azure Confidential Container Instances (C-ACI)](../../../../container-instances/container-instances-confidential-overview.md) serverless infrastructure. If you need to orchestrate containerized workloads, consider using [Virtual nodes on Azure Container Instances (C-VN2) for Azure Kubernetes Service (AKS)](../../../../container-instances/container-instances-virtual-nodes.md).

To learn more about Azure confidential computing options, visit [https://aka.ms/azurecc](/azure/confidential-computing/confidential-vm-overview).

Additionally, there may be changes to your Azure Virtual Machines billing because of this retirement. Refer to our Azure Virtual Machines [pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) for more information.

## Migration Steps

Start planning your migration from the DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 series today.

### Identify the target migration option

- Learn more about the different migration options and their benefits in the section *Recommended options for migration*.
- Evaluate your current VM's workload and performance requirements and identify the target migration option.

### Check and request quota increases

- Before resizing and migrating, verify that your subscription has sufficient quota for the target VM series.
- Request more quota through the [Azure portal](../../../quotas.md) if needed.

### Complete migration

- Complete migration as soon as possible to prevent business impact and to take advantage of the improved performance and extensive regional coverage of the newer confidential computing offerings.
- Depending on the chosen migration option, follow the documentation in the following sections.

## Resize to a target VM series

If your target migration option is a VM series available in your current region, you can resize your virtual machines by using the Azure portal, PowerShell, or the CLI. The following examples show how to resize your VM.

> [!IMPORTANT]
> Resizing a virtual machine restarts the VM. Perform actions that result in a restart during off-peak business hours.
>
> Deallocating the VM also releases any dynamic IP addresses assigned to the VM. The OS and data disks aren't affected.

### Azure portal

1. Open the [Azure portal](https://portal.azure.com).
2. Type *virtual machines* in the search.
3. Under **Services**, select **Virtual machines**.
4. In the **Virtual machines** page, select the virtual machine you want to resize.
5. Stop (deallocate) the VM.
6. In the left menu under **Availability + scale**, select the **Size** option.
7. Pick a new size from the list of available sizes and select **Resize**.
8. **Start the VM** after resizing.

For more detailed instructions, see the full [Azure VM resizing guide](../../resize-vm.md).

### Azure PowerShell

1. Open a PowerShell session.
2. Authenticate your device and set your subscription.

    ```powershell
    Connect-AzAccount -UseDeviceAuthentication
    Set-AzContext -SubscriptionName "Your-Subscription-Name"
    ```
3. Set the resource group and VM name variables. Replace the values with information for the VM you want to resize.

    ```powershell
    $resourceGroup = "myResourceGroup"
    $vmName = "myVM"
    ```
4. Stop the VM.

    ```powershell
    Stop-AzVM -Name $vmName -ResourceGroupName $resourceGroup
    ```
5. List the VM sizes that are available on the hardware cluster where the VM is hosted.

    ```powershell
    Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName
    ```
6. Resize the VM to the new size.

    ```powershell
    $vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
    $vm.HardwareProfile.VmSize = "<newVMsize>"
    Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
    Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName
    ```

### Azure CLI

1. Open a terminal session and make sure Azure CLI is installed (or use the portal CLI).
2. Authenticate your device and set your subscription.

    ```azurecli
    az login
    az account set --subscription "<subscriptionIdHere>"
    ```
3. Set the resource group, VM name, and new size variables. Replace the values with information of the VM you want to resize.

    ```azurecli
    resourceGroup="myResourceGroup"
    vmName="myVM"
    newSize=<newVMsize>
    ```
4. Resize the VM to the new size.

    ```azurecli
    az vm deallocate --resource-group $resourceGroup --name $vmName
    az vm resize --resource-group $resourceGroup --name $vmName --size $newSize
    az vm start --resource-group $resourceGroup --name $vmName
    ```

## Frequently asked questions	

### How does this retirement affect me?

If you're running your workload on the DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, or ECads_cc_v5 series—either by using virtual machines, Virtual Machine Scale Sets, or services built on top of these SKUs—this retirement affects you - this retirement affects you because you might lose your workloads and your VMs are shut down when the SKUs are retired.

### What is the migration timeline?

On August 1, 2026, the DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 series virtual machines (VMs) are retired. Before that date, migrate your workloads to one of the recommended options described in *Recommended options for migration*.

### Will these series still allow new customer sign-ups?

**Starting June 30, 2026**, capacity restrictions are applied to the DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 series, and no new subscriptions are allowed.

### Will Microsoft continue to support my current workload?

Yes, support continues for your workloads on these VMs until the retirement date. You continue to receive SLA assurance, infrastructure updates, and maintenance until August 1, 2026. Use one of the recommended migration options before that date to preserve your workloads.

### Will other services built on top of these SKUs still be available after the SKUs retire?

No, Microsoft isn't taking feature requests or building new features for these series. Instead, the company is focusing on next-generation offerings with broader regional availability and a cloud-native approach with containerized workloads and serverless infrastructure.

### Will these series provide any new features during the retirement period?

No, we aren't taking feature requests or building new features for these series. Instead, we're focusing on next-generation offerings with broader regional availability and a cloud-native approach with containerized workloads and serverless infrastructure.

### How can I get a quota for the target VM size?

Follow the guide to [request an increase in vCPU quota by VM family](/azure/quotas/per-vm-quota-requests).

### Are my OS and data disks affected when resizing?

Deallocating the VM also releases any dynamic IP addresses assigned to the VM. The OS and data disks aren't affected.

### What impact will migration have on my dynamic IP addresses?

Deallocating the VM releases any dynamic IP addresses assigned to the VM. If you need to retain the same IP addresses, consider using static IP addresses and set them in the network settings ahead of migration.

### How does migration affect my current billing?

There may be a price change when you switch SKUs. For more information, see the Azure Virtual Machines [pricing page](https://azure.microsoft.com/pricing/calculator/).

### How can I get transition help and support during migration?

If you have any questions, you can [create a support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) through the Azure portal for technical help.

### What will happen after the retirement date?

After August 1, 2026, any remaining DCas_cc_v5, DCads_cc_v5, ECas_cc_v5, and ECads_cc_v5 series virtual machine subscriptions will stop working and will no longer incur billing charges. To avoid disruption, migrate ahead of the retirement schedule.

## Help and support

If you have a support plan and need technical help, [create a support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest):

1. In the [Help + support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) page, select **Create a support request**. Follow the **New support request** page instructions. Use the following values:
    - For **Issue type**, select **Technical**.
    - For **Service**, select **My services**.
    - For **Service type**, select **Virtual Machine running Windows/Linux**.
    - For **Resource**, select your VM.
    - For **Problem type**, select **Assistance with resizing my VM**.
    - For **Problem subtype**, select the option that applies to you.

Follow instructions in the **Solutions** and **Details** tabs, as applicable, and then **Review + create**.
