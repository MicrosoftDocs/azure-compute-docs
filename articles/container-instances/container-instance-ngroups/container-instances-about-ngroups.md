---
title: Introduction to NGroups
description: Introduction to NGroups and its features describing how customers can manage N instances of containers with NGroups.
ms.author: shivg
author: shivg
ms.service: azure-container-instances
ms.custom: devx-track-azurecli
services: container-instances
ms.topic: how-to
ms.date: 11/17/2025
# Customer intent: As a cloud architect, I want to manage multiple container group instances using a single API call through NGroups, so that I can streamline operations, enhance scalability, and improve the availability of my cloud applications.
---

# About NGroups (Preview)

Containers became the standard for packaging, deploying, and managing cloud applications, and effectively managing these containers is as crucial as running the apps themselves. [Azure Container Instances](../container-instances-overview.md) (ACI) is a flexible and scalable serverless computing service that allows you to run containerized applications without managing infrastructure.

NGroups provides you with advanced capabilities for managing multiple related container groups. Supported features include:

- Maintaining multiple instances
- Rolling upgrades
- High availability through Availability Zones (AZs)
- Managed identity support
- Confidential container support
- Load balancing
- Zone rebalancing (Zone Any)
  
 The NGroups feature builds upon ACI, ensuring container groups are secure, highly available, and supports the feature set of ACI.

For information about Azure Container Instances, see: [What is Azure Container Instances?](../container-instances-overview.md)

## NGroups High Level Architecture

:::image type="content" source="../media/container-instances-ngroups/ngroups-overview.png" alt-text="A Diagram that Shows NGroups High Level Workflow.":::

With Azure Container Instances, customers need to create and maintain each individual container group manually. NGroups offers an easier solution to create, update, and manage N container group instances with a single API call.

Creating an NGroups resource is a two step process.

1. Create a [Container Group Profile](#container-group-profile-cg-profile) (CGProfile), which serves as a template. In the CGProfile, a user specifies CG properties that are applied across all CGs created by NGroups.

2. Create an [NGroups](#ngroups) resource. You can provide the desired count (number of required CGs) and a reference to the container group profile along with other relevant properties.

NGroups references this Container Group Profile and then calls ACI APIs in order to create/update CGs with the properties mentioned in the CGProfile.

## Concepts

### Container Group Profile (CG Profile)

A large-scale cloud application may require you to manage multiple container groups. As of today, in order to run multiple CGs (Container Groups), customers need to provide relevant properties such as container images, restart policy, and other properties each time. This requirement can result in throttling, duplicated effort, and management overhead.

To alleviate this concern, NGroups introduced Container Group Profiles. The container group profile (CGProfile) serves as a *template* for creating container groups with same set of properties.  

Here are some of the common properties that can be specified in a container group profile:

- osType (Example: Linux, Windows)
- containers. Image name, memory, CPU, etc.
- restartPolicy
- ipAddress protocol and internal port
- shutdownGracePeriod
- timeToLive

And here's a sample CG profile:

``` json
{ 
    "location": "{{location}}", 
    "properties": { 
        "sku": "Standard", 
        "containers": [ 
            { 
                "name": "container1", 
                "properties": { 
                    "image": "nginx", 
                    "ports": [ 
                        { 
                            "protocol": "TCP", 
                            "port": 80 
                        } 
                    ], 
                    "resources": { 
                        "requests": { 
                            "memoryInGB": 2.0, 
                            "cpu": 1.0 
                        } 
                    } 
                } 
            } 
        ], 
        "restartPolicy": "Always", 
        "shutdownGracePeriod": "PT1H", 
        "ipAddress": { 
            "ports": [ 
                { 
                    "protocol": "TCP", 
                    "port": 80 
                } 
            ], 
            "type": "Public",
        }, 
        "timeToLive": "PT1H", 
        "osType": "Linux" 
    }     
}
```

### NGroups

NGroups resource provides a way to create and manage ‘n’ container groups with a rich set of operations. An NGroups resource references a container group profile resource and uses that to create N instances of similar looking CGs. Within NGroups resource, customers can also specify other properties including but not limited to number of CGs, update preferences (manual or rolling update), load balancers, subnets, and other relevant properties they want to associate with CGs under an NGroups resource.

> [!NOTE]
> A CG profile needs to be created **before** creating an NGroups resource. Since the CG profile is an Azure Resource Manager (ARM) resource, it has its own Azure Resource Manager APIs. A CG profile needs to be created **before** creating an NGroups resource.

#### Benefits of Referencing Container Group Profile

- Container group profile is a separate resource from NGroups. Customers can create multiple NGroups that can refer to the same container group profile. It also guarantees consistency across all the NGroups that refer to a single container group profile and avoids duplication.

- A single ACI CG could also be created from a CG profile. It allows you to rapidly move from prototype to production mode.

Here's a sample of an NGroups resource with managed identity and zones that refers to a container group profile and creates three container groups:

``` json
{ 
    "location": "{{location}}", 
    "properties": { 
        "elasticProfile": { 
            "desiredCount": 100 // specifies how many CGs to create
        }, 
        "containerGroupProfiles": [ 
            { 
                "resource": { 
                    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{{cgProfile1}}" 
                } 
            } 
        ] 
    } 
}
```

### NGroups Feature Highlights

- Offers both Rolling and Manual update
- Manage cross zonal container groups
- Supports managed identities
- Add load balancer and application gateway to manage traffic across container groups
- Manage container groups with different container group profiles
- Attach and detach container groups

### NGroups API

NGroups references a CG profile and adds other related properties and capabilities. Example:  

- The desired count of CGs to create or scale out
- The subnet into which CGs are deployed when using a virtual network
- The Load Balancer or Application Gateway to provide network ingress to the CGs

NGroups in turn invokes the ACI Azure Resource Manager APIs to create and manage each CG. Since it uses the same Azure Resource Manager APIs, there's no difference between the CGs created by NGroups and the CGs created directly by the customer. They have the exact same API experience.

### Updating an NGroups Resource

As requirements change, we would need to keep updating our NGroups and its CGs. There are two update modes with which we can update an NGroups – **Manual** (default option) and **Rolling**.

Consider a basic example of updating a CG profile reference from *cgprofile1* to *cgprofile2*:

- In Manual mode, we update the reference to cgprofile2 and send an UPDATE PUT request to NGroups:

NGroups stores this new CG profile reference. But it doesn't update existing CGs with this reference. Existing CGs are currently running and there's no impact on them. However, when NGroups is scaled out, the CGs are created with cgprofile2. 

- How do we update existing CGs with cgprofile2?  

To update existing CGs with new CGProfile, we issue a *manual update* command with an explicit list of CGs we want to update. This command only updates the CGs specified in its list. Updating the CG involves calling the ACI’s PUT CG API. The CGs not specified in this list continue to run with cgprofile1.

This mode gives us flexibility to update CGs selectively and provides full control over impact on production workloads.  

In **Rolling** mode, when we update the reference to cgprofile2 and issue an UPDATE NGroups command, existing CGs are updated with cgprofile2. The update to existing CGs happens in small batches (and not all at once). This ensures that there's a minimal impact on your workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.

## Try NGroups

### Prerequisites for Working on NGroups

The currently supported API version is **2024-09-01-preview**.

1. [Register the feature](/azure/azure-resource-manager/management/preview-features) `Microsoft.ContainerInstace/NGroupsPreview` on your subscriptions.

2. Once the feature flags are applied to the subscription, [register the resource provider](/azure/azure-resource-manager/management/resource-providers-and-types) `Microsoft.ContainerInstance` on your subscriptions.

> [!NOTE]
> Use the api-version - **2024-09-01-preview** and onwards for preview.

> [!TIP]
> Follow Azure Container Instance Swagger for up to date information on NGroups APIs. [Container Instance NGroups Swagger - 2024-11-01-preview](https://github.com/Azure/azure-rest-api-specs/blob/01791fcf5fc21ffa1b79b0558b3ec98e658c8caf/specification/containerinstance/resource-manager/Microsoft.ContainerInstance/preview/2024-11-01-preview/containerInstance.json#L837)

If these prerequisites aren't met, requests fail, and the NGroups resource type isn't recognized.

### ARM Template Samples

Create CG Profile: [ContainerGroupProfile-Sample.json](#container-group-profile-sample)
Create Zonal NGroups with CGProfile: [NGroups-Zonal-Sample.json](#ngroups-with-zones-sample)

Customers can see if a container group is associated to an NGroups resource by checking container group orchestratorId property under the JSON view. The orchestratorId represents the associated NGroups Azure Resource Manager resource ID.

:::image type="content" source="../media/container-instances-ngroups/cg-arm-json-structure.png" alt-text="A screenshot of an NGroups CG Azure Resource Manager JSON displaying OrchestratorId property.":::

## How-To Guide

### Perform a Rolling Update

We can use the Rolling Update feature to automatically update all CGs to a newer version without downtime of the NGroups.
See rolling update documentation: [NGroups Rolling update](container-instances-rolling-update.md).

### Create a Regional (zonal/non-zonal) NGroups

First create a CG profile. Here's a sample CG profile. Currently supported API version is **2024-09-01-preview**.

``` json
{ 
    "properties": { 
        "sku": "Standard", 
        "containers": [ 
            { 
                "name": "container1", 
                "properties": { 
                    "image": "nginx", 
                    "ports": [ 
                    { 
                        "protocol": "TCP", 
                        "port": 80 
                    }], 
                    "resources": { 
                        "requests": { 
                            "memoryInGB": 2.0, 
                            "cpu": 1.0 
                        } 
                    } 
                } 
            } 
        ], 
        "restartPolicy": "Always", 
        "shutdownGracePeriod": "PT1H", 
        "ipAddress": { 
            "ports": [ 
            { 
                "protocol": "TCP", 
                "port": 80 
            }], 
            "type": "Public"
        }, 
        "timeToLive": "PT1H", 
        "osType": "Linux" 
    }
```

Next, you can create a Zonal/Non-Zonal NGroups by either adding zones outside the properties or leaving the zones array empty.

``` json
{ 
    "properties": { 
        "elasticProfile": { 
            "desiredCount": 5 
        }, 
        "containerGroupProfiles": [ 
            { 
                "resource": { 
                    "id": "[resourceId('Microsoft.ContainerInstance/containerGroupProfiles', parameters('cgProfileName'))]" 
                } 
            } 
        ] 
    }, 
    "zones": [ "1", "2", "3" ] 
}
```

When NGroups is scaled out by setting its desiredCount property, the CGs are distributed evenly across all specified zones. If one zone goes down, the application remains available because the remaining CGs of NGroups continue to run in other zones.  

#### Can I Update CG Created By an NGroups Resource Directly Through ACI CG APIs?

Yes, customers have the flexibility to update container groups (CGs) directly by using the Azure Container Instances (ACI) APIs. For a deeper understanding of ACI container groups and to explore the related API options, check out this resource: [Container groups in Azure Container Instances](/azure/container-instances/container-instances-container-groups?branch=main)

While creating or updating container groups, NGroups relies on the same ACI APIs. This means that customers can use these APIs to update specific container groups as needed, without any extra configurations.

##### Technical Capabilities and Constraints

- Once an NGroups resource is created with a set of zones (for example, { “1”, “2” }), the zones can't be removed. However, a new zone can be added to the list. For example, { “1”, “2”, “3” }

- If a specified zone is down, then the overall NGroups operation to create the CGs fails. Retry the request once the zone is operational again. Another option is to delete the failed CGs.

- During scale down, NGroups randomly deletes instances, which might not always maintain AZ spread. However, subsequent scale-out operations always try to rebalance the AZ spread.  

- AZ spread isn’t supported with Spot containers. If you have such a requirement, reach out to the ACI team.

- See also: [impact of availability due to infrastructure/platform updates](#what-is-the-impact-of-availability-due-to-infrastructureplatform-updates).

#### Create NGroups CGs With a Prefix

Customers can create NGroups CGs with a prefix instead of just GUID names:

``` json
"properties": { 
    "elasticProfile": { 
        "desiredCount": 2,             
        "containerGroupNamingPolicy": { 
            "guidNamingPolicy": { 
                "prefix": "cg-" 
            } 
        } 
    },
```

This can be useful when you have multiple NGroups in a single resource group and want to differentiate CGs belonging to each NGroup (for example, in the Azure portal view). You can also change it for each scale-out operation to identify CGs that were scaled out together in one operation.

#### Create NGroups With Both System-Assigned and User-Assigned Managed Identities

``` json
“location”: “{{location}}” 
"identity": { 
    "type": "SystemAssigned, UserAssigned", 
    "userAssignedIdentities": { 
        "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity1}}": {},  
        "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity2}}": {} 
    }
```

#### If I delete some CGs from an NGroups, can the NGroups rebuild itself with new CGs to maintain its desired count?

Yes, you can set the properties.elasticProfile.maintainDesiredCount bool property to true.  

It creates a new CG for every CG that's being deleted/detached from the NGroups. It tries to maintain the desiredCount property of the NGroups at its set value.

This is useful when you want to use the NGroups as a *pool*, which automatically gets replenished when you take away CGs from the pool for your workload scenarios.  

It's a nullable bool property. If you omit it for subsequent NGroups PUT/update calls, it doesn't reset to false. To reset, you must explicitly set it to false. When it's null/false, and when a CG is deleted/detached from the NGroups, the desiredCount property for the NGroups reduces accordingly.

#### How do I get the CG name, NGroups ID and other metadata propagated down into the container?

Currently, we expose only the CG name and orchestrator ID (the Azure Resource Manager resource ID). In the future, other relevant properties could be considered. These two properties show up as container environment variables.

To get these environment variables on the container, specify these tags *at the NGroups level*:

``` json
tags: { 
    “metadata.container.environmentVariable.containerGroupName”: true, 
    “metadata.container.environmentVariable.orchestratorId”: true, 
    : 
    : // other NGroups tags you may have 
    : 
}
```

NGroups understands these tags as *special* and propagates the required environment variables down to each container as shown here.

:::image type="content" source="../media/container-instances-ngroups/cg-name-env-variable.png" alt-text="A screenshot of a container resource on Azure portal displaying environment variables containing 'ContainerGroupName' and 'OrchestratorId' properties.":::

#### What is the impact of availability due to infrastructure/platform updates?

For workloads that offer higher availability (for example, NGroups spread across multiple AZs), there's still a low possibility of CGs in more than one AZ going down at the same time. It can happen when the underlying Azure infrastructure (host machines, Virtual Machine Scale Sets, etc.) goes through an update (called as an infrastructure update or platform update).  

This update is done AZ by AZ with not much automated coordination across AZs. Coordination is manually tracked and best-effort.

So, if by chance, a platform update happens simultaneously across two or more AZs, then CGs across these AZs can be down simultaneously thus causing unavailability for your NGroups.

#### How to use Confidential Containers with NGroups

NGroups supports Confidential ACI container groups. Confidential instances are defined using the following properties within a Container Group Profile.

``` json
{ 
    "location": "{{location}}", 
    "properties": { 
        "sku": "Confidential",
        "confidentialComputeProperties": { 
            "ccePolicy": "<base 64 encoded policy>" 
          }, 
        "containers": [ ... ], 
        "restartPolicy": "Always", 
        "shutdownGracePeriod": "PT1H", 
        "ipAddress": { ... }, 
        "timeToLive": "PT1H", 
        "osType": "Linux" 
    }     
} 
```

Refer to the ACI documentation of confidential containers here: [Tutorial: Prepare a deployment for a confidential container on Azure Container Instances](../container-instances-tutorial-deploy-confidential-containers-cce-arm.md)

## Samples

### Container Group Profile Sample

``` json
{
    "properties": {
        "sku": "Standard",
        "containers": [
            {
                "name": "web",
                "properties": {
                    "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
                    "ports": [
                        {
                            "protocol": "TCP",
                            "port": 80
                        }
                    ],
                    "targetState": "Running",
                    "resources": {
                        "requests": {
                            "memoryInGB": 1,
                            "cpu": 1
                        }
                    }
                }
            }
        ],
        "restartPolicy": "Always",
        "shutdownGracePeriod": "PT2H",
        "ipAddress": {
            "ports": [
                {
                    "protocol": "TCP",
                    "port": 80
                }
            ],
            "type": "Public"
        },
        "osType": "Linux",
        "revision": 1
    },
    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{{cgProfile1}}",
    "name": "{{cgProfile1}}",
    "type": "Microsoft.ContainerInstance/containerGroupProfiles",
    "location": "{{location}}"
}
```

### NGroups with Zones Sample

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "apiVersion": {
      "type": "string",
      "maxLength": 32
    },
    "NGroupsName": {
      "type": "string",
      "maxLength": 64
    },
    "containerGroupProfileName": {
      "type": "string",
      "maxLength": 64
    },
    "resourceTags": {
      "type": "object"
    },
    "desiredCount": {
      "type": "int"
    }
  },
  "variables": {
    "description": "This ARM template can be parameterized for a basic CRUD scenario for NGroups. It is self contained with cgProfile and NGroups resource",
    "cgProfileName": "[parameters('containerGroupProfileName')]",
    "NGroupsName": "[parameters('NGroupsName')]",
    "resourcePrefix": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/')]"
  },
  "resources": [
    {
      "apiVersion": "[parameters('apiVersion')]",
      "type": "Microsoft.ContainerInstance/containerGroupProfiles",
      "name": "[variables('cgProfileName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "sku": "Standard",
        "containers": [
          {
            "name": "web",
            "properties": {
              "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "resources": {
                "requests": {
                  "memoryInGB": 1.0,
                  "cpu": 1.0
                }
              }
            }
          }
        ],
        "restartPolicy": "Always",
        "ipAddress": {
          "ports": [
            {
              "protocol": "TCP",
              "port": 80
            }
          ],
          "type": "Public"
        },
        "osType": "Linux"
      }
    },
    {
      "apiVersion": "[parameters('apiVersion')]",
      "type": "Microsoft.ContainerInstance/NGroups",
      "name": "[variables('NGroupsName')]",
      "tags": "[parameters('resourceTags')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.ContainerInstance/containerGroupProfiles/', variables('cgProfileName'))]"
      ],
      "identity": {
        "type": "systemAssigned"
      },
      "properties": {
        "elasticProfile": {
          "desiredCount": "[parameters('desiredCount')]",
          "maintainDesiredCount": true
        },
        "containerGroupProfiles": [
          {
            "resource": {
              "id": "[concat(variables('resourcePrefix'), 'Microsoft.ContainerInstance/containerGroupProfiles/', variables('cgProfileName'))]"
            }
          }
        ]
      },
      "zones": [ "1", "2", "3" ]
    }
  ]
}
```

## Related articles

- [Azure Container Instances](../container-instances-overview.md)
- [NGroups Rolling Update](container-instances-rolling-update.md)
- [Virtual nodes on Azure Container Instances](../container-instances-virtual-nodes.md)