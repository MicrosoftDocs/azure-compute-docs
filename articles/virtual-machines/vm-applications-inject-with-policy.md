---
title: Govern and enforce compliance for VM Applications with Azure Policy
description: Use Azure Policy to govern and enforce Azure Virtual Machine (VM) Application with right configurations across all VMs and Virtual Machine Scale Sets.
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

[Azure VM Applications](./vm-applications.md) let you package, version, and deliver software to Azure Virtual Machines (VMs) and Virtual Machine Scale Sets from an [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery).

Using [Azure Policy](/azure/governance/policy/overview) with VM Applications enables customers and admin teams to: 
- Monitor presence of required VM Applications across Virtual Machines & Virtual Machines Scale Sets. 
- Inject required VM Applications across all Virtual Machines & Virtual Machine Scale Sets.
- Enforce consistent configuration and best practices across all Virtual Machines and Virtual Machine Scale Sets for improved reliability and security.

This guide shows how to:
- Govern required VM application for compliance monitoring.
- Enforce required VM Application for driving consistency and security.


## Prerequisites
- An Azure Compute Gallery with the published VM Application and versions to enforce. See [create and manage VM Applications](/azure/virtual-machines/vm-applications#publish-a-vm-application-version).
    - Ensure the version is [replicated to all regions](./vm-applications-how-to.md#create-the-vm-application) where the application presence is required. 
    - Ensure the gallery is [shared to all subscriptions](./share-gallery.md) requiring access to the VM Application. 

- Permissions:
    - Policy Contributor to create/assign policies. See [policy definition structure and assignments](/azure/governance/policy/concepts/definition-structure).

## Set up compliance monitor to govern required VM applications

Azure Policy with `audit` effect can be used to monitor presence of specific VM Applications across Azure Virtual Machine and Virtual Machine Scale Sets. Admin teams can use this policy to 
- Setup compliance monitors for VM Applications (view compliant vs noncompliant Virtual Machines and Virtual Machine Scale Sets) 
- Measuring rollout progress of software packaged as a VM Application.

#### 1. Create a custom policy definition

[Create a new custom policy](/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy) using the following definition. This policy checks whether specific VM Applications are present on Virtual Machines and Virtual Machine Scale Sets, and reports compliance status.

#### [Policy](#tab/policy1)
This policy monitors compliance of all Virtual Machines, Virtual Machines Scale Sets across linux and windows. Linux apps on windows VMs and windows apps on linux VMs are considered as compliant resources. 

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
---

#### 2. Assign the policy and view compliance
[Assign the newly created policy](/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) to start monitoring the VM application and generate compliance score. All Virtual Machines and Virtual Machine Scale Sets within the assigned scope are monitored for compliance. 

Create separate assignments per VM application for granular and accurate monitoring.

Once the policy is assigned, all existing resources are evaluated and [displayed on compliance monitor](/azure/governance/policy/tutorials/create-and-manage#check-initial-compliance). Noncompliant resources are missing the VM Application defined in the policy. Resources without `applicationProfile` are also counted as noncompliant. Newly created or updated resources may take a few minutes to appear in evaluation cycles.

:::image type="content" source="./media/vmapps/vm-applications-compliance-monitor.png" alt-text="Screenshot that shows Azure Policy compliance view listing VMs and Virtual Machine Scale Sets audited for required VM Application presence.":::

#### Common adjustments

- **Audit multiple required applications**: Create a separate policy (one per application) or update the policy by adding other applications under `allOf` parameter.
- **Block creation of Virtual Machines without the required applications**: Change effect to `deny` to prevent noncompliant creations.
- **Limit policy scope to Virtual Machines or Virtual Machine Scale Sets**: Remove the unused branch from `anyOf` within `policyRule`.
- **Limit policy scope by OS type**: Check `osType` of `storageProfile` within `policyRule` to filter based on Window / Linux OS:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType", "equals": "[parameters('osType')]" }
    ```

- **Limit policy scope by OS Image**: Check `offer` and `sku` within `policyRule` to filter based on image:
    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer", "equals": "[parameters('imageOffer')]" },
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.sku", "equals": "[parameters('imageSku')]" }
    ```
---


## Inject VM Applications on Virtual Machine and Virtual Machine Scale Sets

Azure Policy with `modify` effect injects VM applications while creating new Virtual Machine and Virtual Machine Scale Sets. Remediation tasks can be used to modify existing resources and inject the VM Application. 

### Pre-requisite
- Managed identity on the policy assignment with **Virtual Machine Contributor** role, so remediation can modify resources. See [remediation identity requirements](/azure/governance/policy/how-to/remediate-resources#identity-requirements).

#### 1. Create a custom policy definition

[Create a new custom policy](/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy) using the policy definition. This policy checks for the presence of VM applications while creating Virtual Machines or Virtual Machine Scale Sets and appends the VM application if it doesn't exist. To modify multiple resource types (Virtual Machine and Virtual Machine Scale Sets), it's recommended to create different policies per resource type. 

#### [VM](#tab/vm)

```json
{
    "displayName": "Inject VM Application into single instance VMs",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Appends a VM Application reference to applicationProfile for single instance VMs if the application is not already present. Filters by OS type to prevent cross-platform injection.",
    "parameters": {
      "subscriptionId": {
        "type": "String",
        "metadata": {
          "description": "Subscription ID where the Compute Gallery resides."
        }
      },
      "resourceGroup": {
        "type": "String",
        "metadata": {
          "description": "Resource group of the Compute Gallery."
        }
      },
      "galleryName": {
        "type": "String",
        "metadata": {
          "description": "Name of the Compute Gallery."
        }
      },
      "applicationName": {
        "type": "String",
        "metadata": {
          "description": "VM Application name."
        }
      },
      "applicationVersion": {
        "type": "String",
        "defaultValue": "latest",
        "metadata": {
          "description": "VM Application version. Defaults to latest."
        }
      },
      "osType": {
        "type": "String",
        "allowedValues": [
          "Windows",
          "Linux"
        ],
        "metadata": {
          "description": "Target OS type. Ensures the application is only injected onto VMs with a matching OS."
        }
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/virtualMachines"
          },
          {
            "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType",
            "equals": "[parameters('osType')]"
          },
          {
            "count": {
              "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*]",
              "where": {
                "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*].packageReferenceId",
                "contains": "[concat('/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/')]"
              }
            },
            "equals": 0
          }
        ]
      },
      "then": {
        "effect": "modify",
        "details": {
          "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/9980e02c-c2be-4d73-94e8-173b1dc7cf3c"
          ],
          "operations": [
            {
              "operation": "add",
              "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications",
              "value": {
                "packageReferenceId": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.Compute/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/versions/', parameters('applicationVersion'))]",
                "order": 1,
                "treatFailureAsDeploymentFailure": true
              }
            }
          ]
        }
      }
    }
}
```

#### [Virtual Machine Scale Sets](#tab/vmss)

```json
{
    "displayName": "Inject VM Application into Virtual Machine Scale Sets",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Appends a VM Application reference to applicationProfile for Virtual Machine Scale Sets if the application is not already present. Filters by OS type to prevent cross-platform injection.",
    "parameters": {
      "subscriptionId": {
        "type": "String",
        "metadata": {
          "description": "Subscription ID where the Compute Gallery resides."
        }
      },
      "resourceGroup": {
        "type": "String",
        "metadata": {
          "description": "Resource group of the Compute Gallery."
        }
      },
      "galleryName": {
        "type": "String",
        "metadata": {
          "description": "Name of the Compute Gallery."
        }
      },
      "applicationName": {
        "type": "String",
        "metadata": {
          "description": "VM Application name."
        }
      },
      "applicationVersion": {
        "type": "String",
        "defaultValue": "latest",
        "metadata": {
          "description": "VM Application version. Defaults to latest."
        }
      },
      "osType": {
        "type": "String",
        "allowedValues": [
          "Windows",
          "Linux"
        ],
        "metadata": {
          "description": "Target OS type. Ensures the application is only injected onto Virtual Machine scale Sets with a matching OS."
        }
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/virtualMachineScaleSets"
          },
          {
            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.storageProfile.osDisk.osType",
            "equals": "[parameters('osType')]"
          },
          {
            "count": {
              "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*]",
              "where": {
                "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].packageReferenceId",
                "contains": "[concat('/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/')]"
              }
            },
            "equals": 0
          }
        ]
      },
      "then": {
        "effect": "modify",
        "details": {
          "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/9980e02c-c2be-4d73-94e8-173b1dc7cf3c"
          ],
          "operations": [
            {
              "operation": "add",
              "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications",
              "value": {
                "packageReferenceId": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.Compute/galleries/', parameters('galleryName'), '/applications/', parameters('applicationName'), '/versions/', parameters('applicationVersion'))]",
                "order": 1,
                "treatFailureAsDeploymentFailure": true
              }
            }
          ]
        }
      }
    }
}
```

---

#### 2. Assign the policy
[Assign the newly created policy](/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) to start generating compliance score. All Virtual Machines and Virtual Machine Scale Sets within the assigned scope are monitored for compliance. Once the policy is assigned, all existing resources are evaluated and [displayed on compliance monitor](/azure/governance/policy/tutorials/create-and-manage#check-initial-compliance).

Creating new Virtual Machines and Virtual Machine Scale Sets resource triggers this policy and modifies the applicationProfile of the resource to inject the application. 

#### 3. Create Remediation Task and modify existing resources
To modify existing resources, create a new [Remediation tasks](/azure/governance/policy/how-to/remediate-resources). 

> [!NOTE]
> Gradually remediate noncompliant resources for higher availability and failure resiliency. Create multiple remediation tasks, each scoped to one or more regions. 

:::image type="content" source="./media/vmapps/vm-applications-create-remediation-task.png" alt-text="Screenshot of Azure Portal experience showing how to create a new remediation task.":::

#### Common adjustments

The following examples show more conditions you can add to the `allOf` block within `policyRule` to narrow the policy scope.

- **Limit policy scope by OS image**: Filter based on a specific image offer and SKU:

    ```json
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.offer", "equals": "[parameters('imageOffer')]" },
    { "field": "Microsoft.Compute/virtualMachines/storageProfile.imageReference.sku", "equals": "[parameters('imageSku')]" }
    ```

- **Limit policy scope by region**: Target specific Azure regions:

    ```json
    { "field": "location", "in": "[parameters('allowedLocations')]" }
    ```

- **Limit policy scope by resource group or subscription**: To reduce the scope for remediation, assign the policy to a specific resource group or subscription. Use the [assignment scope](/azure/governance/policy/concepts/assignment-structure#scope) when assigning the policy rather than modifying the policy definition. For more granular control, use [exclusions](/azure/governance/policy/concepts/assignment-structure#excluded-scopes) to omit specific resource groups from the assignment.


## Next steps
- Learn more about [Azure VM Applications](vm-applications.md).
- Learn how to [create and deploy VM application packages](vm-applications-how-to.md).
- Learn how to [manage and delete Azure VM Applications](vm-applications-manage.md).
- Learn how to [create application package for VM Applications](vm-applications-create-app-package.md)
