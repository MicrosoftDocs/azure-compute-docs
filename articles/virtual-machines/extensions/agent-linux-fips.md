---
title: FIPS 140-3 Support for Azure Linux VM Extensions and Guest Agent
description: Learn how to opt in to FIPS 140-3 support for Azure Linux VM Extensions and Guest Agent.
ms.topic: how-to
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.custom: linux-related-content; references_regions
ms.collection: linux
ms.date: 09/25/2025
---
# FIPS 140-3 support for Azure Linux VM Extensions and Guest Agent

> [!NOTE]
> This feature is currently in **Public Preview**, production workloads are supported.

Linux Virtual Machine (VM) Extensions currently comply with FIPS 140-2 but updates to the platform were required to add support for FIPS 140-3. These changes are currently being enabled across the Commercial Cloud and Azure Government Clouds. Linux VM Extensions that use protected settings are also being updated to be able to use a FIPS 140-3 compliant encryption algorithm. This document helps enable support for FIPS 140-3 on Linux VMs where compliance with FIPS 140-3 is enforced. This change isn't needed on Windows images due to the way FIPS compliance is implemented.

- [What is the Federal Information Processing Standards (FIPS)](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips)

## Confirmed Supported Extensions

| Extension | Supported Clouds<br> _Commercial,<br> Government,<br> Air-Gap_ |
|:----------|:---------------------------------------------------:|
| MICROSOFT.AKS.COMPUTE.AKS.LINUX.AKSNODE | Commercial,<br>Government |
| MICROSOFT.AKS.COMPUTE.AKS.LINUX.BILLING | Commercial,<br>Government |
| MICROSOFT.AZURE.EXTENSIONS.CUSTOMSCRIPT | Commercial |
| MICROSOFT.AZURE.KEYVAULT.KEYVAULTFORLINUX | Commercial |
| MICROSOFT.AZURE.MONITORING.DEPENDENCYAGENT.DEPENDENCYAGENTLINUX | Commercial |
| MICROSOFT.AZURE.NETWORKWATCHER.NETWORKWATCHERAGENTLINUX | Commercial,<br>Government |
| MICROSOFT.AZURE.RECOVERYSERVICES.VMSNAPSHOT | Commercial |
| MICROSOFT.AZURE.SECURITY.MONITORING.AZURESECURITYLINUXAGENT | Commercial |
| MICROSOFT.CPLAT.CORE.LINUXPATCHEXTENSION | Commercial,<br>Government |
| MICROSOFT.CPLAT.CORE.RUNCOMMANDLINUX | Commercial |
| MICROSOFT.CPLAT.CORE.VMAPPLICATIONMANAGERLINUX | Commercial |
| MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTLINUX | Commercial |
| MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTLINUXARM64 | Commercial |
| MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTWINDOWS | Commercial |
| MICROSOFT.GUESTCONFIGURATION.CONFIGURATIONFORLINUX | Commercial |
| MICROSOFT.MANAGEDSERVICES.APPLICATIONHEALTHLINUX | Commercial,<br>Government |
| MICROSOFT.OSTCEXTENSIONS.VMACCESSFORLINUX | Commercial |

## Prerequisites

There are four requirements to being able to use a FIPS 140-3 compliant VM in Azure:

- The Virtual Machine must be in a region where FIPS 140-3 platform changes are rolled out.
- Your Azure Subscription must be opted-in to FIPS 140-3 enablement.
- Each VM must be enrolled in FIPS 140-3 enablement in the Azure Resource Manager.
- Inside of the guest OS, the operating system must be configured for FIPS 140 mode, and running a version of the Azure guest agent (waagent) which is also FIPS 140-3 compliant.

Once these steps are followed, validation should be done to ensure functionality of VM extensions.

---

## Implementing prerequisites

### 1. Enabled Regions
To view the latest supported regions, use the Linux VM Guest [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1) release page.

| Cloud | Region |
|:-----|:-----|
| Commercial | Central US EUAP, East US 2 EUAP, Australia East, Brazil South, Canada Central, East Asia, France South, Japan East, Korea South, Southeast US, South India, Sweden Central, UK South, UK West, West Central US, West India |
| USGov | All Regions |
| Air-Gap | All Regions |

### 2. Subscription Enablement / Opt-In

Because not all extensions are onboarded onto using FIPS 140-3 encryption yet, we’re requiring the subscription to opt into this feature.
- The Subscription needs to enable the feature: “_Microsoft.Compute/OptInToFips1403Compliance_”

**Azure CLI**
```
az feature register --namespace Microsoft.Compute --name OptInToFips1403Compliance
```

Verify with the following command
```
az feature list | jq '.[] | select(.name=="Microsoft.Compute/OptInToFips1403Compliance")'
```

```json
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Features/providers/Microsoft.Compute/features/OptInToFips1403Compliance",
  "name": "Microsoft.Compute/OptInToFips1403Compliance",
  "properties": {
    "state": "Registered"
  },
  "type": "Microsoft.Features/providers/features"
}
```

---

### 3. Per-VM Opt-In

There are different methods available for opting-in each VM. The changes can be made at deployment for a new VM, or an existing VM can be altered to add the FIPS 140-3 enablement on the Azure platform.

> [!WARNING]
> We do not recommend using the below Opt-In methods on RedHat 9.5 and 9.6 using version 2.7.0.6 of WALinuxAgent on production systems. This is due to an issue that will surface after rebooting, after the FIPS enablement and subsequent reboot. In these VMs the `waagent.service` will enter an internal loop and never come to a "Ready" state, and because of this error, no extensions are able to function. For testing you can try the below "RedHat 9 Workaround".

#### Deploying a new VM

In order to deploy a new VM with FIPS 140-3 enablement turned on immediately, use an ARM Template or CLI and add the `enableFips1403Encryption` property to the `additionalCapabilities` section of the `virtualMachines` object definition

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2022-03-01",
  "name": "[parameters('vmName')]",
  "location": "[parameters('location')]",
  "properties": {
	  "additionalCapabilities": {
      "enableFips1403Encryption": "true"
    }
  }
}
```

#### Modifying an existing VM

##### az cli commands

> [!NOTE]
> For the Government cloud, use "https://management.usgovcloudapi.net" instead of "https://management.azure.com"

While updates to SDK/CLI are still in progress, you can still use AZ CLI to add the property. 

```
az rest \
--method put \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01' \
--body '{"location": "<LOCATION>", "properties": {"additionalCapabilities": {"enableFips1403Encryption": true}}}'
```

Running the `put` command outputs the resulting json for the modified VM. For later verification, this `get` command can be run against the VM object, which outputs the full JSON again

``` 
az rest \
--method get \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01'
```

The command output should include

```json
{
 "enableFips1403Encryption": true
}
```

In order to more easily find the property in the output, you can add `jq` to parse out the specific section needed. This block is the new command

```
az rest \
--method get \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01' \
| jq .properties.additionalCapabilities
```

For comparison, one possible outcome when trying to enable FIPS 140-3 on a VM when the VM isn't in an enabled region, the `put` command can output the following, indicating the action isn't possible in the region

```json
({
  "error": {
    "code": "BadRequest",
    "message": "Creation of VMs using a Fips 140-3 compliant encryption for extension settings isn't supported in this region."
  }
})
```

<!-- 
---

##### Option 2 - modifying ARM with template
Leaving the marker here, but deleting the content pending research -->

---

### 4. In-guest considerations

There are important changes that need to be done to the Linux operating system environment to enable and support FIPS 140-3 compliance.

#### Configuring the operating system for FIPS enablement

The following distributions support FIPS 140-3 and provide instructions for enabling

- Ubuntu 22.04 LTS and newer
  - Use an Ubuntu pro client or pro image: https://documentation.ubuntu.com/pro-client/en/docs/howtoguides/enable_fips/
- Red Hat Enterprise Linux 9
  - Steps to enable FIPS on Redhat: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening


Older versions of these operating systems operate at the FIPS 140-2 level and don't require any of these special considerations.


#### Linux Guest Agent

Minimum [Goal State Agent](https://github.com/Azure/WALinuxAgent/wiki/FAQ#what-does-goal-state-agent-mean-in-waagent---version-output) Version: [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1). To be sure the goal state is updating, the `AutoUpdate.Enabled` flag should be `y` or commented out entirely so that the default behavior is used

/etc/waagent.conf:
```
AutoUpdate.Enabled=y
```

##### RedHat 9 Workaround

> [!NOTE]
> This workaround is intended for testing purposes only and does not support all VM deployment scenarios. After enabling FIPS on a running VM, execute the following commands to proceed.

```
systemctl stop waagent

# apply the patch
sed -i -E '/(.+)(self._initialize_telemetry\(\))/s//\1# \2/' /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py

```

Use the following command to verify that the previous change was applied successfully

```
grep self\._initialize_telemetry /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py

```

The output should be exactly this text:

```
        # self._initialize_telemetry()
```

Once verified, restart the agent

```
systemctl start waagent
```

---

## Validation

To validate proper functionality of the VM Extensions
- Check that the agent status is 'Ready'
- Test an extension utilizing the "protected settings" of the VM extensions
  - Using the "Reset Password" function of the Azure portal or az cli, reset a password or create a new temporary user.
  - Run a custom script

If these tests fail, it is necessary to force the Azure platform to generate a new PFX.


### Reset Password

Using either the Azure portal, or an az cli command such as this example, to set a user's password or create a temporary user. Check the execution state for success or failure.

```bash
az vm user update \
  --resource-group <YourResourceGroup> \
  --name <YourVMName> \
  --username <NewUsername> \
  --password <NewPassword>

```

### Run a custom script

Use the [Custom Script Extension](/azure/virtual-machines/extensions/custom-script-linux) documentation to send a basic script such as `cat /etc/os-release` to test extension functionality

### Fixing a validation failure

If the validations fail to execute, it is required to force the Azure platform to generate a new PFX package. There are two methods to force this regeneration to happen. Reallocating the VM or applying a Keyvault Certificate.

#### Deallocate/Reallocate the VM

Using any method such as Azure CLI, the Azure portal, or any other method to deallocate the VM, wait for the deallocation to occur, then start the VM.

#### Add a Keyvault Certificate

Create the keyvault/certificate then add it to the modified ARM template and deploy.
- [Get started with Key Vault certificates | Microsoft Learn](/azure/key-vault/certificates/certificate-scenarios)

Example: 'properties' section of the VM model:

```json
      "secrets": [
        {
          "sourceVault": {
            "id": "/subscriptions/<subId>/resourceGroups/<resource group>/providers/Microsoft.KeyVault/vaults/<keyvault name>"
          },
          "vaultCertificates": [
            {
              "certificateUrl": "https://<keyvault name>.vault.azure.net/secrets/rsa/5cc588f8f3404268b2ca4d05e544e7fb"
            }
          ]
        }
      ],
```
