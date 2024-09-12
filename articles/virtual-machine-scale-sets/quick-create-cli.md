---
title: Quickstart - Create a Virtual Machine Scale Set with Azure CLI
description: Get started with your deployments by learning how to quickly create a Virtual Machine Scale Set with Azure CLI.
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: azure-virtual-machine-scale-sets 
ms.date: 06/14/2024
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli, mode-api, innovation-engine
---

# Quickstart: Create a Virtual Machine Scale Set with the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale sets

> [!NOTE]
> The following article is for Uniform Virtual Machine Scale Sets. We recommend using Flexible Virtual Machine Scale Sets for new workloads. Learn more about this new orchestration mode in our [Flexible Virtual Machine Scale Sets overview](flexible-virtual-machine-scale-sets.md).

A Virtual Machine Scale Set allows you to deploy and manage a set of auto-scaling virtual machines. You can scale the number of VMs in the scale set manually, or define rules to autoscale based on resource usage like CPU, memory demand, or network traffic. An Azure load balancer then distributes traffic to the VM instances in the scale set. In this quickstart, you create a Virtual Machine Scale Set and deploy a sample application with the Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0.29 or later of the Azure CLI. If using Azure Cloud Shell, the latest version is already installed.

## Create a resource group

Before you can create a scale set, create a resource group with [az group create](/cli/azure/group). The following example creates a resource group named *myResourceGroup* in the *eastus* location:

```azurecli-interactive
export RANDOM_ID=$(openssl rand -hex 3)
export MY_RESOURCE_GROUP_NAME="myResourceGroup$RANDOM_ID"
az group create --name $MY_RESOURCE_GROUP_NAME --location eastus
```

Results:
<!-- expected_similarity=0.3 -->
```json   
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroupxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create a scale set

> [!IMPORTANT]
>Starting November 2023, VM scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](
https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)


Now create a Virtual Machine Scale Set with [az vmss create](/cli/azure/vmss). The following example creates a scale set named *myScaleSet* that is set to automatically update as changes are applied, and generates SSH keys if they do not exist in *~/.ssh/id_rsa*. These SSH keys are used if you need to log in to the VM instances. To use an existing set of SSH keys, instead use the `--ssh-key-value` parameter and specify the location of your keys.

```azurecli-interactive
az vmss create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name myScaleSet \
  --image Debian11 \
  --orchestration-mode uniform \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

It takes a few minutes to create and configure all the scale set resources and VMs.

<!-- expected_similarity=0.3 -->
```json
{
  "vmss": {
    "doNotRunExtensionsOnOverprovisionedVMs": false,
    "orchestrationMode": "Uniform",
    "overprovision": true,
    "platformFaultDomainCount": 5,
    "provisioningState": "Succeeded",
    "singlePlacementGroup": true,
    "timeCreated": "2024-09-12T14:59:53.9329416+00:00",
    "uniqueId": "ac6d7ee7-f69f-498d-9ce4-dd325eb0f89e",
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
      "networkProfile": {
        "networkInterfaceConfigurations": [
          {
            "name": "myscaxxxxNic",
            "properties": {
              "disableTcpStateTracking": false,
              "dnsSettings": {
                "dnsServers": []
              },
              "enableAcceleratedNetworking": false,
              "enableIPForwarding": false,
              "ipConfigurations": [
                {
                  "name": "myscaxxxxIPConfig",
                  "properties": {
                    "loadBalancerBackendAddressPools": [
                      {
                        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/loadBalancers/myScaleSetLB/backendAddressPools/myScaleSetLBBEPool",
                        "resourceGroup": "myResourceGroupxxxxx"
                      }
                    ],
                    "privateIPAddressVersion": "IPv4",
                    "subnet": {
                      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/virtualNetworks/myScaleSetVNET/subnets/myScaleSetSubnet",
                      "resourceGroup": "myResourceGroupxxxxx"
                    }
                  }
                }
              ],
              "networkSecurityGroup": {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/networkSecurityGroups/myScaleSetNSG",
                "resourceGroup": "myResourceGroupxxxxx"
              },
              "primary": true
            }
          }
        ]
      },
      "osProfile": {
        "adminUsername": "azureuser",
        "allowExtensionOperations": true,
        "computerNamePrefix": "myscaxxxx",
        "linuxConfiguration": {
          "disablePasswordAuthentication": true,
          "provisionVMAgent": true,
          "ssh": {
            "publicKeys": [
              {
                "keyData": "ssh-rsa xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxXxxxx",
                "path": "/home/azureuser/.ssh/authorized_keys"
              }
            ]
          }
        },
        "requireGuestProvisionSignal": true,
        "secrets": []
      },
      "securityProfile": {
        "securityType": "TrustedLaunch",
        "uefiSettings": {
          "secureBootEnabled": true,
          "vTpmEnabled": true
        }
      },
      "storageProfile": {
        "diskControllerType": "SCSI",
        "imageReference": {
          "offer": "debian-11",
          "publisher": "Debian",
          "sku": "11-backports-gen2",
          "version": "latest"
        },
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage",
          "diskSizeGB": 30,
          "managedDisk": {
            "storageAccountType": "Premium_LRS"
          },
          "osType": "Linux"
        }
      },
      "timeCreated": "2024-09-12T14:59:53.9329416+00:00"
    }
  }
}
```

## Deploy sample application
To test your scale set, install a basic web application. The Azure Custom Script Extension is used to download and run a script that installs an application on the VM instances. This extension is useful for post deployment configuration, software installation, or any other configuration / management task. For more information, see the [Custom Script Extension overview](../virtual-machines/extensions/custom-script-linux.md).

Use the Custom Script Extension to install a basic NGINX web server. Apply the Custom Script Extension that installs NGINX with [az vmss extension set](/cli/azure/vmss/extension) as follows:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```

Results:
<!-- expected_similarity=0.3 -->
```json
{
  "additionalCapabilities": null,
  "automaticRepairsPolicy": null,
  "constrainedMaximumCapacity": null,
  "doNotRunExtensionsOnOverprovisionedVMs": false,
  "etag": "\"3\"",
  "extendedLocation": null,
  "hostGroup": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet",
  "identity": null,
  "location": "eastus",
  "name": "myScaleSet",
  "orchestrationMode": "Uniform",
  "overprovision": true,
  "plan": null,
  "platformFaultDomainCount": 5,
  "priorityMixPolicy": null,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resiliencyPolicy": null,
  "resourceGroup": "myResourceGroupxxxxx",
  "scaleInPolicy": null,
  "scheduledEventsPolicy": null,
  "singlePlacementGroup": true,
  "sku": {
    "capacity": 2,
    "name": "Standard_DS1_v2",
    "tier": "Standard"
  },
  "skuProfile": null,
  "spotRestorePolicy": null,
  "tags": {},
  "timeCreated": "2024-09-12T14:59:53.932941+00:00",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "uniqueId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx",
  "upgradePolicy": {
    "automaticOsUpgradePolicy": null,
    "mode": "Automatic",
    "rollingUpgradePolicy": null
  },
  "virtualMachineProfile": {
    "applicationProfile": null,
    "billingProfile": null,
    "capacityReservation": null,
    "diagnosticsProfile": null,
    "evictionPolicy": null,
    "extensionProfile": {
      "extensions": [
        {
          "autoUpgradeMinorVersion": true,
          "enableAutomaticUpgrade": null,
          "forceUpdateTag": null,
          "id": null,
          "name": "CustomScript",
          "protectedSettings": null,
          "protectedSettingsFromKeyVault": null,
          "provisionAfterExtensions": null,
          "provisioningState": null,
          "publisher": "Microsoft.Azure.Extensions",
          "settings": {
            "commandToExecute": "./automate_nginx.sh",
            "fileUris": [
              "https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"
            ]
          },
          "suppressFailures": null,
          "type": null,
          "typeHandlerVersion": "2.0",
          "typePropertiesType": "CustomScript"
        }
      ],
      "extensionsTimeBudget": null
    },
    "hardwareProfile": null,
    "licenseType": null,
    "networkProfile": {
      "healthProbe": null,
      "networkApiVersion": null,
      "networkInterfaceConfigurations": [
        {
          "auxiliaryMode": null,
          "auxiliarySku": null,
          "deleteOption": null,
          "disableTcpStateTracking": false,
          "dnsSettings": {
            "dnsServers": []
          },
          "enableAcceleratedNetworking": false,
          "enableFpga": null,
          "enableIpForwarding": false,
          "ipConfigurations": [
            {
              "applicationGatewayBackendAddressPools": null,
              "applicationSecurityGroups": null,
              "loadBalancerBackendAddressPools": [
                {
                  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/loadBalancers/myScaleSetLB/backendAddressPools/myScaleSetLBBEPool",
                  "resourceGroup": "myResourceGroupxxxxx"
                }
              ],
              "loadBalancerInboundNatPools": null,
              "name": "myscaxxxxIPConfig",
              "primary": null,
              "privateIpAddressVersion": "IPv4",
              "publicIpAddressConfiguration": null,
              "subnet": {
                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/virtualNetworks/myScaleSetVNET/subnets/myScaleSetSubnet",
                "resourceGroup": "myResourceGroupxxxxx"
              }
            }
          ],
          "name": "myscaxxxxNic",
          "networkSecurityGroup": {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/networkSecurityGroups/myScaleSetNSG",
            "resourceGroup": "myResourceGroupxxxxx"
          },
          "primary": true
        }
      ]
    },
    "osProfile": {
      "adminPassword": null,
      "adminUsername": "azureuser",
      "allowExtensionOperations": true,
      "computerNamePrefix": "myscaxxxx",
      "customData": null,
      "linuxConfiguration": {
        "disablePasswordAuthentication": true,
        "enableVmAgentPlatformUpdates": null,
        "patchSettings": null,
        "provisionVmAgent": true,
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxXxxxx",
              "path": "/home/azureuser/.ssh/authorized_keys"
            }
          ]
        }
      },
      "requireGuestProvisionSignal": true,
      "secrets": [],
      "windowsConfiguration": null
    },
    "priority": null,
    "scheduledEventsProfile": null,
    "securityPostureReference": null,
    "securityProfile": {
      "encryptionAtHost": null,
      "encryptionIdentity": null,
      "proxyAgentSettings": null,
      "securityType": "TrustedLaunch",
      "uefiSettings": {
        "secureBootEnabled": true,
        "vTpmEnabled": true
      }
    },
    "serviceArtifactReference": null,
    "storageProfile": {
      "dataDisks": null,
      "diskControllerType": "SCSI",
      "imageReference": {
        "communityGalleryImageId": null,
        "exactVersion": null,
        "id": null,
        "offer": "debian-11",
        "publisher": "Debian",
        "sharedGalleryImageId": null,
        "sku": "11-backports-gen2",
        "version": "latest"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "createOption": "FromImage",
        "deleteOption": null,
        "diffDiskSettings": null,
        "diskSizeGb": 30,
        "image": null,
        "managedDisk": {
          "diskEncryptionSet": null,
          "securityProfile": null,
          "storageAccountType": "Premium_LRS"
        },
        "name": null,
        "osType": "Linux",
        "vhdContainers": null,
        "writeAcceleratorEnabled": null
      }
    },
    "timeCreated": "2024-09-12T15:01:22.416655+00:00",
    "userData": null
  },
  "zonalPlatformFaultDomainAlignMode": null,
  "zoneBalance": null,
  "zones": null
}
```

## Allow traffic to port 80 
To allow traffic to flow through the load balancer to the virtual machines, the default network security group needs to be updated. 

```azurecli-interactive
az network nsg rule create \
  --name AllowHTTP \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --nsg-name myScaleSetNSG \
  --access Allow \
  --priority 1010 \
  --destination-port-ranges 80 \
```

Results:
<!-- expected_similarity=0.3 -->
```json 
{
  "access": "Allow",
  "destinationAddressPrefix": "*",
  "destinationAddressPrefixes": [],
  "destinationPortRange": "80",
  "destinationPortRanges": [],
  "direction": "Inbound",
  "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupxxxxx/providers/Microsoft.Network/networkSecurityGroups/myScaleSetNSG/securityRules/AllowHTTP",
  "name": "AllowHTTP",
  "priority": 1010,
  "protocol": "*",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroupxxxxx",
  "sourceAddressPrefix": "*",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}

```

## Test your scale set
To see your scale set in action, access the sample web application in a web browser. Obtain the public IP address of your load balancer with [az network public-ip show](/cli/azure/network/public-ip). The following example obtains the IP address for *myScaleSetLBPublicIP* created as part of the scale set:

```azurecli-interactive
az network public-ip show \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name myScaleSetLBPublicIP \
  --query '[ipAddress]' \
  --output tsv
```

Results:
<!-- expected_similarity=0.3 -->
``` 
17x.17x.21x.13x
```

Enter the public IP address of the load balancer in to a web browser. The load balancer distributes traffic to one of your VM instances, as shown in the following example:

![Default web page in NGINX](media/virtual-machine-scale-sets-create-cli/running-nginx-site.png)


## Clean up resources
When no longer needed, you can use [az group delete](/cli/azure/group) to remove the resource group, scale set, and all related resources as follows. The `--no-wait` parameter returns control to the prompt without waiting for the operation to complete. The `--yes` parameter confirms that you wish to delete the resources without an additional prompt to do so.

## Next steps
In this quickstart, you created a basic scale set and used the Custom Script Extension to install a basic NGINX web server on the VM instances. To learn more, continue to the tutorial for how to create and manage Azure Virtual Machine Scale Sets.

> [!div class="nextstepaction"]
> [Create and manage Azure Virtual Machine Scale Sets](tutorial-create-and-manage-cli.md)
