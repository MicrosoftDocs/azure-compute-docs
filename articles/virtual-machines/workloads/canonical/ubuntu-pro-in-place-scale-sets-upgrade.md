---
title: In-place upgrade to Ubuntu Pro on Azure for VMSS
description: Learn how to do an in-place upgrade from Ubuntu Server to Ubuntu Pro for VMSS
author: MarckK
ms.service: azure-virtual-machines
ms.subservice: billing
ms.custom: linux-related-content, devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 07/07/2026
ms.author: kdelamarck
# Customer intent: "As a system administrator managing Ubuntu servers on Azure, I want to perform a VMSS in-place upgrade to Ubuntu Pro, so that I can access enhanced security features and support for my applications without experiencing downtime."
---

# VMSS: Ubuntu Server to Ubuntu Pro in-place upgrade on Azure

You can now upgrade your Ubuntu Server Virtual Machine Scale Sets (VMSS) to Ubuntu Pro without redeployment or downtime. The upgrade to Ubuntu Pro is available for Ubuntu LTS versions 16.04 or higher.

When an Ubuntu LTS image reaches its end of life (EOL), it no longer receives security updates. If you upgrade to Ubuntu Pro, you continue to receive security updates during [Extended Security Maintenance (ESM)](https://ubuntu.com/security/esm).
 
> [!IMPORTANT]
> EOL versions are not officially supported unless they have the PRO entitlement.
> For an updated list of versions that are EOL, check the official [Canonical Release Cycle](https://ubuntu.com/about/release-cycle) page.

## What's Ubuntu Pro?

Ubuntu Pro is a cross-cloud OS, optimized for Azure, and security maintained for 10 years. The
secure use of open-source software allows the operating system to use the latest technologies while
meeting internal governance and compliance requirements. Ubuntu Pro provides Extended Security Maintenance (ESM) for infrastructure and applications support, providing security patching for all Ubuntu packages.

## Available upgrade paths

The upgrade path is from Ubuntu LTS to Ubuntu Pro LTS of the same version. For example, you can upgrade from Ubuntu 18.04 LTS to Ubuntu Pro 18.04 LTS, but you can't upgrade from Ubuntu 18.04 LTS to Ubuntu Pro 20.04 LTS.

> [!IMPORTANT]
> Converting to Ubuntu Pro is an **irreversible** process.

> [!WARNING]
> - Only Ubuntu images on the Azure Marketplace **published by Canonical** can be converted to "UBUNTU_PRO"
> 
> - Images published by any other vendors or custom images are not supported.

The following method is recommended for Virtual Machine Scale Sets (VMSS) when customers wish to convert their servers from versions that have EOL status to Ubuntu Pro.

## Convert VMSS to Ubuntu Pro using the Azure CLI

The CLI commands to convert to Ubuntu Pro differ between VMSS orchestration modes. 

Azure supports two VMSS orchestration modes:  

- Uniform 

- Flexible 
 
## Determine the VMSS orchestration mode

The VMSS ARM resource contains an `orchestrationMode` property whose value is either "Uniform" or "Flexible".

The following command shows whether the `orchestrationMode` property is Uniform or Flexible:

```Azure CLI
az vmss show \ 
  -g $RG \ 
  -n $VMSS \ 
  --query orchestrationMode \ 
  -o tsv 
```

If the result is Uniform, follow the VMSS Uniform procedure. If the result is Flexible, follow the VMSS Flexible procedure. 
 
For more information about the differences between VMSS Uniform and VMSS Flexible orchestration modes, see [Virtual machine scale sets orchestration modes](../../../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).

## VMSS Uniform 

To upgrade VMSS Uniform to Ubuntu Pro, run the following commands:

```Azure CLI
az vmss update \ 
  -g $RG \ 
  -n $VMSS \ 
  --license-type UBUNTU_PRO 
```

```Azure CLI
az vmss update-instances \ 
  -g $RG \ 
  -n $VMSS \ 
  --instance-ids '*' 
```

```Azure CLI
az vmss list-instances \ 
  -g $RG \ 
  -n $VMSS \ 
  --query "[].id" \ 
  -o tsv | \ 
az vmss run-command invoke \ 
  --ids @- \ 
  --command-id RunShellScript \ 
  --scripts "pro auto-attach" 
```

## VMSS Flexible

To upgrade VMSS Flexible to Ubuntu Pro, run the following commands:

```Azure CLI
az vmss update \ 
  -g "$RG" \ 
  -n "$VMSS" \ 
  --license-type UBUNTU_PRO 
``` 

```Azure CLI 
az vmss update-instances \
-g $RG \
-n $VMSS \
--instance-ids '*'
```

The following command uses an `az vm` command instead of an `az vmss` command, due to how VMSS Flexible treats the underlying VMs. All VMs in the scale set are upgraded.

```Azure CLI 
az vm list \ 
  -g "$RG" \ 
  --query "[?starts_with(name, '${VMSS}_')].id" \ 
  -o tsv | \ 
az vm run-command invoke \ 
  --ids @- \ 
  --command-id RunShellScript \ 
  --scripts "pro auto-attach" \ 
  -o json 
```

## Validate the license

Use the `pro status --all` command to validate the license.

Expected output:

```output
SERVICE          ENTITLED           STATUS                DESCRIPTION
anbox-cloud      yes                disabled              Scalable Android in the cloud
cc-eal           yes                n/a                   Common Criteria EAL2 Provisioning Packages
esm-apps         yes                enabled               Expanded Security Maintenance for Applications
esm-infra        yes                enabled               Expanded Security Maintenance for Infrastructure
fips             yes                n/a                   NIST-certified FIPS crypto packages
fips-preview     yes                n/a                   Preview of FIPS crypto packages undergoing certification with NIST
fips-updates     yes                disabled              FIPS compliant crypto packages with stable security updates
landscape        yes                disabled              Management and administration tool for Ubuntu
livepatch        yes                enabled               Canonical Livepatch service
usg              yes                disabled              Security compliance and audit tools
```