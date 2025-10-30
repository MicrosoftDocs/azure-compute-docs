---
title: VM Customization
description: Feature that allows control over CPU resources of a virtual machine
author: eehindero
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 10/21/2025
ms.author: eehindero
ms.reviewer: mimckitt
---
# VM Customization Feature: Disable Simultaneous Multi-Threading (SMT/HT) and Configurable Constrained Cores (Preview)

VM Customization is a new Azure VM feature that gives you greater control over the CPU resources of a virtual machine. It consists of two related capabilities:

Disable Simultaneous Multi-Threading (Threads Per Core = 1): Allows you to run a VM with only one thread per physical CPU core, effectively turning off Simultaneous Multithreading (SMT). Disabling gives your VM full use of each physical core, which can improve performance for certain workloads (like some HPC or latency-sensitive applications) that benefit from exclusive core access.

Configurable Constrained Cores (Customize vCPUs): Allows you to **choose a custom number of vCPUs** for a new VM, lower than the default count for that VM size. It lets you allocate only the CPU cores you need, for example, to reduce licensing costs for software that is licensed per core (such as databases or analytics servers) while still getting the full memory and I/O of a larger VM.

Benefits: With these features, you can optimize VMs for both performance and cost:

_Performance:_ Disabling hyperthreading can provide more consistent and sometimes higher single-thread performance by eliminating contention between threads on the same core.

_Cost Optimization:_ Reducing the vCPU count of a VM can lower costs for software that charges per CPU. You could run a memory-intensive SQL Server on a VM with fewer vCPUs active, cutting SQL licensing fees, without paying for unused CPU capacity.

There's _no extra charge_ to use these CPU configuration options. The base VM price remains the same as if you deployed the full-size VM with default settings. However, customers get reduced licensing costs for software billed per vCPU.

## VM Customization Settings Configuration

You can configure the "Threads per core" and "vCPUs available" settings using the Azure portal, Azure Resource Manager templates (ARM), or command-line tools. 

## Azure portal

In the Azure portal, the VM creation workflow has a UI for these options. 

Start creating a VM as usual (for example, click **Create a resource > Virtual Machine** and fill out the Basics tab).

1. In the Size section of the Basics tab, select a VM size that you want to use. Under the size selection, click the "Customize cores" button. This opens more fields for VM Customization.

2. To disable SMT, set Threads per core to 1. (Leave it at 2 if you want to keep hyperthreading enabled.)

3. To  the vCPU count, set vCPUs available to your desired number of vCPUs. The portal provides valid values for the chosen VM size. 

4. Continue with the rest of the VM creation (set up disks, networking, etc.) and create the VM.

Once the VM is deployed, it has the specified number of vCPUs. If you set Threads per core to 1, the VM's OS sees half the usual number of processors (since hyperthreading is off). If you reduced vCPUs, it would see that lower count.

## Azure CLI

To disable SMT and configure cores during instance launch:

To disable SMT/HT, use the Azure CLI command and specify a value of 1 for vCPUsPerCore for the--cpu-options parameter. To configure cores, specify the number of CPU cores for vCPUsAvailable. In this example, to specify the default CPU core count for a Standard_D8s_v6 instance, specify a value of 8.

```azure cli
Az vm create --resource-group ccctest-rg-01 --name ccctestvm01 --image Ubuntu2204 --size Standard_D8s_v6 --location eastus2euap --admin-username azureuser --generate-ssh-keys --public-ip-address '""' --v-cpus-available 4 --v-cpus-per-core 1
```

## PowerShell

To disable SMT and configure cores during instance launch

Use PowerShell and specify the properties on the underlying configuration object. To disable SMT/HT, specify a value of 1 for vCPUsPerCore for the --cpu-options parameter. To configure cores, specify the number of CPU cores for vCPUsAvailable. 

```powershell
$vmConfig = New-AzVMConfig -VMName "MyVM" -VMSize "Standard_D8s_v6"

$vmConfig.HardwareProfile.VmSizeProperties = New-Object Microsoft.Azure.Management.Compute.Models.VMSizeProperties

$vmConfig.HardwareProfile.VmSizeProperties.VCPUsAvailable = 4

$vmConfig.HardwareProfile.VmSizeProperties.VCPUsPerCore = 1
```

Then proceed to set OS, network, etc., and use New-AzVM to create the VM. This approach uses the Azure PowerShell SDK objects directly to inject the values. 

## ARM Template (Azure Resource Manager)

For automation or scenarios where you need to deploy via infrastructure-as-code, you can use Azure Resource Manager templates to specify these CPU options. This can be used to disable SMT and deploy customized VMs via CLI or PowerShell as well (by deploying a template).

In your ARM template's resource definition for the virtual machine, the CPU options are specified under the hardwareProfile property of the VM. Specifically, you use vmSizeProperties inside hardwareProfile to set the values:

vCPUsPerCore - Set this to 1 to disable hyperthreading (for example, 1 thread per core). Omit this property or set to null/2 to use the default hyperthreading (2 threads per core).

vCPUsAvailable - Set this to the number of vCPUs you want active. If this property isn't set, the VM uses the default number of vCPUs for that size.

Here are brief examples of ARM template snippets for different scenarios:

## Disable SMT (SMT/HT Off)

This snippet shows the setting to turn off SMT on a VM (the VM uses 1 thread per core)


```json
"properties": {

"hardwareProfile": {

"vmSize": " Standard_D8s_v6",

"vmSizeProperties": {

"vCPUsPerCore": 1

}

},

...

}
```
In this case, if Standard_D8s_v6 normally has 8 vCPUs (4 cores * 2 threads), setting vCPUsPerCore: 1 means the VM has 4 vCPUs (one per core).

## Constrain vCPU Count (Customize Cores)

This snippet shows a VM configured to use a specific number of vCPUs (fewer than the default)


```json
"properties": {

"hardwareProfile": {

"vmSize": " Standard_D8s_v6",

"vmSizeProperties": {

"vCPUsAvailable": 2

}

},

...

}
```
Here, we requested two cores. On Standard_D8s_v6 (which is hyperthreaded by default), 2 _physical_ cores are allocated, and since SMT is still on by default (2 threads per core), the VM has 4 logical vCPUs. 

## Disable SMT and Customize vCPUs 

You can combine both settings as shown:


```json
"properties": {

"hardwareProfile": {

"vmSize": " Standard_D8s_v6",

"vmSizeProperties": {

"vCPUsPerCore": 1,

"vCPUsAvailable": 2

}

},

...

}
```
In this example, vCPUsPerCore: 1 disables SMT, and vCPUsAvailable: 2 then requests 2 vCPUs. With SMT off, those 2 correspond one-to-one with two physical cores (no threading). The VM has two logical processors in the OS. 

Make sure to use an API version **2021-07-01 or later** for the Microsoft.Compute/virtualMachines resource in your template, as that's when these properties were introduced.

## Considerations

Most Azure VM families support these features, but there are some important rules and considerations to understand when using the feature:

- You can only disable hyperthreading on VM sizes that use hyperthreading by default (for example, VMs with 2 threads per core).

- You can only _reduce_ the number of vCPUs, not increase it beyond the VM's default. The vCPUsAvailable value specified must be less than or equal to the default vCPU count of the chosen VM size. 

- On VM sizes that are hyperthreaded (default 2 threads/core), any custom vCPU count must be an even number. 

- You can disable hyperthreading and constrain vCPUs at the same time on the same VM. In this case, both above rules apply. 

- CPU options can only be specified at VM creation time or during a resize (redeploy) operation. You can't dynamically adjust the core count or SMT setting on a running VM without redeploying.

- If you move to a new VM size in the same family that also supports the feature, your settings are carried over by default. 

- If you resize to a VM size that doesn't support the setting, the operation is blocked or error. 

- Anytime you resize a VM (either within the same series or to a different series), a VM reboot occurs. Plan for downtime during the resize operation.

- In preview, only first-party Azure Marketplace images (Windows Server, Ubuntu, Red Hat, SUSE, etc.) and custom images are supported. 
