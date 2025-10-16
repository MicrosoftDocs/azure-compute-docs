---
title: Govern and enforce compliance for VM Applications with Azure Policy
description: Use Azure Policy to govern and enforce Azure VM Application with right configurations across all VMs and VM scale sets.
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: vm-applications
ms.date: 2025-08-20
author: your-alias
ms.author: your-alias
---

# Govern and enforce compliance for Azure VM Applications using Azure Policy

## Overview

[Azure VM Applications](/azure/virtual-machines/vm-applications) let you package, version, and deliver softwares, files, and scripts from [Azure Compute Gallery](/azure/virtual-machines/azure-compute-gallery) to Azure VMs and VM scale sets (VMSS). [Learn more about Azure VM Applications.](./vm-applications.md)

Using [Azure Policy](/azure/governance/policy/overview) with VM Applications enables customers and admin teams to: 
- View all VMs & VM Scale Sets with and without the required VM Applications. 
- Inject required VM Applications across all VMs & VM scale sets.
- Enforce consistent configuration and best practices across all VMs and VMSS for improved reliability and security.

This guide shows how to:
- Inject a required VM Application during VM/VMSS creation.
- Remediate existing VMs/VMSS to have the right VM Application and settings.
- Govern gallery application versions for replication resiliency.


## Prerequisites

- Permissions:
    - Policy Contributor to create/assign policies. See [policy definition structure and assignments](/azure/governance/policy/concepts/definition-structure).
    - Contributor (or narrower, resource-scoped roles) on the target scope for modify/deployIfNotExists to update VMs/VMSS and gallery app versions.

- An Azure Compute Gallery with the published VM Application and versions to enforce. See [create and manage VM Applications](/azure/virtual-machines/vm-applications#publish-a-vm-application-version).
- Managed identity on the policy assignment (system-assigned) so remediation can modify resources. See [remediation identity requirements](/azure/governance/policy/how-to/remediate-resources#identity-requirements).

## Setup compliance monitor for required VM applications

Azure Policy with Audit effect can be used to identify VMs and VM Scale Sets without a specific VM Applications. Admin teams can leverage this to 
- Setup compliance monitors for VM Applications (view compliant vs non-compliant VMs and VM Scale Sets) 
- Measuring rollout progress of a mandatory platform or security agent packaged as a VM Application.

#### 1. Create a custom policy definition

[Create a new custom policy](/azure/governance/policy/tutorials/create-and-manage#implement-a-new-custom-policy) using the policy definition provided below. This policy checks for the presence of VM applications on VMs and VM Scale Sets and reports the number of compliance and non-compliant instances.

```json
{
    "mode": "Indexed",
    "policyRule": {
        "if": {
            "anyOf": [
                {
                    "allOf": [
                        { "field": "type", "equals": "Microsoft.Compute/virtualMachines" },
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
            "defaultValue": "testsigscus",
            "metadata": { "description": "Name of the Azure Compute Gallery containing the VM Application." }
        },
        "applicationName": {
            "type": "String",
            "defaultValue": "testappWindows",
            "metadata": { "description": "Name of the VM Application to audit for presence." }
        }
    }
}
```

#### 2. Assign the policy and view compliance
[Assign the newly created policy](/azure/governance/policy/tutorials/create-and-manage#assign-a-policy) to start generating compliance score. All VMs and VM Scale Sets within the assigned scope will be monitored for compliance. 

Once the policy is assigned, all existing resources are evaluated and [displayed on compliance monitor](/azure/governance/policy/tutorials/create-and-manage#check-initial-compliance). Non-compliant resources are missing the VM Application defined in the policy. Resources without `applicationProfile` are also counted as non-compliant. Newly created or updated resources may take a few minutes to appear in evaluation cycles.

#### Common adjustments

- Audit multiple required applications: Create a separate policy (one per application) or update the policy by adding other applications under `allOf` parameter.
- Switch to Deny for enforcement: change effect to deny (after validating), or create a second policy to prevent non-compliant creations.
- Restrict to only VMs or only VMSS: remove the unused branch from anyOf.

## Inject VM Applications on VM and VM Scale Sets

Azure Policy currently cannot use the Modify effect for VM Application aliases (applicationProfile.galleryApplications*), so use Audit (to report) and Deny (to block non-compliant creates); automatic injection must be performed outside Policy (for example via template deployment or script).



```json
{
    "mode": "Indexed",
    "parameters": {
        "galleryName": {
            "type": "String",
            "metadata": {
                "description": "Name of the Azure Compute Gallery containing the VM Application."
            }
        },
        "applicationName": {
            "type": "String",
            "metadata": {
                "description": "Name of the VM Application to require on new VMs."
            }
        }
    },
    "policyRule": {
        "if": {
            "allOf": [
                { "field": "type", "equals": "Microsoft.Compute/virtualMachines" },
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
        "then": { "effect": "deny" }
    }
}
```

Notes:
- Effect changed from audit to modify; conflictEffect=deny prevents conflicting manual values.
- conditions ensure injection only when the target application reference is absent.
- For remediation of existing resources, run a remediation task after assignment.
- If you prefer DeployIfNotExists, create two separate definitions each with a deployment template that PUTs the resource including applicationProfile, but Modify is simpler and idempotent here.

```json
{
    "properties": {
        "displayName": "Enforce VM Application on VMs (inject and configure)",
        "policyType": "Custom",
        "mode": "Indexed",
        "description": "Injects a required VM Application at VM create time and enforces required properties.",
        "metadata": {
            "category": "Compute"
        },
        "parameters": {
            "packageReferenceId": {
                "type": "String",
                "metadata": {
                    "description": "Resource ID of the gallery application or a specific version.",
                    "displayName": "Package reference ID"
                }
            },
            "order": {
                "type": "Integer",
                "defaultValue": 1,
                "metadata": {
                    "description": "Install order for the application.",
                    "displayName": "Install order"
                }
            },
            "treatFailureAsDeploymentFailure": {
                "type": "Boolean",
                "defaultValue": true,
                "metadata": {
                    "description": "If the application install fails, fail the VM deployment.",
                    "displayName": "Treat failure as deployment failure"
                }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    { "field": "type", "equals": "Microsoft.Compute/virtualMachines" }
                ]
            },
            "then": {
Notes:
- Modify effect is not supported for VM Application aliases today; use Deny (to block non-compliant creates) plus a separate Audit policy (to report existing gaps).
- Automatic injection/remediation must be handled outside Policy (script, template deployment, or future DeployIfNotExists approach).
- A DeployIfNotExists pattern would require a full VM (or VMSS) template update which is not shown here.
                    ],
                    "operations": [
                        {
{
    "properties": {
        "displayName": "Require VM Application on VMs (deny create if missing)",
        "policyType": "Custom",
        "mode": "Indexed",
        "description": "Denies creation of a VM that does not reference the required VM Application.",
        "metadata": {
            "category": "Compute"
        },
        "parameters": {
            "packageReferenceId": {
                "type": "String",
                "metadata": {
                    "description": "Resource ID of the required gallery application (optionally include /versions/<version>).",
                    "displayName": "Package reference ID"
                }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    { "field": "type", "equals": "Microsoft.Compute/virtualMachines" },
                    {
                        "count": {
                            "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*]",
                            "where": {
                                "allOf": [
                                    {
                                        "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*].packageReferenceId",
                                        "equals": "[parameters('packageReferenceId')]"
                                    }
                                ]
                            }
                        },
                        "equals": 0
                    }
                ]
            },
            "then": { "effect": "deny" }
        }
    }
}
                    "roleDefinitionIds": [
                        "/providers/microsoft.authorization/roledefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
                    ],
                    "operations": [
                        {
                            "operation": "Add",
                            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications",
                            "value": [
                                {
                                    "packageReferenceId": "[parameters('packageReferenceId')]",
                                    "order": "[parameters('order')]",
                                    "enableAutomaticUpgrade": true,
                                    "treatFailureAsDeploymentFailure": "[parameters('treatFailureAsDeploymentFailure')]"
                                }
                            ],
                            "condition": "[not(contains(field('Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].packageReferenceId'), parameters('packageReferenceId')))]"
                        },
                        {
                            "operation": "AddOrReplace",
                            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].enableAutomaticUpgrade",
                            "value": true
                        },
                        {
                            "operation": "AddOrReplace",
                            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].treatFailureAsDeploymentFailure",
                            "value": "[parameters('treatFailureAsDeploymentFailure')]"
                        }
                    ]
                }
            }
        }
    }
}
```

Assign at your target scope (subscription or resource group) with a system-assigned identity:

```bash
# Create definitions
az policy definition create \
{
    "properties": {
        "displayName": "Require VM Application on VMSS (deny create if missing)",
        "policyType": "Custom",
        "mode": "Indexed",
        "description": "Denies creation of a VM Scale Set that does not reference the required VM Application.",
        "metadata": {
            "category": "Compute"
        },
        "parameters": {
            "packageReferenceId": {
                "type": "String",
                "metadata": {
                    "description": "Resource ID of the required gallery application (optionally include /versions/<version>).",
                    "displayName": "Package reference ID"
                }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    { "field": "type", "equals": "Microsoft.Compute/virtualMachineScaleSets" },
                    {
                        "count": {
                            "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*]",
                            "where": {
                                "allOf": [
                                    {
                                        "field": "Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile.applicationProfile.galleryApplications[*].packageReferenceId",
                                        "equals": "[parameters('packageReferenceId')]"
                                    }
                                ]
                            }
                        },
                        "equals": 0
                    }
                ]
            },
            "then": { "effect": "deny" }
        }
    }
}
        "policyType": "Custom",
        "mode": "All",
        "metadata": { "category": "Compute" },
        "parameters": {
            "effect": {
                "type": "String",
                "allowedValues": [ "Deny", "Audit" ],
                "defaultValue": "Deny",
                "metadata": { "description": "Enable or audit the rule.", "displayName": "Effect" }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    { "field": "type", "equals": "Microsoft.Compute/galleries/applications/versions" },
                    {
                        "anyOf": [
                            {
                                "count": {
                                    "field": "Microsoft.Compute/galleries/applications/versions/publishingProfile.targetRegions[*]"
                                },
                                "less": 2
                            },
                            {
                                "field": "Microsoft.Compute/galleries/applications/versions/publishingProfile.targetRegions[*].regionalReplicaCount",
                                "less": 2
                            }
                        ]
                    }
                ]
            },
            "then": { "effect": "[parameters('effect')]" }
        }
    }
}
```

Assign it at the scope where you create gallery application versions. For the schema of targetRegions and replicas, see [Gallery Application Versions (REST)](/rest/api/compute/gallery-application-versions/create-or-update?tabs=HTTP#targetregion).

## Any other scenarios possible with Azure Policy?

- Deny creation of VMs/VMSS that do not reference only allowed VM Applications (allowlist). See [policy rule examples](/azure/governance/policy/samples/built-in-policies).
- Audit which VMs/VMSS are not using enableAutomaticUpgrade = true for their VM Applications.
- Enforce treatFailureAsDeploymentFailure = true to fail deployments if app installation fails.
- Combine the above into an [initiative](/azure/governance/policy/concepts/initiative-definition) so compliance is reported in one view and a single remediation can be run.

## What is and isn’t possible

- Create-time injection and enforcement on VM/VMSS: Possible. Use modify to add/normalize properties and conflictEffect = deny to block conflicting requests. See [effects](/azure/governance/policy/concepts/effects).
- Existing VMs/VMSS: Possible. Use remediation with the modify policies. See [remediation](/azure/governance/policy/how-to/remediate-resources).
- Enforcing “latest version”:
    - Possible if you reference the application (no version) and enforce enableAutomaticUpgrade = true. The platform moves VMs to the latest available version. See [automatic updates](/azure/virtual-machines/vm-applications#automatic-updates).
    - Not possible for policy to calculate “latest” and rewrite a hardcoded version string.
- targetRegions and replicas requirement: Possible, but must be enforced on gallery application versions (Microsoft.Compute/galleries/applications/versions). Policy assigned to VMs cannot change gallery replication. See [publish a VM Application version](/azure/virtual-machines/vm-applications#publish-a-vm-application-version).
- Operational limits: Policy cannot install apps if the VM agent is missing/offline, or when VMs are deallocated. Those cases remain noncompliant until the VM is healthy and remediation can run. See [VM agent](/azure/virtual-machines/extensions/overview).

## Next steps

- Convert the policies into a single initiative and assign at subscription or management group. See [create an initiative](/azure/governance/policy/tutorials/create-and-manage#create-and-assign-an-initiative-definition).
- Test in a non-production subscription. Validate:
    - New VMs/VMSS get the application injected with enableAutomaticUpgrade = true.
    - Existing resources are remediated.
    - Gallery app versions that don’t meet replication requirements are denied or audited.
- Monitor compliance and remediation progress in the [Azure Policy compliance dashboard](/azure/governance/policy/concepts/assignments#view-policy-compliance).

References:
- Managing VM Applications with Azure Policies (sample patterns and code): https://devblogs.microsoft.com/azure-vm-runtime/managing-vm-applications-with-azure-policies/
- Azure Policy overview: /azure/governance/policy/overview
- Policy definition structure: /azure/governance/policy/concepts/definition-structure
- Policy effects (Modify, DeployIfNotExists, Deny): /azure/governance/policy/concepts/effects
- Policy remediation: /azure/governance/policy/how-to/remediate-resources
- VM Applications: /azure/virtual-machines/vm-applications
- Azure Compute Gallery: /azure/virtual-machines/azure-compute-gallery
- Azure CLI for policy: /cli/azure/policy
- VMSS concepts: /azure/virtual-machine-scale-sets/overview
- Azure Resource Manager policy language: /azure/governance/policy/concepts/definition-structure#policy-rule
- Aliases browser (portal): /azure/governance/policy/concepts/definition-structure#aliases
- Gallery application versions REST (target regions): /rest/api/compute/gallery-application-versions/create-or-update?tabs=HTTP#targetregion