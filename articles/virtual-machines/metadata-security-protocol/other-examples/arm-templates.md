---
title: ARM Templates
description: Example of ARM templates with MSP
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/15/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: As a cloud administrator, I want to deploy virtual machines using ARM templates with managed service provider (MSP) configurations, so that I can ensure consistent setup and management of virtual environments across both Windows and Linux platforms.
---

# ARM templates
This page provides examples of Azure Resource Manager (ARM) templates to:

- [Deploy a Virtual Machine (VM) with MSP on Windows](#deploy-vm-with-msp-on-windows)
- [Deploy a VM with MSP on Linux](#deploy-a-vm-with-msp-on-linux)

For both templates, `proxyAgentSettings` must be updated to configure MSP. Also, ensure the minimum API version is `2024-03-01`. For more details on the different settings for MSP, go to [Configuration](../configuration.md)

## Deploy VM with MSP on Windows

The JSON below is an example of an ARM template that can be used to deploy a VM with MSP enabled. 
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "languageVersion": "2.0",
  "parameters": {
    "proxyAgentEnabledValue": {
      "type": "bool",
      "nullable": true,
      "metadata": {
        "description": "Opt-in or Opt-out proxyagent feature"
      }
    },
    "proxyAgentWireServerSettingsJson": {
      "type": "object",
      "nullable": true,
      "metadata": {
        "description": "Input 'null' if you don't have any setting."
      },
      "properties": {
        "mode": {
          "type": "string",
          "allowedValues": ["Audit", "Enforce", "Disabled"],
          "nullable": true
        },
        "inVMAccessControlProfileReferenceId": {
          "type": "string",
          "nullable": true,
          "metadata": {
            "description": "Specifies the InVMAccessControlProfileVersion resource id on the form of /subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
          }
        }
      }
    },
    "proxyAgentImdsSettingsJson": {
      "type": "object",
      "nullable": true,
      "metadata": {
        "description": "Input 'null' if you don't have any setting."
      },
      "properties": {
        "mode": {
          "type": "string",
          "allowedValues": ["Audit", "Enforce", "Disabled"],
          "nullable": true
        },
        "inVMAccessControlProfileReferenceId": {
          "type": "string",
          "nullable": true,
          "metadata": {
            "description": "Specifies the InVMAccessControlProfileVersion resource id on the form of /subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
          }
        }
      }
    },
    "proxyAgentKeyIncarnation": {
      "type": "Int",
      "defaultValue": null,
      "metadata": {
        "description": "Increase this value to trigger a reset key flow. Set to 'null' if you don't care"
      },
      "nullable": true
    },
    "availabilitySet": {
      "type": "string",
      "defaultValue": null,
      "metadata": {
        "description": "Set to 'null' if you don't need to specify availabilitySet"
      },
      "nullable": true
    },
    "location": {
      "defaultValue": "[resourceGroup().location]",
      "type": "String",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "imageReference": {
      "type": "object",
      "defaultValue": {
        "publisher": "MicrosoftWindowsServer",
        "offer": "WindowsServer",
        "sku": "2019-Datacenter",
        "version": "latest"
      },
      "properties": {
        "publisher": {
          "type": "String",
          "nullable": true
        },
        "offer": {
          "type": "String",
          "nullable": true
        },
        "sku": {
          "type": "String",
          "nullable": true
        },
        "version": {
          "type": "String",
          "nullable": true
        },
        "sharedGalleryImageId": {
          "type": "String",
          "nullable": true
        }
      }
    },
    "vmSize": {
      "type": "String",
      "defaultValue": "Standard_DS1_v2"
    },
    "vmName": {
      "type": "String",
      "metadata": {
        "description": "Name of the virtual machine. Will also be used as computerName so please using simple character and no more than 9 characters."
      }
    }
  },
  "variables": {
    "nicName": "[format('{0}-VMNic', parameters('vmName'))]",
    "addressPrefix": "10.0.0.0/16",
    "subnetName": "Subnet",
    "subnetPrefix": "10.0.0.0/24",
    "virtualNetworkName": "[format('{0}-VNET', parameters('vmName'))]",
    "publicIpName": "[format('{0}-PublicIp', parameters('vmName'))]",
    "dnsLabelPrefix": "[toLower(format('{0}-{1}', parameters('vmName'), uniqueString(resourceGroup().id, parameters('vmName'))))]",
    "networkSecurityGroupName": "[format('{0}-NSG', parameters('vmName'))]",
    "availabilitySet": {
      "id": "[parameters('availabilitySet')]"
    }
  },
  "resources": {
    "publicIP": {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2022-05-01",
      "name": "[variables('publicIpName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Basic"
      },
      "properties": {
        "publicIPAllocationMethod": "Dynamic",
        "dnsSettings": {
          "domainNameLabel": "[variables('dnsLabelPrefix')]"
        }
      }
    },
    "nsg": {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2022-05-01",
      "name": "[variables('networkSecurityGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "rdp-3389",
            "properties": {
              "priority": 1000,
              "access": "Deny",
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
    "vNet": {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2022-05-01",
      "name": "[variables('virtualNetworkName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
      ],
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["[variables('addressPrefix')]"]
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
      }
    },
    "networkNic": {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2022-05-01",
      "name": "[variables('nicName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName'))]"
              },
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
              }
            }
          }
        ]
      }
    },
    "vm": {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2024-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "adminTest",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": "[parameters('imageReference')]"
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        },
        "securityProfile": {
          "proxyAgentSettings": {
            "enabled": "[parameters('proxyAgentEnabledValue')]",
            "wireServer": "[parameters('proxyAgentWireServerSettingsJson')]",
            "imds": "[parameters('proxyAgentImdsSettingsJson')]",
            "keyIncarnationId": "[parameters('proxyAgentKeyIncarnation')]"
          }
        },
        "availabilitySet": "[if(equals(parameters('availabilitySet'), 'null'), json('null'), variables('availabilitySet'))]"
      }
    }
  },
  "outputs": {
    "hostname": {
      "type": "String",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName')), '2022-05-01').dnsSettings.fqdn]"
    }
  }
}
```
## Deploy a VM with MSP on Linux

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "languageVersion": "2.0",
    "parameters": {
        "proxyAgentEnabledValue": {
            "type": "bool",
            "nullable": true,
            "metadata": {
                "description": "Opt-in or Opt-out proxyagent feature"
            }
        },
        "proxyAgentWireServerSettingsJson": {
            "type": "object",
            "nullable": true,
            "metadata": {
                "description": "Input 'null' if you don't have any setting."
            },
            "properties": {
                "mode": {
                    "type": "string",
                    "allowedValues": [
                        "Audit",
                        "Enforce",
                        "Disabled"
                    ],
                    "nullable": true
                },
                "inVMAccessControlProfileReferenceId": {
                    "type": "string",
                    "nullable": true,
                    "metadata": {
                        "description": "Specifies the InVMAccessControlProfileVersion resource id on the form of /subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
                    }
                }
            }
        },
        "proxyAgentImdsSettingsJson": {
            "type": "object",
            "nullable": true,
            "metadata": {
                "description": "Input 'null' if you don't have any setting."
            },
            "properties": {
                "mode": {
                    "type": "string",
                    "allowedValues": [
                        "Audit",
                        "Enforce",
                        "Disabled"
                    ],
                    "nullable": true
                },
                "inVMAccessControlProfileReferenceId": {
                    "type": "string",
                    "nullable": true,
                    "metadata": {
                        "description": "Specifies the InVMAccessControlProfileVersion resource id on the form of /subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
                    }
                }
            }
        },
        "proxyAgentKeyIncarnation": {
            "type": "Int",
            "defaultValue": null,
            "metadata": {
                "description": "Increase this value to trigger a reset key flow. Set to 'null' if you don't care"
            },
            "nullable": true
        },
        "availabilitySet": {
            "type": "string",
            "defaultValue": null,
            "metadata": {
                "description": "Set to 'null' if you don't need to specify availabilitySet"
            },
            "nullable": true
        },
        "location": {
            "defaultValue": "[resourceGroup().location]",
            "type": "String",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "imageReference": {
            "type": "object",
            "defaultValue": {
                "sharedGalleryImageId": "/sharedgalleries/0a2c89a7-a44e-4cd0-b6ec-868432ad1d13-cloudinitdirectsig/images/proxyagent-testing-with-cloudinit-mariner-2/versions/latest"
            },
            "properties": {
                "publisher": {
                    "type": "String",
                    "nullable": true
                },
                "offer": {
                    "type": "String",
                    "nullable": true
                },
                "sku": {
                    "type": "String",
                    "nullable": true
                },
                "version": {
                    "type": "String",
                    "nullable": true
                },
                "sharedGalleryImageId": {
                    "type": "String",
                    "nullable": true
                }   
            }                    
        },
        "vmSize": {
            "type": "String",
            "defaultValue": "Standard_D2s_v3"
        },
        "vmName": {
            "type": "String",
            "metadata": {
                "description": "Name of the virtual machine. Will also be used as computerName so please using simple character and no more than 9 characters."
            }
        }
    },
    "variables": {
        "nicName": "[format('{0}-VMNic', parameters('vmName'))]",
        "addressPrefix": "10.0.0.0/16",
        "subnetName": "Subnet",
        "subnetPrefix": "10.0.0.0/24",
        "virtualNetworkName": "[format('{0}-VNET', parameters('vmName'))]",
        "publicIpName": "[format('{0}-PublicIp', parameters('vmName'))]",
        "dnsLabelPrefix": "[toLower(format('{0}-{1}', parameters('vmName'), uniqueString(resourceGroup().id, parameters('vmName'))))]",
        "networkSecurityGroupName": "[format('{0}-NSG', parameters('vmName'))]",
        "availabilitySet": {
            "id": "[parameters('availabilitySet')]"
        }
    },
    "resources": {
        "publicIP":
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2022-05-01",
            "name": "[variables('publicIpName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Basic"
            },
            "properties": {
                "publicIPAllocationMethod": "Dynamic",
                "dnsSettings": {
                    "domainNameLabel": "[variables('dnsLabelPrefix')]"
                }
            }
        },
        "nsg":
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2022-05-01",
            "name": "[variables('networkSecurityGroupName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "rdp-3389",
                        "properties": {
                            "priority": 1000,
                            "access": "Deny",
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
        "vNet":
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2022-05-01",
            "name": "[variables('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
            ],
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
            }
        },
        "networkNic":
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2022-05-01",
            "name": "[variables('nicName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName'))]"
                            },
                            "subnet": {
                                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
                            }
                        }
                    }
                ]
            }
        },
        "vm":
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2024-03-01",
            "name": "[parameters('vmName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('vmSize')]"
                },
                "osProfile": {
                    "computerName": "[parameters('vmName')]",
                    "adminUsername": "adminTest",
                    "adminPassword": "!QAZ2wsx3edc"
                },
                "storageProfile": {
                    "imageReference": "[parameters('imageReference')]"
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                        }
                    ]
                },
                "securityProfile": {
                    "proxyAgentSettings": {
                        "enabled": "[parameters('proxyAgentEnabledValue')]",
                        "wireServer": "[parameters('proxyAgentWireServerSettingsJson')]",
                        "imds": "[parameters('proxyAgentImdsSettingsJson')]",
                        "keyIncarnationId": "[parameters('proxyAgentKeyIncarnation')]"                    
                    }   
                },
                "availabilitySet": "[if(equals(parameters('availabilitySet'), 'null'), json('null'), variables('availabilitySet'))]"
            }
        },
        "extension": {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "apiVersion": "2024-03-01",
            "name": "[format('{0}/AzureProxyAgentExtension', parameters('vmName'))]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
             ],
            "properties": {
                "publisher": "Microsoft.CPlat.ProxyAgent",
                "type": "ProxyAgentLinux",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": false,
                "enableAutomaticUpgrade": false,
                "settings": {}
            }       
        }
    },
    "outputs": {
        "hostname": {
            "type": "String",
            "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIpName')), '2022-05-01').dnsSettings.fqdn]"
        }
    }
}
```