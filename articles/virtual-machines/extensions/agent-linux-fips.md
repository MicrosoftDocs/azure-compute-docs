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
ms.date: 08/27/2025
---

# FIPS 140-3 support for Azure Linux VM Extensions and Guest Agent
Linux VM Extensions currently comply with FIPS 140-2 but updates to the platform were required to add support for FIPS 140-3.  These changes are currently being enabled across the Commercial Cloud and Azure Government Clouds. Linux VM Extensions that use protected settings are also being updated to be able to use a FIPS 140-3 compliant encryption algorithm. This document helps enable support for FIPS 140-3 on Linux VMs where compliance with FIPS 140-3 is enforced.  This change is not needed on Windows images due to the way FIPS compliance is implemented.

## Prerequisites
### Linux Guest Agent
- Minimum [Goal State Agent](https://github.com/Azure/WALinuxAgent/wiki/FAQ#what-does-goal-state-agent-mean-in-waagent---version-output) Version: [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1)

### Supported Images
- **Ubuntu**
  - Use an Ubuntu pro client or pro image: https://documentation.ubuntu.com/pro-client/en/docs/howtoguides/enable_fips/
- **Red Hat Enterprise Linux**
  - Steps to enable FIPS on Redhat: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening

### Enabled Regions
To view the latest supported regions, use the Linux VM Guest [v2.14.0.1](https://github.com/Azure/WALinuxAgent/releases/tag/v2.14.0.1) release page.

| Cloud | Region(s) |
|:-----|:-----|
| Commercial | Central US EUAP, East US 2 EUAP, West Central US, East Asia, UK South, Australia East, South India, North Europe |
| USGov | All Regions |
| Air-Gap | US Sec |


### Subscription Enablement
Because not all extensions are onboarded onto using FIPS 140-3 encryption yet, we’re requiring the subscription to opt into this feature.
- The Subscription needs to enable the feature: “_Microsoft.Compute/OptInToFips1403Compliance_”

**Azure CLI**
```
az feature register --namespace Microsoft.Compute --name OptInToFips1403Compliance
```

### Confirmed Supported Extensions
**Linux VM Extensions with Confirmed Support**
- MICROSOFT.AKS.COMPUTE.AKS.LINUX.AKSNODE
- MICROSOFT.AKS.COMPUTE.AKS.LINUX.BILLING
- MICROSOFT.AZURE.EXTENSIONS.CUSTOMSCRIPT
- MICROSOFT.AZURE.KEYVAULT.KEYVAULTFORLINUX
- MICROSOFT.AZURE.MONITORING.DEPENDENCYAGENT.DEPENDENCYAGENTLINUX
- MICROSOFT.AZURE.NETWORKWATCHER.NETWORKWATCHERAGENTLINUX
- MICROSOFT.AZURE.RECOVERYSERVICES.VMSNAPSHOT
- MICROSOFT.AZURE.SECURITY.MONITORING.AZURESECURITYLINUXAGENT
- MICROSOFT.CPLAT.CORE.LINUXPATCHEXTENSION
- MICROSOFT.CPLAT.CORE.RUNCOMMANDLINUX
- MICROSOFT.CPLAT.CORE.VMAPPLICATIONMANAGERLINUX
- MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTLINUX
- MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTLINUXARM64
- MICROSOFT.CPLAT.PROXYAGENT.PROXYAGENTWINDOWS
- MICROSOFT.GUESTCONFIGURATION.CONFIGURATIONFORLINUX
- MICROSOFT.MANAGEDSERVICES.APPLICATIONHEALTHLINUX
- MICROSOFT.OSTCEXTENSIONS.VMACCESSFORLINUX

## Opt-In Guide
### New VMs
**Step 1:**
- Deploy VM from ARM Template or CLI ([Example ARM Template](#example-arm-template))

**Step 2: Enable FIPS on Linux**
- Information on enabling FIPS with Ubuntu: https://documentation.ubuntu.com/pro-client/en/docs/howtoguides/enable_fips/
- Information for enabling FIPS on Redhat: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/switching-rhel-to-fips-mode_security-hardening

### Existing VMs
**Step 1: ARM template flag to enable**

*NOTE:* For the Goverment cloud use "https://management.usgovcloudapi.net" instead of "https://management.azure.com"

**Option 1: Az CLI**

Updates to SDK/CLI are still in progress, you can still use AZ CLI to add the property. 

```
az rest \
--method put \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01' \
--body '{"location": "<LOCATION>", "properties": {"additionalCapabilities": {"enableFips1403Encryption": true}}}'
```

**Verify with:**
``` 
az rest \
--method get \
--url 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>/?api-version=2024-11-01'
```

**Expected Result:**

The command output should include
```json
{
 "enableFips1403Encryption": true
}
```

You can also query this value using the jq command
```
$ az rest --method get --url 'https://...' \
  | jq .properties.additionalCapabilities
{
"enableFips1403Encryption": true
}
```

**Option 2: ARM Template**

1.	Export the ARM Template for an existing VM
    1. Go to the VM in Portal
    2. On the left side, go to Automation-> Export Template
    3. Download the template
2.	Add/modify the _additionalCapabilities_ section as mentioned [Example ARM Template](#example-arm-template).


**Step 2: Generate new PFX Cert (Existing VMs Only)**
- For existing VMs, it might be necessary to generate a new PFX. 
- To determine if so, try running an extension that uses protected settings first. 

**Example**
```
az vm extension set --subscription <subid> --resource-group <group> --vm-name<vm> --name CustomScript --publisher Microsoft.Azure.Extensions --version 2.0 --force-update --protected-settings {"commandToExecute": "date"}
```
If there's a failure, then try these steps:

**Testing if PFX is needed**

Execute the Custom Script Extension to test if there's a need to generate a new PFX certificate. If it fails to successfully run (not just install) due to FIPS then generating a new PFX certificate is needed.

**CLI example:**
```
az vm extension set --subscription <subid> --resource-group <group> --vm-name<vm> --name CustomScript --publisher Microsoft.Azure.Extensions --version 2.0 --force-update --protected-settings {"commandToExecute": "date"}
```

**Option 1: Deallocate/Reallocate the VM**

If you’re doing this step, first deploy the modified ARM Template or execute the az cli commands, then do this step.

> [!WARNING]
> We don't recommend option 1 on RHEL 9.5 or RHEL 9.6. The current WALinuxAgent (v2.7.0.6) has an issue that can send the Agent into an infinite loop if the machine is rebooted after enabling FIPS. 

**Workaround**

```
systemctl stop waagent
# apply the patch
sed -i -E '/(.+)(self._initialize_telemetry\(\))/s//\1# \2/' /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py
# verify the patch (the output of grep must be "        # self._initialize_telemetry()"; the "#" characters removes the call to the problematic code
grep self\._initialize_telemetry /usr/lib/python3.9/site-packages/azurelinuxagent/daemon/main.py
        # self._initialize_telemetry() <<<<< OUTPUT
systemctl restart waagent
```

**Option 2: Add a Keyvault Certificate**

Create the keyvault/certificate then add it to the modified ARM template and deploy.
- [Get started with Key Vault certificates | Microsoft Learn](/azure/key-vault/certificates/certificate-scenarios)

**Example** (“properties” section of the VM model):

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

## Example ARM Template
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "bicep",
      "version": "0.16.2.56959",
      "templateHash": "14427937023370378081"
    }
  },
  "parameters": {
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Username for the Virtual Machine."
      }
    },
    "adminPassword": {
      "type": "securestring",
      "minLength": 12,
      "metadata": {
        "description": "Password for the Virtual Machine."
      }
    },
    "dnsLabelPrefix": {
      "type": "string",
      "defaultValue": "[toLower(format('{0}-{1}', parameters('vmName'), uniqueString(resourceGroup().id, parameters('vmName'))))]",
      "metadata": {
        "description": "Unique DNS Name for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIpName": {
      "type": "string",
      "defaultValue": "myPublicIP",
      "metadata": {
        "description": "Name for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIPAllocationMethod": {
      "type": "string",
      "defaultValue": "Dynamic",
      "allowedValues": [
        "Dynamic",
        "Static"
      ],
      "metadata": {
        "description": "Allocation method for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIpSku": {
      "type": "string",
      "defaultValue": "Basic",
      "allowedValues": [
        "Basic",
        "Standard"
      ],
      "metadata": {
        "description": "SKU for the Public IP used to access the Virtual Machine."
      }
    },
    "OSVersion": {
      "type": "string",
      "defaultValue": "2022-datacenter-azure-edition",
      "allowedValues": [
        "2016-datacenter-gensecond",
        "2016-datacenter-server-core-g2",
        "2016-datacenter-server-core-smalldisk-g2",
        "2016-datacenter-smalldisk-g2",
        "2016-datacenter-with-containers-g2",
        "2016-datacenter-zhcn-g2",
        "2019-datacenter-core-g2",
        "2019-datacenter-core-smalldisk-g2",
        "2019-datacenter-core-with-containers-g2",
        "2019-datacenter-core-with-containers-smalldisk-g2",
        "2019-datacenter-gensecond",
        "2019-datacenter-smalldisk-g2",
        "2019-datacenter-with-containers-g2",
        "2019-datacenter-with-containers-smalldisk-g2",
        "2019-datacenter-zhcn-g2",
        "2022-datacenter-azure-edition",
        "2022-datacenter-azure-edition-core",
        "2022-datacenter-azure-edition-core-smalldisk",
        "2022-datacenter-azure-edition-smalldisk",
        "2022-datacenter-core-g2",
        "2022-datacenter-core-smalldisk-g2",
        "2022-datacenter-g2",
        "2022-datacenter-smalldisk-g2"
      ],
      "metadata": {
        "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version."
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v5",
      "metadata": {
        "description": "Size of the virtual machine."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "vmName": {
      "type": "string",
      "defaultValue": "simple-vm",
      "metadata": {
        "description": "Name of the virtual machine."
      }
    },
    "securityType": {
      "type": "string",
      "defaultValue": "TrustedLaunch",
      "allowedValues": [
        "Standard",
        "TrustedLaunch"
      ],
      "metadata": {
        "description": "Security Type of the Virtual Machine."
      }
    }
  },
  "variables": {
    "storageAccountName": "[format('bootdiags{0}', uniqueString(resourceGroup().id))]",
    "nicName": "myVMNic",
    "addressPrefix": "10.0.0.0/16",
    "subnetName": "Subnet",
    "subnetPrefix": "10.0.0.0/24",
    "virtualNetworkName": "MyVNET",
    "networkSecurityGroupName": "default-NSG",
    "securityProfileJson": {
      "uefiSettings": {
        "secureBootEnabled": true,
        "vTpmEnabled": true
      },
      "securityType": "[parameters('securityType')]"
    },
    "extensionName": "GuestAttestation",
    "extensionPublisher": "Microsoft.Azure.Security.WindowsAttestation",
    "extensionVersion": "1.0",
    "maaTenantName": "GuestAttestation",
    "maaEndpoint": "[substring('emptyString', 0, 0)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2022-05-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {
        "defaultToOAuthAuthentication": true
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2022-05-01",
      "name": "[parameters('publicIpName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('publicIpSku')]"
      },
      "properties": {
        "publicIPAllocationMethod": "[parameters('publicIPAllocationMethod')]",
        "dnsSettings": {
          "domainNameLabel": "[parameters('dnsLabelPrefix')]"
        }
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2022-05-01",
      "name": "[variables('networkSecurityGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "default-allow-3389",
            "properties": {
              "priority": 1000,
              "access": "Allow",
              "direction": "Inbound",
              "destinationPortRange": "3389",
              "protocol": "Tcp",
              "sourcePortRange": "*",
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "*"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2022-05-01",
      "name": "[variables('virtualNetworkName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]",
              "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
      ]
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2022-05-01",
      "name": "[variables('nicName')]",
      "location": "[parameters('location')]",
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
              },
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
      ]
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2022-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "properties": {
		"additionalCapabilities": {
          "enableFips1403Encryption": "true"
        },  
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "[parameters('OSVersion')]",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "StandardSSD_LRS"
            }
          },
          "dataDisks": [
            {
              "diskSizeGB": 1023,
              "lun": 0,
              "createOption": "Empty"
            }
          ]
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        },
        "diagnosticsProfile": {
          "bootDiagnostics": {
            "enabled": true,
            "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2022-05-01').primaryEndpoints.blob]"
          }
        },
        "securityProfile": "[if(equals(parameters('securityType'), 'TrustedLaunch'), variables('securityProfileJson'), null())]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "condition": "[and(equals(parameters('securityType'), 'TrustedLaunch'), and(equals(variables('securityProfileJson').uefiSettings.secureBootEnabled, true()), equals(variables('securityProfileJson').uefiSettings.vTpmEnabled, true())))]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "apiVersion": "2022-03-01",
      "name": "[format('{0}/{1}', parameters('vmName'), variables('extensionName'))]",
      "location": "[parameters('location')]",
      "properties": {
        "publisher": "[variables('extensionPublisher')]",
        "type": "[variables('extensionName')]",
        "typeHandlerVersion": "[variables('extensionVersion')]",
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true,
        "settings": {
          "AttestationConfig": {
            "MaaSettings": {
              "maaEndpoint": "[variables('maaEndpoint')]",
              "maaTenantName": "[variables('maaTenantName')]"
            }
          }
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
      ]
    }
  ],
  "outputs": {
    "hostname": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName')), '2022-05-01').dnsSettings.fqdn]"
    }
  }
}
```