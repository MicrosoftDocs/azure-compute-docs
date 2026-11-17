---
title: Create a VM with compute templates in Azure (Preview)
description: Create a virtual machine with compute templates using Azure REST APIs. Follow this quickstart to deploy VMs step by step with Azure CLI.
ms.date: 11/05/2026
ms.topic: quickstart
ms.service: azure-virtual-machines
ai-usage: ai-assisted
---

# Quickstart: Create a VM with compute templates

> [!IMPORTANT]
> Compute templates are in **Limited preview**. Your subscription must be enabled for the preview.
See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability.

This quickstart shows you how to use Azure REST APIs to deploy a virtual machine (VM) by using compute templates in Azure.

Use Azure CLI commands to create the prerequisite resources. Then use `az rest` to call the Azure REST APIs to create an instance profile definition and version, and deploy a VM that references the version. Replace `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` in the examples with your subscription ID.

## Sign in to Azure and select a subscription

```azurecli
az login
az account list --output table
az account set --subscription "<your-subscription-id>"
az account show --output table
```

Example output:

```output
EnvironmentName HomeTenantId IsDefault Name State TenantId

----------------- ------------------------------------ ----------- ------------------------------------- ------- ------------------------------------

AzureCloud xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx True Visual Studio Enterprise Subscription Enabled xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## Check subscription access to the compute template APIs

Check that your subscription is registered for the preview. The response should show `state` as `Registered`.

```azurecli
az feature show \
    --namespace Microsoft.Compute \
    --name InstanceProfile
```

Example response:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Features/providers/Microsoft.Compute/features/InstanceProfile",
  "name": "Microsoft.Compute/InstanceProfile",
  "properties": {
    "state": "Registered"
  },
  "type": "Microsoft.Features/providers/features"
}
```

## Create a resource group

```azurecli
az group create \
    --name "rg-ip-preview-test" \
    --location "eastus2euap"
```

Example response:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test",
  "location": "eastus2euap",
  "managedBy": null,
  "name": "rg-ip-preview-test",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create a virtual network

```azurecli
az network vnet create \
    --resource-group "rg-ip-preview-test" \
    --name "vnet-ip-preview" \
    --location "eastus2euap" \
    --address-prefixes "10.0.0.0/16"
```

Example response:

```json
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"89b9de21-57ca-4da1-8892-b01ef17ad76e\"",
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/virtualNetworks/vnet-ip-preview",
    "location": "eastus2euap",
    "name": "vnet-ip-preview",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-ip-preview-test",
    "resourceGuid": "40f1ddc5-0687-455e-811b-c77869b6d13b",
    "subnets": [],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
```

## Create a subnet

```azurecli
az network vnet subnet create \
    --resource-group "rg-ip-preview-test" \
    --vnet-name "vnet-ip-preview" \
    --name "subnet-ip-preview" \
    --address-prefixes "10.0.1.0/24"
```

Example response:

```json
{
  "addressPrefix": "10.0.1.0/24",
  "delegations": [],
  "etag": "W/\"e694bef0-f2af-4b34-b5df-eae67561a5dd\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/virtualNetworks/vnet-ip-preview/subnets/subnet-ip-preview",
  "name": "subnet-ip-preview",
  "privateEndpointNetworkPolicies": "Disabled",
  "privateLinkServiceNetworkPolicies": "Enabled",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-ip-preview-test",
  "type": "Microsoft.Network/virtualNetworks/subnets"
}
```

## Create a network interface

```azurecli
az network nic create \
    --resource-group "rg-ip-preview-test" \
    --location "eastus2euap" \
    --name "nic-ip-vm-01" \
    --vnet-name "vnet-ip-preview" \
    --subnet "subnet-ip-preview"
```

Example response:

```json
{
  "NewNIC": {
    "auxiliaryMode": "None",
    "auxiliarySku": "None",
    "disableTcpStateTracking": false,
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDomainNameSuffix": "yxo5cqehazpelai1y32gtnwrhd.cbnx.internal.cloudapp.net"
    },
    "enableIPForwarding": false,
    "etag": "W/\"6ea1c3fd-02e0-4876-80a8-e26e5bd474d2\"",
    "hostedWorkloads": [],
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/networkInterfaces/nic-ip-vm-01",
    "ipConfigurations": [
      {
        "etag": "W/\"6ea1c3fd-02e0-4876-80a8-e26e5bd474d2\"",
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/networkInterfaces/nic-ip-vm-01/ipConfigurations/ipconfig1",
        "name": "ipconfig1",
        "primary": true,
        "privateIPAddress": "10.0.1.4",
        "privateIPAddressVersion": "IPv4",
        "privateIPAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-ip-preview-test",
        "subnet": {
          "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/virtualNetworks/vnet-ip-preview/subnets/subnet-ip-preview",
          "resourceGroup": "rg-ip-preview-test"
        },
        "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
      }
    ],
    "location": "eastus2euap",
    "name": "nic-ip-vm-01",
    "nicType": "Standard",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-ip-preview-test",
    "resourceGuid": "d550f89d-c156-469a-9975-bed2d2406453",
    "tapConfigurations": [],
    "type": "Microsoft.Network/networkInterfaces",
    "vnetEncryptionSupported": false
  }
}
```

> [!NOTE]
> This network interface is used in the VM create request, not stored in the instance profile. A network interface can be attached to only one VM. To reuse the instance profile for multiple VMs, provide a separate network interface in each VM create request, or define the virtual network and subnet in the instance profile to create network interfaces implicitly.

## Create an Azure Compute Gallery

```azurecli
az sig create \
    --resource-group "rg-ip-preview-test" \
    --gallery-name "testIpGallery12" \
    --location "eastus2euap"
```

Example response:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12",
  "identifier": {
    "uniqueName": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx-TESTIPGALLERY12"
  },
  "location": "eastus2euap",
  "name": "testIpGallery12",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-ip-preview-test",
  "type": "Microsoft.Compute/galleries"
}
```

## Create an instance profile definition

Save the following JSON as `instance-profile.json` in your current directory:

```json
{
  "tags": {
    "test": "1"
  },
  "location": "eastus2euap",
  "properties": {
    "description": "This is a test IP definition"
  }
}
```

Run the following command:

```azurecli
az rest \
    --method PUT \
    --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12/instanceProfiles/testIp1?api-version=2026-03-03" \
    --body "@instance-profile.json"
```

Example response:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12/instanceProfiles/testIp1",
  "location": "eastus2euap",
  "name": "testIp1",
  "properties": {
    "description": "This is a test IP definition",
    "provisioningState": "Succeeded"
  },
  "tags": {
    "test": "1"
  },
  "type": "Microsoft.Compute/galleries/instanceProfiles"
}
```

## Create an instance profile version

Save the following JSON as `instance-profile-version.json` in your current directory:

```json
{
  "name": "1.0.0",
  "location": "eastus2euap",
  "properties": {
    "targetLocations": [
      {
        "name": "eastus2euap"
      }
    ],
    "kind": "VirtualMachine",
    "virtualMachineProfile": {
      "hardwareProfile": {
        "vmSize": "Standard_D2s_v3"
      },
      "storageProfile": {
        "imageReference": {
          "publisher": "Canonical",
          "offer": "0001-com-ubuntu-server-jammy",
          "sku": "22_04-lts-gen2",
          "version": "latest"
        },
        "osDisk": {
          "createOption": "FromImage"
        }
      },
      "osProfile": {
        "adminUsername": "azureuser",
        "linuxConfiguration": {
          "disablePasswordAuthentication": true
        }
      }
    }
  }
}
```

Run the following command:

```azurecli
az rest \
    --method PUT \
    --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12/instanceProfiles/testIp1/versions/1.0.0?api-version=2026-03-03" \
    --body "@instance-profile-version.json"
```

Example response:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12/instanceProfiles/testIp1/versions/1.0.0",
  "location": "eastus2euap",
  "name": "1.0.0",
  "properties": {
    "excludeFromLatest": false,
    "kind": "VirtualMachine",
    "provisioningState": "Succeeded",
    "publishedDate": "2026-07-23T00:21:41.722368+00:00",
    "replicationStatus": {
      "aggregatedState": "Completed",
      "summary": [
        {
          "progress": 100,
          "region": "eastus2euap",
          "state": "Completed"
        }
      ]
    },
    "targetLocations": [
      {
        "name": "eastus2euap"
      }
    ],
    "virtualMachineProfile": {
      "hardwareProfile": {
        "vmSize": "Standard_D2s_v3"
      },
      "osProfile": {
        "adminUsername": "azureuser",
        "linuxConfiguration": {
          "disablePasswordAuthentication": true
        }
      },
      "storageProfile": {
        "imageReference": {
          "offer": "0001-com-ubuntu-server-jammy",
          "publisher": "Canonical",
          "sku": "22_04-lts-gen2",
          "version": "latest"
        },
        "osDisk": {
          "createOption": "FromImage"
        }
      }
    }
  },
  "type": "Microsoft.Compute/galleries/instanceProfiles"
}
```

## Create a VM that references the instance profile

Save the following JSON as `virtual-machine.json` in your current directory. Replace `<ssh-public-key>` with your SSH public key:

```json
{
  "location": "eastus2euap",
  "properties": {
    "instanceProfile": {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/galleries/testIpGallery12/instanceProfiles/testIp1/versions/1.0.0"
    },
    "hardwareProfile": {
      "vmSize": "Standard_D8s_v6"
    },
    "osProfile": {
      "computerName": "ip-vm-01",
      "adminUsername": "azureuser",
      "linuxConfiguration": {
        "ssh": {
          "publicKeys": [
            {
              "path": "/home/azureuser/.ssh/authorized_keys",
              "keyData": "<ssh-public-key>"
            }
          ]
        }
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/networkInterfaces/nic-ip-vm-01",
          "properties": {
            "primary": true
          }
        }
      ]
    }
  }
}
```

Run the following command:

```azurecli
az rest \
    --method PUT \
    --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/virtualMachines/ip-vm-01?api-version=2026-03-01" \
    --body "@virtual-machine.json"
```

Example response:

```json
{
  "etag": "\"2\"",
  "hardwareProfile": {
    "vmSize": "Standard_D8s_v6"
  },
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-IP-PREVIEW-TEST/providers/Microsoft.Compute/virtualMachines/ip-vm-01",
  "location": "eastus2euap",
  "name": "ip-vm-01",
  "networkProfile": {
    "networkInterfaces": [
      {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Network/networkInterfaces/nic-ip-vm-01",
        "primary": true,
        "resourceGroup": "rg-ip-preview-test"
      }
    ]
  },
  "osProfile": {
    "adminUsername": "azureuser",
    "allowExtensionOperations": true,
    "computerName": "ip-vm-01",
    "linuxConfiguration": {
      "disablePasswordAuthentication": true,
      "enableVMAgentPlatformUpdates": true,
      "patchSettings": {
        "assessmentMode": "ImageDefault",
        "patchMode": "ImageDefault"
      },
      "provisionVMAgent": true
    },
    "requireGuestProvisionSignal": true,
    "secrets": []
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "RG-IP-PREVIEW-TEST",
  "storageProfile": {
    "dataDisks": [],
    "diskControllerType": "NVMe",
    "imageReference": {
      "exactVersion": "22.04.202607140",
      "offer": "0001-com-ubuntu-server-jammy",
      "publisher": "Canonical",
      "sku": "22_04-lts-gen2",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "deleteOption": "Detach",
      "diskSizeGB": 30,
      "managedDisk": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/rg-ip-preview-test/providers/Microsoft.Compute/disks/ip-vm-01_OsDisk_1_543bba9bf0e74bc798eff02716ee7f2c",
        "resourceGroup": "rg-ip-preview-test",
        "storageAccountType": "Premium_LRS"
      },
      "name": "ip-vm-01_OsDisk_1_543bba9bf0e74bc798eff02716ee7f2c",
      "osType": "Linux"
    }
  },
  "tags": {
    "platformsettings.host_environment.service.platform_optedin_for_rootcerts": "true"
  },
  "timeCreated": "2026-07-28T18:57:52.0150041+00:00",
  "type": "Microsoft.Compute/virtualMachines",
  "vmId": "52466330-9c85-47d1-ae38-93c316ebd09d"
}
```

Confirm that the response shows `provisioningState` as `Succeeded`.

## Requests with overrides

The instance profile defines a baseline configuration. In the VM create request, you can override supported properties or add properties that aren't defined in the profile.

For example, the instance profile version specifies the following VM size:

```json
{
  "virtualMachineProfile": {
    "hardwareProfile": {
      "vmSize": "Standard_D2s_v3"
    }
  }
}
```

The VM create request overrides `hardwareProfile.vmSize` with `Standard_D8s_v6`. The response shows the size used for the new VM:

```json
{
  "etag": "\"2\"",
  "hardwareProfile": {
    "vmSize": "Standard_D8s_v6"
  }
}
```

## Next steps

- [Create VM with adaptive placement using compute templates](how-to-create-adaptive-placement-with-compute.md).
- [Review compute template capabilities and limitations](overview-compute-templates.md).
