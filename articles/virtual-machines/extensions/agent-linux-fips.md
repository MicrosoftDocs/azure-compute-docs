---
title: FIPS 140-3 Support for Azure Linux VM Extensions and Guest Agent
description: Learn how to opt in to FIPS 140-3 support for Azure Linux VM extensions and the guest agent.
ms.topic: how-to
ms.service: azure-virtual-machines
ms.subservice: extensions
ms.author: gabsta
author: GabstaMSFT
ms.custom: linux-related-content; references_regions
ms.collection: linux
ms.date: 09/25/2025
---
# FIPS 140-3 support for Azure Linux VM extensions and guest agent

> [!NOTE]
> This feature is currently in public preview. Production workloads are supported.

Linux virtual machine (VM) extensions currently comply with FIPS 140-2, but updates to the platform were required to add support for FIPS 140-3. These changes are currently being enabled across the commercial cloud and Azure Government clouds. Linux VM extensions that use protected settings are also being updated to be able to use a FIPS 140-3-compliant encryption algorithm. This article helps enable support for FIPS 140-3 on Linux VMs where compliance with FIPS 140-3 is enforced. This change isn't needed on Windows images because of the way that FIPS compliance is implemented.

For more information, see [What are the Federal Information Processing Standards (FIPS)?](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips).

[!INCLUDE [VM assist troubleshooting tools](../includes/vmassist-include.md)]

## Confirmed supported extensions

| Extension | Supported clouds<br> Commercial,<br> Government,<br> Air-Gap |
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

You must meet the following four requirements to use a FIPS 140-3-compliant VM in Azure:

- The VM must be in a region where FIPS 140-3 platform changes are rolled out.
- Your Azure subscription must be opted in to FIPS 140-3 enablement.
- Each VM must be enrolled in FIPS 140-3 enablement in Azure Resource Manager.
- Inside the guest operating system (OS), the OS must be configured for FIPS 140 mode. The OS must run a version of the Azure guest agent (waagent), which is also compliant with FIPS 140-3.

Afterward, validate to ensure the functionality of the VM extensions.

---

## Implement prerequisites

### 1. Enabled regions

To view the latest supported regions, use the Linux VM Guest [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1) release page.

| Cloud | Region |
|:-----|:-----|
| Commercial | Australia Central, Australia Central 2, Australia East, Australia Southeast, Brazil South, Brazil Southeast, Canada Central, Canada East, Central India, Denmark East, East Asia, France Central, France South, Germany North, Germany West Central, India South, Japan East, Japan West, Jio India Central, Jio India West, Korea Central, Korea South, Norway East, Norway West, Qatar Central, South Africa North, South Africa West, Southeast US, Sweden Central, Sweden South, Switzerland North, Switzerland West, Taiwan North, UAE Central, UAE North, UK West, West Central US, West India, West US 2, West US 3 |
| USGov | All regions |
| Air-Gap | All regions |

### 2. Subscription enablement/opt-in

Because not all extensions are onboarded by using FIPS 140-3 encryption yet, we require the subscription to opt in to the feature `_Microsoft.Compute/OptInToFips1403Compliance_`.

#### Azure CLI

```
az feature register --namespace Microsoft.Compute --name OptInToFips1403Compliance
```

Verify with the following command:
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

### 3. Per-VM opt-in

There are different methods available for opting in to each VM. You can make the changes at deployment for a new VM. You can also alter an existing VM to add the FIPS 140-3 enablement on the Azure platform.

> [!WARNING]
> We don't recommend using the following opt-in methods on Red Hat Enterprise Linux (RHEL) 9.5 and 9.6 by using version 2.7.0.6 of `WALinuxAgent` on production systems. An issue can surface after rebooting, after the FIPS enablement and subsequent reboot. On these VMs, the `waagent.service` enters an internal loop and never comes to a `Ready` state. Because of this error, no extensions can function. For testing, you can try the RHEL 9 workaround.

#### Deploy a new VM

To deploy a new VM with FIPS 140-3 enablement turned on immediately, use an Azure Resource Manager template (ARM template) or the Azure CLI. Add the `enableFips1403Encryption` property to the `additionalCapabilities` section of the `virtualMachines` object definition.

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

#### Modify an existing VM

##### az cli commands

> [!NOTE]
> For the Government cloud, use `https://management.usgovcloudapi.net` instead of `https://management.azure.com`.

While updates to the SDK/CLI are still in progress, you can continue to use `az cli` to add the property.

```
az rest \
--method put \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01' \
--body '{"location": "<LOCATION>", "properties": {"additionalCapabilities": {"enableFips1403Encryption": true}}}'
```

Running the `put` command outputs the resulting JSON for the modified VM. For later verification, you can run this `get` command against the VM object, which outputs the full JSON again.

``` 
az rest \
--method get \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01'
```

The command output should include:

```json
{
 "enableFips1403Encryption": true
}
```

To more easily find the property in the output, you can add `jq` to parse out the specific section needed. This block is the new command:

```
az rest \
--method get \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01' \
| jq .properties.additionalCapabilities
```

For comparison, one possible outcome when you try to enable FIPS 140-3 on a VM when the VM isn't in an enabled region, the `put` command can output the following code, which indicates that the action isn't possible in the region.

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

Important changes must be made to the Linux OS environment to enable and support FIPS 140-3 compliance.

#### Configure the OS for FIPS enablement

The following distributions support FIPS 140-3 and provide instructions for enabling:

- Ubuntu 22.04 LTS and newer:
  - Use an [Ubuntu pro client or pro image](https://documentation.ubuntu.com/pro-client/en/docs/howtoguides/enable_fips/).
- RHEL 9:
  - Steps to [enable FIPS on RHEL](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening).

Older versions of these operating systems operate at the FIPS 140-2 level and don't require any of these special considerations.

#### Linux guest agent

Minimum [Goal State Agent](https://github.com/Azure/WALinuxAgent/wiki/FAQ#what-does-goal-state-agent-mean-in-waagent---version-output) version: [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1). To be sure that the goal state is updating, the `AutoUpdate.Enabled` flag should be `y` or commented out entirely so that the default behavior is used.

/etc/waagent.conf:
```
AutoUpdate.Enabled=y
```

##### RHEL 9 workaround

This workaround is intended for testing purposes only and doesn't support all VM deployment scenarios. After you enable FIPS on a running VM, run the following commands to proceed:

```
systemctl stop waagent

# apply the patch
sed -i -E '/(.+)(self._initialize_telemetry\(\))/s//\1# \2/' /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py

```

To verify that the previous change was applied successfully, use the following command:

```
grep self\._initialize_telemetry /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py

```

The output should be exactly this text:

```
        # self._initialize_telemetry()
```

After verification, restart the agent:

```
systemctl start waagent
```

---

## Validation

To validate proper functionality of the VM extensions:

- Check that the agent status is `Ready`.
- Test an extension by using the protected settings of the VM extensions.
  - By using the `Reset Password` function of the Azure portal or `az cli`, reset a password or create a new temporary user.
  - Run a custom script.

If these tests fail, force the Azure platform to generate a new personal information exchange (PFX) package.

### Reset password

Use either the Azure portal or an `az cli` command, such as this example, to set a user's password or create a temporary user. Check the execution state for success or failure.

```bash
az vm user update \
  --resource-group <YourResourceGroup> \
  --name <YourVMName> \
  --username <NewUsername> \
  --password <NewPassword>

```

### Run a custom script

Use the [Custom Script extension](/azure/virtual-machines/extensions/custom-script-linux) documentation to send a basic script, such as `cat /etc/os-release`, to test extension functionality.

### Fix a validation failure

If the validations fail to execute, force the Azure platform to generate a new PFX package. There are two methods to force this regeneration:

- Reallocate the VM.
- Apply an Azure Key Vault certificate.

#### Deallocate/Reallocate the VM

You can use the Azure CLI, the Azure portal, or any other method to deallocate the VM. Wait for the deallocation to occur, and then start the VM.

#### Add a Key Vault certificate

Create the Key Vault certificate, add it to the modified ARM template, and deploy.

For more information, see [Get started with Key Vault certificates](/azure/key-vault/certificates/certificate-scenarios).

The following example shows the `properties` section of the VM model:

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
