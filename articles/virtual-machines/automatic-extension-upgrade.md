---
title: Automatic Extension Upgrade for VMs and scale sets in Azure
description: Learn how to enable the Automatic Extension Upgrade for your virtual machines and virtual machine scale sets in Azure.
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.topic: how-to
ms.update-cycle: 180-days
ms.date: 11/7/2023
ms.custom: devx-track-azurepowershell, portal
# Customer intent: "As an IT administrator, I want to enable Automatic Extension Upgrade for virtual machines and scale sets, so that I can ensure all extensions are automatically updated to the latest versions without manual intervention for improved security and functionality."
---

# Automatic Extension Upgrade for virtual machines and scale sets in Azure

Automatic Extension Upgrade is available for Azure Virtual Machines and Azure Virtual Machine Scale Sets. When Automatic Extension Upgrade is enabled on a virtual machine (VM) or scale set, Azure automatically monitors for new extension versions, assess the quality & safety of the new version, approves the version for rollout and then automatically upgrades all VMs using the extension. Azure upgrades all the VMs gradually following Safe Deployment Practices (SDP) to prevent bad version from causing outage and ensure high availabilty for applications. This gradual rollout can take 1-2 months to complete based on time required for assessment, the severity of the upgrade and the scale of VMs needed to be upgraded. 

 Automatic Extension Upgrade has the following features:

- Azure VMs, virtual machine scale sets and [Arc VMs](/azure/azure-arc/servers/manage-automatic-vm-extension-upgrade) are supported.
- Upgrades are applied in an availability-first deployment model following (Safe Deployment Practices)[/azure/well-architected/operational-excellence/safe-deployments] (SDP).
- For a virtual machine scale set, by default, no more than 20% of the scale set VMs are upgraded in a single batch. This batch size can be changed by defining [VMSS Rolling Upgrade Policy](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-configure-rolling-upgrades).
- For Azure VMs & Arc VMs, no more than 20% of the VMs are upgraded in a single batch. This batch size cannot be changed.
- All failed upgrades are automatically rolled back to try and bring the VM back to healthy state.
- All upgrades are rebootless. Therefore VM does not need a reboot after the upgrade. 
- All VM sizes are supported.
- Both Windows and Linux extensions are compatible.
- Automatic upgrades are enabled by default starting API version `2025-04-01`.
- Each supported extension is enrolled individually. You can choose which extensions to upgrade automatically.
- All public, sovereign and airgapped cloud regions are supported.
  
## How does Automatic Extension Upgrade work?

The extension upgrade process removes the existing extension on a VM and reinstalls the extension with the new version. 
1. When the extension publisher publishes a new version of the same extension, Azure detects the new version and starts a quality assessment process. This process validates the version for security, quality and safe upgrades without failures. The version is approved for rollout once it passes the assessment.
2. Azure starts with one region or zone (Rollout starts from canary region) and once it is completely upgraded, it moves to the next region or zone.
3. Within a region/zone, Azure groups VMs into smaller batches (based on default or rolling upgrade policies) and upgrades one batch at a time. Azure moves to the next batch once the previous batch is successfully upgraded.
4. Azure monitors for VM and App health (using App health extension) after the upgrade to detect failures. If the VM isn't healthy within five minutes of the upgrade, the upgrade is rollback and previous version of the extension is installed. The upgrade is automatically retried after some time without needing customer intervention. 

### Availability-first Upgrades

The availability-first model for platform-orchestrated upgrades ensures that availability configurations in Azure are respected across multiple availability levels.

For a group of VMs undergoing an upgrade, the Azure platform orchestrates upgrades across regions, within a region, and within a set.

#### Across regions

- An upgrade moves across Azure globally in a phased manner to prevent Azure-wide deployment failures.
- A phase can have one or more regions, and an upgrade moves across phases only if eligible VMs in the previous phase upgrade successfully.
- Geo-paired regions aren't upgraded concurrently and can't be in the same regional phase.
- The success of an upgrade is measured by tracking the health of a VM post upgrade. VM health is tracked through platform health indicators for the VM (VM health & App Health reported by App Health Extension)

#### Within a region

- VMs in different availability zones aren't upgraded concurrently with the same upgrade.
- Single VMs that aren't part of an availability set are batched on a best-effort basis to avoid concurrent upgrades for all VMs in a subscription.  

#### Within a set

- All VMs in a common availability set or scale set aren't upgraded concurrently.  
- VMs in a common availability set are upgraded within update domain boundaries. VMs across multiple update domains aren't upgraded concurrently.  
- VMs in a common virtual machine scale set are grouped in batches and upgraded within update domain boundaries. [Upgrade policies](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-policy.md) defined on the scale set are honored during the upgrade. Each group is upgraded by using a rolling upgrade strategy.

### Upgrade process for virtual machine scale sets

- Before the upgrade process starts, the orchestrator ensures that no more than 20% of VMs in the entire scale set are unhealthy (for any reason).
- The upgrade orchestrator identifies the batch of VM instances to upgrade. An upgrade batch can have a maximum of 20% of the total VM count, subject to a minimum batch size of one VM. The orchestrator considers the definition of the upgrade policy and availability zones while the batch is identified.  
- After the upgrade, the VM health is always monitored before moving to the next batch. For scale sets with configured application health probes or the Application Health extension, application health is also monitored. The upgrade waits up to five minutes (or the defined health probe configuration) for the VM to become healthy before upgrading the next batch. If a VM doesn't recover its health after an upgrade, then by default, the previous extension version on the VM is reinstalled.
- The upgrade orchestrator also tracks the percentage of VMs that become unhealthy after an upgrade. The upgrade stops if more than 20% of upgraded instances become unhealthy during the upgrade process.

This process continues until all instances in the scale set are upgraded.

The scale set upgrade orchestrator checks for the overall scale set health before upgrading every batch. During a batch upgrade, other concurrent planned or unplanned maintenance activities could affect the health of your scale set VMs. In such cases, if more than 20% of the scale set's instances become unhealthy, the scale set upgrade stops at the end of the current batch.

## Supported extensions
To check if your extensions are supported for automatic upgrade, view Automatic Upgrade status on Azure Portal - Extension blade.

:::image type="content" source="media/auto-extension.png" alt-text="Screenshot that shows if extension is supported for automatic upgrade in the Azure portal.":::

Following are popular extensions supported for automatic upgrades (and more are added periodically):

| Publisher                                         | Type                          |
|---------------------------------------------------|-------------------------------|
| Microsoft.Azure.Automation.HybridWorker           | [HybridWorkerForLinux](/azure/automation/extension-based-hybrid-runbook-worker-install)       |
| Microsoft.Azure.Automation.HybridWorker           | [HybridWorkerForWindows](/azure/automation/extension-based-hybrid-runbook-worker-install)        |
| Microsoft.Azure.AzureDefenderForSQL               | AdvancedThreatProtection.Windows |
| Microsoft.Azure.AzureDefenderForSQL               | VulnerabilityAssessment.Windows |
| Microsoft.Azure.AzureDefenderForServers           | MDE.Linux                     |
| Microsoft.Azure.AzureDefenderForServers           | MDE.Windows                   |
| Microsoft.Azure.ChangeTrackingAndInventory        | ChangeTracking-Linux          |
| Microsoft.Azure.ChangeTrackingAndInventory        | ChangeTracking-Windows        |
| Microsoft.Azure.Diagnostics                       | [LinuxDiagnostic](/azure/azure-monitor/agents/diagnostics-extension-overview)               |
| Microsoft.Azure.Extensions.Edp                    | LinuxHibernateTestExtension   |
| Microsoft.Azure.Extensions.Edp                    | WindowsHibernateTestExtension |
| Microsoft.Azure.FleetDiagnostics                  | FleetDiagnosticsForWindows    |
| Microsoft.Azure.Geneva                            | GenevaMonitoring              |
| Microsoft.Azure.KeyVault                          | [KeyVaultForLinux](./extensions/key-vault-linux.md)              |
| Microsoft.Azure.KeyVault                          | [KeyVaultForWindows](./extensions/key-vault-windows.md)            |
| Microsoft.Azure.Labservices                       | Agent.Linux                   |
| Microsoft.Azure.Labservices                       | Agent.Windows                 |
| Microsoft.Azure.Monitor                           | [AzureMonitorLinuxAgent](/azure/azure-monitor/agents/azure-monitor-agent-overview)        |
| Microsoft.Azure.Monitor                           | [AzureMonitorWindowsAgent](/azure/azure-monitor/agents/azure-monitor-agent-overview)      |
| Microsoft.Azure.Monitoring.DependencyAgent.EDP    | [DependencyAgentLinux](./extensions/agent-dependency-linux.md)          |
| Microsoft.Azure.Monitoring.DependencyAgent.EDP    | [DependencyAgentWindows](./extensions/agent-dependency-windows.md)        |
| Microsoft.Azure.Monitoring.DependencyAgent        | [DependencyAgentLinux](./extensions/agent-dependency-linux.md)          |
| Microsoft.Azure.Monitoring.DependencyAgent        | [DependencyAgentWindows](./extensions/agent-dependency-windows.md)        |
| Microsoft.Azure.NetworkWatcher                    | NetworkWatcherAgentLinux      |
| Microsoft.Azure.NetworkWatcher                    | NetworkWatcherAgentWindows    |
| Microsoft.Azure.Networking.DNS                    | DNSClientCache                |
| Microsoft.Azure.SCOMMI                            | GatewayServer                 |
| Microsoft.Azure.SCOMMI                            | WindowsAgent                  |
| Microsoft.Azure.Security.AntimalwareSignature     | AntimalwareConfiguration      |
| Microsoft.Azure.Security.Dsms                     | DSMSForWindows                |
| Microsoft.Azure.Security.LinuxAttestation         | [GuestAttestation](../virtual-machines/boot-integrity-monitoring-overview.md)              |
| Microsoft.Azure.Security.Monitoring               | AzureSecurityLinuxAgent       |
| Microsoft.Azure.Security.Monitoring               | AzureSecurityWindowsAgent     |
| Microsoft.Azure.Security.WindowsAttestation       | [GuestAttestation](../virtual-machines/boot-integrity-monitoring-overview.md)              |
| Microsoft.Azure.Security.WindowsCodeIntegrity     | CodeIntegrityAgent            |
| Microsoft.Azure.ServiceFabric                     | [ServiceFabricLinuxNode](../service-fabric/service-fabric-tutorial-create-vnet-and-linux-cluster.md#service-fabric-extension)        |
| Microsoft.Azure.Watson                            | WatsonLinuxAgent              |
| Microsoft.Azure.Workloads                         | MonitoringExtensionLinux      |
| Microsoft.Azure.Workloads                         | MonitoringExtensionWindows    |
| Microsoft.CPlat.Core                              | LinuxHibernateExtension       |
| Microsoft.CPlat.Core                              | WindowsHibernateExtension     |
| Microsoft.CPlat.ProxyAgent                        | ProxyAgentLinux               |
| Microsoft.CPlat.ProxyAgent                        | ProxyAgentWindows             |
| Microsoft.EnterpriseCloud.Monitoring              | [MicrosoftMonitoringAgent](./extensions/oms-windows.md)      |
| Microsoft.EnterpriseCloud.Monitoring              | [OmsAgentForLinux](./extensions/oms-linux.md)              |
| Microsoft.GuestConfiguration                      | [ConfigurationForLinux](./extensions/guest-configuration.md)         |
| Microsoft.GuestConfiguration                      | [ConfigurationForWindows](./extensions/guest-configuration.md)       |
| Microsoft.ManagedServices                         | [ApplicationHealthLinux](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md)        |
| Microsoft.ManagedServices                         | [ApplicationHealthWindows](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md)      |
| Microsoft.OSTCExtensions                          | DSCForLinux                   |
| Microsoft.Sentinel.AzureMonitorAgentExtensions    | MicrosoftDnsAgent             |
| Microsoft.SqlServer.Management                    | SqlIaaSAgent                  |
| Microsoft.SqlServer.Management                    | SqlIaaSAgentLinux             |

---

## Enable Automatic Extension Upgrade

To enable Automatic Extension Upgrade for an extension, you must ensure that the property `enableAutomaticUpgrade` is set to `true` and added to every extension definition individually.

### Use the Azure portal

In the Azure portal, use the **Extension** pane to enable automatic upgrade of extensions on existing VMs and virtual machine scale sets.

1. Go to the [Virtual Machines](https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.Compute%2FVirtualMachines) or [Virtual Machines Scale Sets](https://ms.portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.Compute%2FvirtualMachineScaleSets) pane, and select the resource name.
1. Under **Settings**, go to the **Extensions + applications** pane, which shows all extensions installed on the resource. The **Automatic upgrade status** column shows you if the automatic upgrade of the extension is enabled, disabled, or not supported.
1. Select the extension name to open the **Extensions** details pane.

   :::image type="content" source="media/auto-extension.png" alt-text="Screenshot that shows the Extensions pane in the Azure portal." lightbox="media/auto-extension.png":::

1. Select **Enable automatic upgrade** to enable automatic upgrade of the extension. Use this button to disable an automatic upgrade, if necessary.

   :::image type="content" source="media/auto-extension-upgrade.png" alt-text="Screenshot that shows Enable automatic upgrade in the Azure portal.":::

### For virtual machines

#### [REST API](#tab/RestAPI1)

To enable Automatic Extension Upgrade for an extension (in this example, the Dependency Agent extension) on an Azure VM, use the following call:

```
PUT on `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/extensions/<extensionName>?api-version=2019-12-01`
```

```json
{    
    "name": "extensionName",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "location": "<location>",
    "properties": {
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true, 
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5"
        }
}
```

#### [PowerShell](#tab/powershell1)

Use the [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) cmdlet:

```azurepowershell-interactive
Set-AzVMExtension -ExtensionName "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ExtensionType "DependencyAgentWindows" `
    -TypeHandlerVersion 9.5 `
    -Location WestUS `
    -EnableAutomaticUpgrade $true
```

#### [CLI](#tab/cli1)

Use the [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set) cmdlet:

```azurecli-interactive
az vm extension set \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 \
    --enable-auto-upgrade true
```

#### [Template](#tab/template1)

The following example describes how to set automatic extension upgrades for an extension (Dependency Agent extension in this example) on a VM by using Azure Resource Manager.

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "location": "[resourceGroup().location]",
    "name": "<extensionName>",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true,
        "settings": {
            "enableAMA": "true"
        }
    }
}
```
----

### For virtual machine scale sets

#### [REST API](#tab/RestAPI2)

```
PUT on `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachineScaleSets/<vmssName>?api-version=2019-12-01`
```

```json
{
   "location": "<location>",
   "properties": {
   	    "virtualMachineProfile": {
            "extensionProfile": {
       	        "extensions": [
            	{
                "name": "<extensionName>",
            	  "properties": {
             		    "autoUpgradeMinorVersion": true,
             		    "enableAutomaticUpgrade": true,
              	    "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
              	    "type": "DependencyAgentWindows",
              	    "typeHandlerVersion": "9.5"
            		}
          	    }
        	    ]
    	    }
    	}
    }
}
```

#### [PowerShell](#tab/powershell2)

Use the [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) cmdlet to add the extension to the scale set model:

```azurepowershell-interactive
Add-AzVmssExtension -VirtualMachineScaleSet $vmss
    -Name "Microsoft.Azure.Monitoring.DependencyAgent" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -Type "DependencyAgentWindows" `
    -TypeHandlerVersion 9.5 `
    -EnableAutomaticUpgrade $true
```

Update the scale set by using [Update-AzVmss](/powershell/module/az.compute/update-azvmss) after you add the extension.

#### [CLI](#tab/cli2)

Use the [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set) cmdlet to add the extension to the scale set model:

```azurecli-interactive
az vmss extension set \
    --resource-group myResourceGroup \
    --vmss-name myVMSS \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 \
    --enable-auto-upgrade true
```

#### [Template](#tab/template2)

Use the following example to set Automatic Extension Upgrade on the extension within the scale set model:

```json
{
   "type": "Microsoft.Compute/virtualMachineScaleSets",
   "apiVersion": "2023-09-01",
   "name": "[variables('vmScaleSetName')]",
   "location": "[resourceGroup().location]",
   "properties": {
   	    "virtualMachineProfile": {
            "extensionProfile": {
       	        "extensions": [{
                     "name": "<extensionName>",
                     "properties": {
                          "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
                          "type": "DependencyAgentWindows",
                          "typeHandlerVersion": "9.5",
                          "autoUpgradeMinorVersion": true,
                          "enableAutomaticUpgrade": true,
                     }
                }]
    	    }
    	}
    }
}
```
----

> [!NOTE]
> These operations set the `enableAutomaticUpgrade` property to `true` on the virtual machine scale set resource but not on the underlying VMs.

If the virtual machine scale set defines [automatic or rolling upgrade mode in the upgradeProfile](../virtual-machine-scale-sets/virtual-machine-scale-sets-change-upgrade-policy.md), the virtual machine scale set automatically propagates the change to each underlying VM.

If the virtual machine scale set defines manual mode in the `upgradeProfile`, you also need to [manually update each instance](../virtual-machine-scale-sets/virtual-machine-scale-sets-perform-manual-upgrades.md) and propagate the change to each underlying VM.

--- 

## Extension upgrades with multiple extensions

A VM or virtual machine scale set can have multiple extensions with Automatic Extension Upgrade enabled. The same VM or scale set can also have other extensions without Automatic Extension Upgrade enabled.  

If multiple extension upgrades are available for a VM, the upgrades might be batched together, but each extension upgrade is applied individually on a VM. A failure on one extension doesn't affect the other extensions that might be upgrading. For example, if two extensions are scheduled for an upgrade, and the first extension upgrade fails, the second extension is still upgraded.

You can also apply Automatic Extension Upgrade when a VM or virtual machine scale set has multiple extensions configured with [extension sequencing](../virtual-machine-scale-sets/virtual-machine-scale-sets-extension-sequencing.md). Extension sequencing is for the first-time deployment of the VM. Any future extension upgrades on an extension are applied independently.

## Difference between EnableAutomaticUpgrade and AutoUpgradeMinorVersion

-  `AutoUpgradeMinorVersion`:

   - This property is used during VM creation and while you upgrade the VM with a new configuration.  
   - When set to `true`, it ensures that the latest minor version of the extension is automatically installed on the VM.
   - It overrides the `TypeHandlerVersion` with the latest stable minor version available.
   - When you upgrade the VM configuration, if a new minor version is available, it's considered a configuration change. The extension is reinstalled with the latest minor version.
   - In this way, newly created VMs keep up to date with the latest stable minor extension version.
   - If you want to manually set the extension to a specific version, set this property to `false`.
     
-  `EnableAutomaticUpgrade`:
   - This property affects existing VMs.
   - It doesn't affect the version installed during VM creation.
   - After VM creation, if the VM isn't running the latest minor version of the extension, enable this property to trigger an automatic upgrade.
   - Upgrades don't cause VM reboot and are rolled out in a safe rolling manner. Failed upgrades are rolled back immediately to provide high service availability and reliability.
   - Existing VMs stay secure and up to date by automatically updating them to the latest minor version.

We recommend that you enable both properties to help keep all VMs secure and up to date.

Upgrades to major extension versions are never performed automatically by either properties because major versions can include breaking changes. You must manually set `TypeHandlerVersion` to a major version and manually upgrade each existing VM to the latest major version.

## Next step
> [!div class="nextstepaction"]
> [Learn about the Application Health extension](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md)
