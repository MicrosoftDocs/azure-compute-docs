---
title: Maintenance Configurations for Azure Virtual Machines using Bicep
description: Learn how to control when maintenance is applied to your Azure VMs by using Maintenance Configurations and Bicep.
author: ApnaLakshay
ms.service: azure-virtual-machines
ms.subservice: maintenance
ms.topic: how-to
ms.date: 11/5/2024
ms.author: lnagpal
#pmcontact: lnagpal
# Customer intent: As a cloud administrator, I want to configure maintenance schedules for Azure VMs using Bicep templates, so that I can manage updates and minimize downtime effectively.
---

# Control updates with Maintenance Configurations and Bicep

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

You can use the Maintenance Configurations feature to control when to apply platform updates to various Azure resources. This article covers the Bicep language for using this feature. For more information about the benefits of using the Maintenance Configurations feature, its limitations, and other management options, see [Managing platform updates with Maintenance Configurations](maintenance-configurations.md).

> [!IMPORTANT]
> Specific *scopes* support certain machine types and schedules. Be sure to select the right scope for your VM.

## Prerequisites

Before proceeding, visit [Bicep Modules](/azure/azure-resource-manager/bicep/modules) to learn more about how bicep deployment can be organized into modules

+ Scope of resource deployment should be understood before proceeding with the Bicep template. For more information, see [Scope Functions for Bicep](/azure/azure-resource-manager/bicep/bicep-functions-scope).
+ It's essential to know how and where resources are deployed.
+ Bicep modules must be used to perform deployments. Deploying resources at different scopes require the use of modules.

> [!TIP]
> Always provide the right set of dependencies since module execute on different scopes. For example - the scope for configuration assignment in maintenance configuration when using dynamic scope should be `Subscription` and when using static assignment (no dynamic scopes configured) should be `ResourceId`

## Bicep resource definition

 Refer to the below guides for the resource format and resource definition of the Bicep template before proceeding ahead with the working example.

+ [Maintenance Configurations](/azure/templates/microsoft.maintenance/maintenanceconfigurations)
+ [Configuration Assignments](/azure/templates/microsoft.maintenance/configurationassignments)
+ [Apply Updates](/azure/templates/microsoft.maintenance/applyupdates)

## Example

There are three bicep files that need to be created. We recommend keeping all the three files in the same folder. You can also keep them in different folders but make sure to update the path of the file names in `example.bicep` file mentioned.

### Template execution

```powershell

az deployment group create --name samplebicep --template-file .\ds.bicep --resource-group  testrg --parameters '{}' --debug --subscription abc4def8-er57-2345-f4t8-dff65f67afr3

```

#### Example template

##### example.bicep

```bicep
metadata description = 'Demo Main'

module mrpmc1 './maintenanceconfig.bicep' = {

  name: 'mrpmc'

  params: {

   configName: 'test-MaintConfig-Bicep'

  }

}

module ca1 './config_assignment.bicep' = {

  name: 'mrpmcconfigassign2'

  scope: subscription()

  params : {

  configId: '${resourceGroup().id}/providers/microsoft.maintenance/maintenanceConfigurations/test-MaintConfig-Bicep'

  }

  dependsOn: [

    mrpmc1

  ]

}
```

##### maintenanceconfig.bicep

```bicep
metadata description = 'Demo1'

param location string = 'Canada Central'

param configName string = ''

targetScope = 'resourceGroup'

resource MaintConfig4 'Microsoft.Maintenance/maintenanceConfigurations@2023-04-01' = {

  name: configName //'test-MaintConfig-Bicep'

  location: location

  tags: null

  properties: {

    extensionProperties: {

      InGuestPatchMode: 'user'

    }

    installPatches: {

      linuxParameters: {

        classificationsToInclude: [

          'Critical'

          'Security'

        ]

        packageNameMasksToExclude: null

        packageNameMasksToInclude: null

      }

      rebootSetting: 'RebootIfRequired'

      windowsParameters: {

        classificationsToInclude: [

          'Critical'

          'Security'

        ]

      //  excludeKbsRequiringReboot: false

        kbNumbersToExclude: null

        kbNumbersToInclude: null

      }

    }

    maintenanceScope: 'InGuestPatch'

    maintenanceWindow: {

      duration: '02:00'

      expirationDateTime: '9999-12-31 23:59:59'

      recurEvery: 'Day'

      startDateTime: '2023-12-29 12:00'

      timeZone: 'Mountain Standard Time'

    }

  //  namespace: 'string'

   //visibility: 'public'

  }

}

output mrpConfId string = MaintConfig4.id
```
##### config_assignment.bicep

```bicep
metadata description = 'Demo'

param location string = 'Canada Central'

param configId string = ''

targetScope = 'subscription'

resource symbolicname 'Microsoft.Maintenance/configurationAssignments@2023-04-01' = {

  name: 'myconfig'

  location: 'global'

  scope: subscription()

  properties: {

    filter: {

      locations: []

      osTypes: [

        'Linux'

        'Windows'

      ]

     // resourceGroups: null

      resourceTypes: [

        'microsoft.compute/virtualmachines'

        'microsoft.hybridcompute/machines'

      ]

      resourceGroups:[

        'RG-MaintConfigs'

      ]

     // tagSettings:{

    //  filterOperator: 'All'

     //   tags: {updates: ['true']}

     // }

    }

    maintenanceConfigurationId: configId

    //resourceId: ''

   

  }

}
```

##### Sample output

```bicep
{
  "id": "/subscriptions/eee2cef4-bc47-4278-b4f8-cfc65f25dfd8/resourceGroups/testrg/providers/Microsoft.Resources/deployments/samplebicep",
  "location": null,
  "name": "samplebicep",
  "properties": {
    "correlationId": "aaaa0000-bb11-2222-33cc-444444dddddd",
    "debugSetting": null,
    "dependencies": [
      {
        "dependsOn": [
          {
            "id": "/subscriptions/eee2cef4-bc47-4278-b4f8-cfc65f25dfd8/resourceGroups/testrg/providers/Microsoft.Resources/deployments/mrpmc",
            "resourceGroup": "testrg",
            "resourceName": "mrpmc",
            "resourceType": "Microsoft.Resources/deployments"
          }
        ],
        "id": "/subscriptions/eee2cef4-bc47-4278-b4f8-cfc65f25dfd8/providers/Microsoft.Resources/deployments/mrpmcconfigassign2",
        "resourceName": "mrpmcconfigassign2",
        "resourceType": "Microsoft.Resources/deployments"
      }
    ],
    "duration": "PT51.5773913S",
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/eee2cef4-bc47-4278-b4f8-cfc65f25dfd8/providers/Microsoft.Maintenance/configurationAssignments/myconfig"
      },
      {
        "id": "/subscriptions/eee2cef4-bc47-4278-b4f8-cfc65f25dfd8/resourceGroups/testrg/providers/Microsoft.Maintenance/maintenanceConfigurations/test-MaintConfig-Bicep",
        "resourceGroup": "testrg"
      }
    ],
    "outputs": null,
    "parameters": null,
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Resources",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              null,
              "canadacentral"
            ],
            "properties": null,
            "resourceType": "deployments",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "9584585695667229579",
    "templateLink": null,
    "timestamp": "2024-02-23T12:22:14.801966+00:00",
    "validatedResources": null
  },
  "resourceGroup": "testrg",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}
```

## Next steps

To learn more, see [Maintenance for virtual machines in Azure](maintenance-and-updates.md).
