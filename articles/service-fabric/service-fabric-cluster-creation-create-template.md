---
title: Create an Azure Service Fabric cluster template 
description: Learn how to create a Resource Manager template for a Service Fabric cluster. Configure security, Azure Key Vault, and Microsoft Entra ID for client authentication.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: no-azure-ad-ps-ref
services: service-fabric
ms.date: 07/14/2022
# Customer intent: "As a cloud architect, I want to create a Resource Manager template for a Service Fabric cluster, so that I can efficiently deploy secure microservices with proper authentication and configuration management."
---

# Create a Service Fabric cluster Resource Manager template

An [Azure Service Fabric cluster](service-fabric-deploy-anywhere.md) is a network-connected set of virtual machines into which your microservices are deployed and managed. A Service Fabric cluster running in Azure is an Azure resource and is deployed, managed, and monitored using the Resource Manager.  This article describes how create a Resource Manager template for a Service Fabric cluster running in Azure.  When the template is complete, you can [deploy the cluster on Azure](service-fabric-cluster-creation-via-arm.md).

Cluster security is configured when the cluster is first set up and cannot be changed later. Before setting up a cluster, read [Service Fabric cluster security scenarios][service-fabric-cluster-security]. In Azure, Service Fabric uses x509 certificate to secure your cluster and its endpoints, authenticate clients, and encrypt data. Microsoft Entra ID is also recommended to secure access to management endpoints. Microsoft Entra tenants and users must be created before creating the cluster.  For more information, read [Set up Microsoft Entra ID to authenticate clients](service-fabric-cluster-creation-setup-aad.md).

Before deploying a production cluster to run production workloads, be sure to first read the [Production readiness checklist](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

## Create the Resource Manager template
Sample Resource Manager templates are available in the [Azure samples on GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). These templates can be used as a starting point for your cluster template.

This article uses the [five-node secure cluster][service-fabric-secure-cluster-5-node-1-nodetype] example template and template parameters. Download *azuredeploy.json* and *azuredeploy.parameters.json* to your computer and open both files in your favorite text editor.

> [!NOTE]
> For national clouds (Azure Government, Microsoft Azure operated by 21Vianet, Azure Germany), you should also add the following `fabricSettings` to your template: `AADLoginEndpoint`, `AADTokenEndpointFormat` and `AADCertEndpointFormat`.

## Add certificates
You add certificates to a cluster Resource Manager template by referencing the key vault that contains the certificate keys. Add those key-vault parameters and values in a Resource Manager template parameters file (*azuredeploy.parameters.json*).

### Add all certificates to the virtual machine scale set osProfile
Every certificate that's installed in the cluster must be configured in the **osProfile** section of the scale set resource (Microsoft.Compute/virtualMachineScaleSets). This action instructs the resource provider to install the certificate on the VMs. This installation includes both the cluster certificate and any application security certificates that you plan to use for your applications:

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "[parameters('sourceVaultValue')]"
          },
          "vaultCertificates": [
            {
              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
            },
            {
              "certificateStore": "[parameters('applicationCertificateStorevalue')",
              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

### Configure the Service Fabric cluster certificate

The cluster authentication certificate must be configured in both the Service Fabric cluster resource (Microsoft.ServiceFabric/clusters) and the Service Fabric extension for virtual machine scale sets in the virtual machine scale set resource. This arrangement allows the Service Fabric resource provider to configure it for use for cluster authentication and server authentication for management endpoints.

#### Add the certificate information the Virtual machine scale set resource

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType0Name'))]",
            "properties": {
              ...
              "settings": {
                ...
                "certificate": {
                  "commonNames": ["[parameters('certificateCommonName')]"],
                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
                },
                ...
              }
            }
          }
        ]
      }
    }
  }
}
```

#### Add the certificate information to the Service Fabric cluster resource

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  "location": "[parameters('clusterLocation')]",
  "dependsOn": [
    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
  ],
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
  }
}
```

<a name='add-azure-ad-configuration-to-use-azure-ad-for-client-access'></a>

## Add Microsoft Entra configuration to use Microsoft Entra ID for client access

You add the Microsoft Entra configuration to a cluster Resource Manager template by referencing the key vault that contains the certificate keys. Add those Microsoft Entra parameters and values in a Resource Manager template parameters file (*azuredeploy.parameters.json*). 

> [!NOTE]
> On Linux, Microsoft Entra tenants and users must be created before creating the cluster.  For more information, read [Set up Microsoft Entra ID to authenticate clients](service-fabric-cluster-creation-setup-aad.md).

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

## Populate the parameter file with the values

Finally, use the output values from the key vault and Microsoft Entra PowerShell commands to populate the parameters file.

If you plan to use the Azure service fabric RM PowerShell modules, then you do not need to populate the cluster certificate information. If you want the system to generate the self signed certificate for cluster security you, just keep them as null. 

> [!NOTE]
> For the RM modules to pick up and populate these empty parameter values, the parameters names much match the names below

```json
"clusterCertificateThumbprint": {
    "value": ""
},
"certificateCommonName": {
    "value": ""
},
"clusterCertificateUrlValue": {
    "value": ""
},
"sourceVaultvalue": {
    "value": ""
},
```

If you are using application certs or are using an existing cluster that you have uploaded to the key vault, you need to get this information and populate it.

The RM modules do not have the ability to generate the Microsoft Entra configuration for you, so if you plan to use the Microsoft Entra ID for client access, you need to populate it.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        ...
        "clusterCertificateStoreValue": {
            "value": "My"
        },
        "clusterCertificateThumbprint": {
            "value": "<thumbprint>"
        },
        "clusterCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
        },
        "applicationCertificateStorevalue": {
            "value": "My"
        },
        "applicationCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
        },
        "sourceVaultvalue": {
            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
        },
        "aadTenantId": {
            "value": "<guid>"
        },
        "aadClusterApplicationId": {
            "value": "<guid>"
        },
        "aadClientApplicationId": {
            "value": "<guid>"
        },
        ...
    }
}
```

## Test your template
Use the following PowerShell command to test your Resource Manager template with a parameter file:

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

In case you run into issues and get cryptic messages, then use "-Debug" as an option.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -Debug
```

The following diagram illustrates where your key vault and Microsoft Entra configuration fit into your Resource Manager template.

![Resource Manager dependency map][cluster-security-arm-dependency-map]

## Next steps
Now that you have a template for your cluster, learn how to [deploy the cluster to Azure](service-fabric-cluster-creation-via-arm.md).  If you haven't already, read the [Production readiness checklist](service-fabric-production-readiness-checklist.md) before deploying a production cluster.

To learn about the JSON syntax and properties for the resources deployed in this article, see:

* [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers)
* [Microsoft.Compute/virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets)

<!-- Links -->
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure
[resource-group-template-deploy]: ../azure-resource-manager/templates/deploy-powershell.md

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-create-template/cluster-security-arm-dependency-map.png
