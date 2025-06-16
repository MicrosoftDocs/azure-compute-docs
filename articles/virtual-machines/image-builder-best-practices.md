---
title: Best Practices for Azure VM Image Builder
description: This article describes best practices to follow while you're using Azure VM Image Builder.
author: sumit-kalra
ms.service: azure-virtual-machines
ms.topic: best-practice
ms.date: 03/25/2024
ms.reviewer: mattmcinnes
ms.subservice: image-builder
ms.custom: references_regions
---

# Best practices for Azure VM Image Builder

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article describes best practices to follow while you're using Azure VM Image Builder.

## Use resource locks for image templates

To prevent image templates from being accidentally deleted, use resource locks on them. For more information, see [Lock your Azure resources to protect your infrastructure](/azure/azure-resource-manager/management/lock-resources).

## Set up image templates for disaster recovery

Make sure your image templates are set up for disaster recovery by following the [reliability recommendations for VM Image Builder](/azure/reliability/reliability-image-builder?toc=/azure/virtual-machines/toc.json&bc=/azure/virtual-machines/breadcrumb/toc.json).

## Set up triggers

Set up VM Image Builder [triggers](image-builder-triggers-how-to.md) to automatically rebuild your images and keep them updated.

## Enable VM boot optimization

Enable [virtual machine (VM) boot optimization](vm-boot-optimization.md) in VM Image Builder to improve the creation time for your VMs.

## Specify subnets

Specify your own build VM subnets and Azure Container Instances subnets for tighter control over deployment of network-related resources by VM Image Builder in your subscription. Specifying these subnets also leads to faster build times for images. To learn more, see the [template reference](./linux/image-builder-json.md#vnetconfig-optional).

## Follow the principle of least privilege

Follow the [principle of least privilege](/entra/identity-platform/secure-least-privileged-access) for your VM Image Builder resources.

### Image template

A principal that has access to your image template can run, delete, or tamper with it. This access then allows the principal to change the images that image template created.

Make sure that only required principals can access image templates.

### Staging resource group

VM Image Builder uses a staging resource group in your subscription to customize your VM image. You must consider this resource group as sensitive and restrict access to only required principals. Keep these risks in mind:

- Because the process of customizing your image happens in this resource group, a principal that has access to the resource group can compromise the image-building process. For example, such a principal can inject malware into the image.

- VM Image Builder delegates privileges associated with the template identity and the build VM identity to resources in this resource group. A principal that has access to the resource group can get access to these identities.

- VM Image Builder maintains a copy of your customizer artifacts in this resource group. A principal that has access to the resource group can inspect these copies.

### Template identity

A principal that has access to your template identity can access all resources that the identity has permissions for. This set of resources includes your customizer artifacts (for example, shell and PowerShell scripts), your distribution targets (for example, an Azure Compute Gallery image version), and your virtual network.

You must provide only the minimum required privileges to this identity.

### Build VM identity

A principal that has access to your build VM identity can access all resources that the identity has permissions for. This set of resources includes any artifacts and virtual networks that you might be using from within the build VM via this identity.

You must provide only the minimum required privileges to this identity.

## Follow Azure Compute Gallery best practices

If you're distributing to Azure Compute Gallery, be sure to also follow [best practices for Azure Compute Gallery resources](azure-compute-gallery.md#best-practices).
