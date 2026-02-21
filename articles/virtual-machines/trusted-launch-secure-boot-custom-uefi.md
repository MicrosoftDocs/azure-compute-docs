---
title: Secure Boot UEFI keys 
description: Customers can replace, append secure boot UEFI keys – PK, KEK, DB, DBX.
author: prasadmsft
ms.author: reprasa
ms.service: azure-virtual-machines
ms.subservice: trusted-launch
ms.topic: concept-article
ms.date: 02/24/2025
ms.custom:
  - template-concept
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: "As a cloud administrator, I want to customize secure boot UEFI keys for my Azure virtual machines, so that I can ensure the integrity and security of the boot process according to my organization's requirements."
---

# Secure Boot UEFI keys

For certain advanced scenarios, it's necessary for customizing secure boot keys. The secure boot UEFI keys customization allows you to modify unified extensible firmware interface (UEFI) keys – PK, KEK, DB, DBX – in your image for a secure boot capable Azure virtual machine (VM) (Trusted Launch and Confidential VM). With this feature, UEFI keys can be fully replaced or appended to default key databases.

When a secure boot Azure VM is deployed, signatures of all the boot components such as UEFI, shim/bootloader, kernel, and kernel modules/drivers are verified during the boot process. Verification fails if the boot component signatures don't match with a key in the trusted key databases, and the VM fails to boot. This failure can occur if: a component is signed by a key not found in the trusted key databases, the signing key is listed in the revoked key database, or a component is unsigned. Through custom secure boot UEFI keys, the necessary signatures can be supplied for a successful VM boot.

UEFI key customization is offered through [Azure compute gallery](azure-compute-gallery.md) resources. If you currently use a Marketplace image to create a Trusted Launch VM, you must first create Azure compute gallery resources using the Marketplace image to modify UEFI keys.

> [!NOTE]
> Customizing UEFI keys is for advanced customers intimately familiar with inner workings of secure boot technology.


## Prerequisites

The following Azure Compute Gallery resources are necessary for customizing UEFI keys: 
- [Gallery](/azure/templates/microsoft.compute/galleries)
- [Image definition](/azure/templates/microsoft.compute/galleries/images)
- [Image version](/azure/templates/microsoft.compute/galleries/images/versions)

Currently, UEFI key customization is available only through the REST API.

You must attach a *VM image version* resource to a *VM image definition* resource with security-type set to *TrustedLaunchSupported* or *TrustedLaunchAndConfidentialVmSupported*. For detailed instructions on how to create an image definition resource, see [Create an image definition and an image version](image-version.md). To create a VM image definition resource, you must also create a gallery resource. You can choose the Azure portal, PowerShell, CLI, or REST API to create gallery and image definition resources.

### ARM template

A generic ARM template is provided as a sample for creating an image version resource. For customizing UEFI keys, you also need to include *security-profile* block in the image version resource. Each scenario in the [Scenarios section](#scenarios) provides details on what goes inside the *securityProfile* block.

```json
{
    "apiVersion": "2022-03-03",
    "type": "Microsoft.Compute/galleries/images/versions",
    "dependsOn": [
        "[concat('Microsoft.Compute/galleries/', parameters('galleryName'), '/images/', parameters('imageDefinitionName'))]"
    ],
    "name": "[concat(parameters('galleryName'), '/', parameters('imageDefinitionName'), '/', parameters('versionName'))]",
    "location": "[parameters('location')]",
    "properties": {
        "publishingProfile": {
            "replicaCount": "[parameters('defaultReplicaCount')]",
            "targetRegions": "[parameters('regionReplications')]",
            "excludeFromLatest": "[parameters('excludedFromLatest')]",
            "replicationMode": "[parameters('replicationMode')]"
        },
        "storageProfile": {
            "source": {
                "id": "[parameters('sourceVmId')]"
            }
        },
        "safetyProfile": {
            "allowDeletionOfReplicatedLocations": "[parameters('allowDeletionOfReplicatedLocations')]"
        },
        "securityProfile": {}
    },
    "tags": {}
}
```

## Scenarios

This section contains instructions on how to customize UEFI keys for common scenarios.

### Scenario 1: Replace all UEFI keys

In this scenario, you fully replace all default keys (PK, KEK, DB, DBX) available in a Trusted Launch VM. 

1. Create certificates and/or keys for each or PK, KEK, DB, and DBX. Refer to the [Useful links](#useful-links) section for detailed instructions for various operating systems.
2. Create a new image version. Use the following *securityProfile* ARM template block with the image version creation [Generic ARM template](#arm-template). Replace `Base64 formatted certificate` and `Base64 formatted sha256 hash` JSON values with the real values.

```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "NoSignatureTemplate"
        ],
        "additionalSignatures": {
            "pk": {
                "type": "x509",
                "value": [
                    "Base64 formatted certificate"
                ]
            },
            "kek": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "db": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "dbx": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                },
                {
                    "type": "sha256",
                    "value": [
                        "Base64 formatted sha256 hash",
                        "Base64 formatted sha256 hash"
                    ]
                }
            ]
        }
    }
}
```

3. Create a Trusted Launch VM using Azure portal, PowerShell, CLI, or REST API with the same source image as the one you created using Azure compute gallery resources.
4. Once the Trusted Launch VM is in the *running* state, [verify that your custom UEFI keys are in the UEFI key databases](#verify-keys-in-uefi-databases).

### Scenario 2: Append keys to DB and DBX to a signature template

In this scenario, you append DB and DBX keys to the default values available in a Trusted Launch VM.

1. Create certificates and/or keys for each or DB and DBX. Refer to the [Useful links](#useful-links) section for detailed instructions for various operating systems.
2. Create a new image version. Use the following *securityProfile* ARM template block with the image version creation [Generic ARM template](#arm-template). Replace `Base64 formatted certificate` and `Base64 formatted sha256 hash` JSON values with the real values.

```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "MicrosoftUefiCertificateAuthorityTemplate"
        ],
        "additionalSignatures": {
            "db": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "dbx": [
                {
                    "type": "sha256",
                    "value": [
                        "Base64 formatted sha256 hash",
                        "Base64 formatted sha256 hash"
                    ]
                }
            ]
        }
    }
}
```

3. Create a Trusted Launch VM using Azure portal, PowerShell, CLI, or REST API with the same source image as the one you created using Azure compute gallery resources.
4. Once the Trusted Launch VM is in the *running* state, [verify that your custom UEFI keys are in the UEFI key databases](#verify-keys-in-uefi-databases).

### Scenario 3: Apply multiple signature templates

In this scenario, you replace default signature templates and inject multiple signature templates (UEFI, Windows) into a Trusted Launch VM.

1. Create a new image version. Use the following *securityProfile* ARM template block with the image version creation [Generic ARM template](#arm-template).

```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "MicrosoftUefiCertificateAuthorityTemplate",
            "MicrosoftWindowsTemplate"
        ]
    }
}
```

2. Create a Trusted Launch VM using Azure portal, PowerShell, CLI, or REST API with the same source image as the one you created using Azure compute gallery resources.
3. Once the Trusted Launch VM is in the *running* state, [verify that your custom UEFI keys are in the UEFI key databases](#verify-keys-in-uefi-databases).

## Verify keys in UEFI databases

To verify the keys stored in UEFI databases, use the following commands in the command line (SSH) on the desired VM.

```bash
// For key(s) added to db
$ mokutil --db

// For keys(s) added to dbx
$ mokutil --dbx
```

Verify that the displayed keys are the ones supplied with the image version.

## Considerations 

- Clouds and regions are supported
- For public cloud, all regions are supported except the following regions: Austria East, Belgium Central, Chile Central, Indonesia Central, Israel Northwest, Jio India Central, Jio India West, Malaysia South, Malaysia West, Mexico Central, Qatar Central, South Central US 2, Southeast US, Southeast US 3, Switzerland West, Taiwan North, Taiwan Northwest, UK West, and West India
- USGov and China clouds are currently not supported

## Useful links

- How to convert Base64 certificates: [Base64 certificate encoding](https://www.base64encode.org/enc/certificate/)
- How to convert X.509 certificate to Base64: [X.509 Certificate Public Key in Base64](https://stackoverflow.com/questions/24492981/x-509-certificate-public-key-in-base64)
- How to sign things for secure boot: [Ubuntu](https://ubuntu.com/blog/how-to-sign-things-for-secure-boot), [Redhat](https://access.redhat.com/documentation/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-a-kernel-and-modules-for-secure-boot_managing-monitoring-and-updating-the-kernel), [Debian](https://wiki.debian.org/SecureBoot), [Oracle](https://docs.oracle.com/en/operating-systems/oracle-linux/9/secure-boot/sboot-SigningKernelModulesforUseWithSecureBoot.html#sb-mod-sign)
