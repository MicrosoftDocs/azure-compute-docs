---
title: Govern and enforce compliance for VM Applications with Azure Policy
description: Use Azure Policy to govern and enforce Azure Virtual Machine (VM) Application with right configurations across all VMs and VM scale sets.
author: tanmaygore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 12/05/2025
ms.author: tagore
ms.reviewer: jushiman
ms.custom: devx-track-azurepowershell, devx-track-azurecli
---

# Govern and enforce compliance for Azure VM Applications using Azure Policy

## Overview

[Azure VM Applications](/azure/virtual-machines/vm-applications) in Azure Compute Gallery let you package, version, and deliver software, files, scripts, security components, etc. from [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) to Azure VMs and VM scale sets (VMSS). [Learn more about Azure VM Applications.](./vm-applications.md)

Using [Azure Policy](/azure/governance/policy/overview) with VM Applications enables customers and admin teams to: 
- Monitor presence of required VM Applications across VMs & VM Scale Sets. 
- Inject required VM Applications across all VMs & VM scale sets.
- Enforce consistent configuration and best practices across all VMs and VMSS for improved reliability and security.

This guide shows how to:
- Govern required VM application for compliance monitoring.
- Enforce required VM Application for driving consistency and security.


## Prerequisites
- An Azure Compute Gallery with the published VM Application and versions to enforce. See [create and manage VM Applications](/azure/virtual-machines/vm-applications#publish-a-vm-application-version).
    - Ensure the version is [replicated to all regions](./vm-applications-how-to.md#create-the-vm-application) where the application presence is required. 
    - Ensure the gallery is [shared to all subscriptions](./share-gallery.md) requiring access to the VM Application. 

- Permissions:
    - Policy Contributor to create/assign policies. See [policy definition structure and assignments](/azure/governance/policy/concepts/definition-structure.md).

## Set up compliance monitor to govern required VM applications

Azure Policy with `audit` effect can be used to monitor presence of specific VM Applications across Azure VM and VM Scale Sets. Admin teams can use this to 
- Setup compliance monitors for VM Applications (view compliant vs noncompliant VMs and VM Scale Sets) 
- Measuring rollout progress of software packaged as a VM Application.

#### 1. Create a custom policy definition

[Create a new custom policy](/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy) using the policy definition provided below. This policy checks for the presence of VM applications on VMs and VM Scale Sets and reports the number of compliance and noncompliant instances.

#### [Policy](#tab/policy1)
This policy monitors compliance of all VMs, VMSS across linux and windows. Linux apps on windows VMs and windows apps on linux VMs are considered as compliant resources. 

```json
{
    "mode": "Indexed",
    "policyRule": {
        "if": {
            "anyOf": [
                {
                    "allOf": [
                        { "field": "type", "equals": "Microsoft.Compute/virtualMachines" },
                        {"field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType", "equals": "[parameters('osType')]" },
                        {
                            "count": {
                                "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*]",
                                "where": {
                                    "allOf": [
                                        {
                                            "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*].packageReferenceId",
                                            "contains": "[concat('/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/')]"
                                        }
                                    ]
                                }
                            },
                            "equals": 0
                        }
                    ]
                },
                {
                    "allOf": [
                        { "field": "type", "equals": "Microsoft.Compute/virtualMachineScaleSets" },
                        {"field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.storageProfile.osDisk.osType", "equals": "[parameters('osType')]" },
                        {
                            "count": {
                                "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*]",
                                "where": {
                                    "allOf": [
                                        {
                                            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].packageReferenceId",
                                            "contains": "[concat('/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/')]"
                                        }
                                    ]
                                }
                            },
                            "equals": 0
                        }
                    ]
                }
            ]
        },
        "then": { "effect": "audit" }
    },
    "parameters": {
        "galleryName": {
            "type": "String",
            "metadata": { "description": "Name of the Azure Compute Gallery containing the VM Application." }
        },
        "applicationName": {
            "type": "String",
            "metadata": { "description": "Name of the VM Application to audit for presence." }
        },
        "osType": {
            "type": "String",
            "allowedValues": [ "Windows", "Linux" ],
            "metadata": { "description": "OS type of the VM Application (Windows or Linux)." }
        }
    }
}
```

#### 2. Assign the policy and view compliance
[Assign the newly created policy](/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) to start monitoring the VM application and generate compliance score. All VMs and VM Scale Sets within the assigned scope are monitored for compliance. 

It's recommended to create separate assignments per VM application for granual and accurate monitoring.

Once the policy is assigned, all existing resources are evaluated and [displayed on compliance monitor](/azure/governance/policy/tutorials/create-and-manage#check-initial-compliance). Noncompliant resources are missing the VM Application defined in the policy. Resources without `applicationProfile` are also counted as noncompliant. Newly created or updated resources may take a few minutes to appear in evaluation cycles.

:::image type="content" source="media/vmapp/vm-applications-compliance-monitor.png" alt-text="Azure Policy compliance view showing VMs and VM scale sets audited for required VM Application presence.":::

#### Common adjustments

- <b>Audit multiple required applications</b>: Create a separate policy (one per application) or update the policy by adding other applications under `allOf` parameter.
- <b>Block creation of VMs without the required applications</b>: Change effect to `deny` to prevent noncompliant creations.
- <b>Limit policy scope to VMs or VMSS</b>: Remove the unused branch from `anyOf` within `policyRule`.
- <b>Limit policy scope by OS type</b>: Check `osType` of `storageProfile` within `policyRule` to filter based on Window / Linux OS:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType", "equals": "[parameters('osType')]" }
    ```

- Limit policy scope by OS Image: Check `offer` and `sku` within `policyRule` to filter based on image:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer", "equals": "[parameters('imageOffer')]" },
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.sku", "equals": "[parameters('imageSku')]" }
    ```
---

## Inject VM Applications on VM and VM Scale Sets

Azure Policy with `modify` effect injects VM applications while creating new VM and VM Scale Sets. Remediation tasks can be used to modify existing resources and inject the VM Application. 

### Pre-requisite
- Managed identity on the policy assignment (system-assigned) so remediation can modify resources. See [remediation identity requirements](/azure/governance/policy/how-to/remediate-resources#identity-requirements).

#### 1. Create a custom policy definition

[Create a new custom policy](/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy) using the policy definition. This policy checks for the presence of VM applications while creating VMs or VM Scale Sets and inserts the VM application if it doesn't exist. To modify multiple resource types (VM and VM Scale Sets), Azure policy requires different policies per resource type. 

#### [VM](#tab/vm)

```json
{
    "displayName": "Inject VM Application into Single Instance VMs",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Adds VM Application reference to applicationProfile for single instance VMs.",
    "parameters": {
      "subscriptionId": {
        "type": "String",
        "metadata": {
          "description": "Subscription ID where the Compute Gallery resides"
        }
      },
      "resourceGroup": {
        "type": "String",
        "metadata": {
          "description": "Resource group of the Compute Gallery"
        }
      },
      "galleryName": {
        "type": "String",
        "metadata": {
          "description": "Name of the Compute Gallery"
        }
      },
      "applicationName": {
        "type": "String",
        "metadata": {
          "description": "VM Application name"
        }
      },
      "applicationVersion": {
        "type": "String",
        "defaultValue": "latest",
        "metadata": {
          "description": "VM Application version (defaults to latest)"
        }
      }
    },
    "policyRule": {
      "if": {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
      },
      "then": {
        "effect": "modify",
        "details": {
          "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
          ],
          "operations": [
            {
              "operation": "addOrReplace",
              "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications",
              "value": [
                {
                  "packageReferenceId": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.Compute/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/versions/', parameters('applicationVersion'))]",
                  "order": 1,
                  "enableAutomaticUpgrade": true
                }
              ]
            }
          ]
        }
      }
    }
}
```

#### [VMSS](#tab/vmss)

```json
{
    "displayName": "Inject VM Application into VM Scale Sets",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Adds VM Application reference to applicationProfile for VM Scale Sets.",
    "parameters": {
      "subscriptionId": {
        "type": "String",
        "metadata": {
          "description": "Subscription ID where the Compute Gallery resides"
        }
      },
      "resourceGroup": {
        "type": "String",
        "metadata": {
          "description": "Resource group of the Compute Gallery"
        }
      },
      "galleryName": {
        "type": "String",
        "metadata": {
          "description": "Name of the Compute Gallery"
        }
      },
      "applicationName": {
        "type": "String",
        "metadata": {
          "description": "VM Application name"
        }
      },
      "applicationVersion": {
        "type": "String",
        "defaultValue": "latest",
        "metadata": {
          "description": "VM Application version (defaults to latest)"
        }
      }
    },
    "policyRule": {
      "if": {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachineScaleSets"
      },
      "then": {
        "effect": "modify",
        "details": {
          "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
          ],
          "operations": [
            {
              "operation": "addOrReplace",
              "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications",
              "value": [
                {
                  "packageReferenceId": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.Compute/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/versions/', parameters('applicationVersion'))]",
                  "order": 1,
                  "enableAutomaticUpgrade": true
                }
              ]
            }
          ]
        }
      }
    }
}
```

---

#### 2. Assign the policy
[Assign the newly created policy](/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) to start generating compliance score. All VMs and VM Scale Sets within the assigned scope are monitored for compliance. Once the policy is assigned, all existing resources are evaluated and [displayed on compliance monitor](/azure/governance/policy/tutorials/create-and-manage#check-initial-compliance).

Creating new VM and VMSS resource triggers this policy and modifies the applicationProfile of the resource to inject the application or configure it correctly. 

#### 3. Create Remediation Task and modify existing resources
Create a new [Remediation tasks](/azure/governance/policy/how-to/remediate-resources) to modify existing resources. 

It's recommended to gradually remediate noncompliant resources for higher availability and failure resiliency. This gradual can be done by creating multiple remediation tasks, each scoped to one region or few resources.

It's recommended to create separate policies for windows & linux.  

:::image type="content" source="media/vmapp/vm-applications-create-remediation-task.png" alt-text="Portal experience showing how to create a new remediation task.":::

#### Common adjustments
- <b>Limit policy scope by OS type</b>: Check `osType` of `storageProfile` within `policyRule` to filter based on Window / Linux OS:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType", "equals": "[parameters('osType')]" }
    ```
    

- <b>Limit policy scope by OS Image</b>: Check `offer` and `sku` within `policyRule` to filter based on image:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer", "equals": "[parameters('imageOffer')]" },
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.sku", "equals": "[parameters('imageSku')]" }
    ```
---


## Any other scenarios possible with Azure Policy?

- Deny creation of VMs/VMSS that don't reference only allowed VM Applications (allowlist). See [policy rule examples](/azure/governance/policy/samples/built-in-policies).
- Audit which VMs/VMSS aren't using enableAutomaticUpgrade = true for their VM Applications.
- Enforce treatFailureAsDeploymentFailure = true to fail deployments if app installation fails.
- Combine multiple policies into an [initiative](/azure/governance/policy/concepts/initiative-definition) so compliance is reported in one view and a single remediation can be run.