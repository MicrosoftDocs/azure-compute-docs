---
title: Configure VM watch
description: Getting started guide on how to configure VM watch on virtual machines
author:      ofemifowode 
ms.author:   ofemifowode 
ms.service: azure-virtual-machines
ms.topic: get-started
ms.update-cycle: 180-days
ms.date:     10/28/2024
ms.custom: portal
# Customer intent: "As a cloud administrator, I want to configure VM watch settings on my virtual machines, so that I can customize monitoring signals and optimize performance according to my specific environment requirements."
---

# Configure VM watch

VM watch signals are logically grouped into Collectors Suite, which can be categorized into two groups: [core and optional](/azure/virtual-machines/vm-watch-collector-suite). By default, only core group collectors are enabled with default configurations. However, these default settings can be easily overwritten from `vmWatchSettings` using either [ARM template](/azure/azure-resource-manager/templates/), [Azure CLI](/cli/azure/), or [PowerShell](/powershell/).

This article describes how to configure VM watch to suit your specific requirements.

### Prerequisites
This article assumes that you're familiar with:
- [VM watch checks, metrics, and logs](/azure/virtual-machines/azure-vm-watch)
- [Installing VM watch to virtual machines and scale sets](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)
- [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite)

### Access `vmWatchSettings` on Azure virtual machines

> [!IMPORTANT]
> The code segment is identical for both Windows and Linux except for the value of the parameter `<application health extension type>` passed into the Extension Type. Replace `<application health extension type>` with `ApplicationHealthLinux` for Linux and `ApplicationHealthWindows` for Windows installations.  

#### [ARM Template](#tab/ARM-template-1)
1. Navigate to the Overview page on [Azure portal](https://portal.azure.com/) and click on the JSON view for the VM to find the code segment below.
2. Copy the code segment to an IDE such as Visual Studio Code and make customizations as needed

```
{
   "settings": {
      "vmWatchSettings": {
         "enabled": true
      }
   }
}
```

#### [CLI](#tab/cli-1)
```
az vm extension show -g <your resource group name> --vm-name <your vm name> -n <application health extension type>
```
#### [PowerShell](#tab/powershell-1)
```
Get-AzVMExtension -ResourceGroupName "<your resource group name>" -VMName "<your vm name>" -Name "<application health extension type>" 
```
---

### Customize VM watch configurations 
VM watch signals can be customized by configuring the `vmWatchSettings` properties to meet specific requirements. The following table lists the properties for `vmWatchSettings`.

#### vmWatchSettings Properties 
| **Name** | **Type** | **Description** | **Is Required**
|---|---|---|---|
| **enabled** | `bool` | This allows you to enable or disable VM watch | true
| **signalFilters** | `object` | This filters the enabled / disabled signals, either by tag or collector name. | false
| **parameterOverrides** | `object` | This specifies the parameters that can be overwritten for each signal execution. The full list of overwritable parameters can be found in the [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite) page. | false
| **environmentAttributes** | `object` | This specifies any environment attributes that help decide if a test is eligible to execute or not. | false

> [!IMPORTANT]
> For the full list of collectors, associated signals, tags, overwritable parameters, and environment attributes, visit [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite) page 
>
 
#### Switch on/off VM watch 

VM watch can be switched on / off by configuring the `enabled` property, as shown in the code segment. 

```
{
   "vmWatchSettings": {
      "enabled": true
   }
}
```
> [!NOTE]
> | **Name**  | **Description** |
> |---|---|
> | **true**  | This setting enables VM watch |
> | **false** | This setting disables VM watch |
> 

#### Enable/Disable signal execution 

By default, only the core group signals are enabled. However, the `signalFilters` property can be used to control and configure the signals to be executed. This property includes the following subfields.

| **Subfields** | **Description** |
|---|------|
| **enabledTags** | This enables the signals in the optional group specified with tags provided by the user |  
| **disabledTags** | This disables the signals in the core and optional groups specified with tags provided by the user | 
| **enabledOptionalSignals** | This enables signals specified in optional group. Provide collector name(s) as parameter | 
| **disabledSignals** | This disables the signals specified in the core and optional groups. Provide collector name(s) as parameter | 


For instance, to enable signals in the optional group containing `Network` tag and disable signals containing `Disk` tag, specify such tags under the `enabledTags` and `disabledTags` as shown:
```
{
   "vmWatchSettings": {
      "enabled": true,
      "signalFilters": {
         "enabledTags": [
            "Network"
         ],
         "disabledTags": [
            "Disk"
         ]
      }
   }
}
```

Similarly, to enable an optional group signal with name `hardware_health_monitor`, and disable signals with name `process` and `dns`, specify such names under the `enabledOptionalSignals` and `disabledSignals` as shown:  

```
{
   "vmWatchSettings": {
      "enabled": true,
      "signalFilters": {
         "enabledOptionalSignals": [
            "hardware_health_monitor"
         ],
         "disabledSignals": [
            "process",
            "dns"
         ]
      }
   }
}
```

#### Configure signal execution frequency 

The signal execution frequency can be customized by adjusting the `parameterOverrides` property. 
For instance, to set the `outbound_connectivity` test execution frequency to 120 seconds, specify the following configuration:
```
{
   "vmWatchSettings": {
      "enabled": true,
      "parameterOverrides": {
         "OUTBOUND_CONNECTIVITY_INTERVAL": "120s"
      }
   }
}
```
 

#### Override default signal execution parameters 

Signal execution parameters can be overwritten by setting the `parameterOverrides` property. For instance, to set `disk_io` signal mount point to `/mnt`, the following configuration can be specified: 
```
{
   "vmWatchSettings": {
      "enabled": true,
      "parameterOverrides": {
         "DISK_IO_MOUNT_POINTS": "/mnt"
      }
   }
}

``` 

#### Environment attribute enrichments 

In addition to tags, VM watch also checks the eligibility of the signals before execution. The `environmentAttributes` can be specified to help VM watch determine the eligibility of each signal for execution. 
For instance, if outbound traffic has been disabled on a VM, this information can be provided to VM watch. This ensures that any outbound network-related signal execution will be marked as ineligible.
```
{
   "vmWatchSettings": {
      "enabled": true,
      "environmentAttributes": {
         "OutboundConnectivityDisabled": true
      }
   }
}
```

### Next steps

- [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite)
- [Configure Event Hubs for VM watch](/azure/virtual-machines/configure-eventhub-vm-watch)
- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)
- [VM watch overview](/azure/virtual-machines/azure-vm-watch)

