---
title: FAQ for converting Azure Cloud Services (extended support) apps to Service Fabric 
description: This guide answers most frequently asked questions (FAQ) for Cloud Services (extended support) migration to Service Fabric managed cluster.
ms.topic: how-to
ms.author: hirshah
author: hirshah
ms.service: azure-service-fabric
services: service-fabric
ms.date: 03/25/2025
# Customer intent: "As a cloud architect, I want to understand the migration process from Cloud Services (extended support) to Service Fabric managed clusters, so that I can ensure a seamless transition and leverage the improved capabilities before the retirement deadline."
---

# Frequently asked questions (FAQ)

This guide answers most frequently asked questions (FAQ) for Cloud Services (extended support) migration to Service Fabric managed cluster.

## Why this retirement?
Migrating from Cloud Services (Extended Support) to Service Fabric managed cluster enhances the scalability, flexibility, and reliability of your Azure deployments. We encourage you to transition to Service Fabric managed cluster before the retirement date to experience the advantages of faster deployment, high-density hosting, and distributed platform hosting.

## What is the retirement date?
From now until March 31, 2027, you can continue to use Cloud Services (extended support) without disruption. To avoid service disruption, you must  migrate workloads running Cloud Services (extended support) to Service Fabric managed clusters by March 31, 2027. 

## Can I request extension?
Unfortunately, we can't grant extension requests and customers must migrate within the two year retirement window until March 31, 2027.

## Pricing change
There's no pricing rate change because of this migration and customers continue to get billed at the same rate while using the same size.

## Migration tooling
Because migrating from Cloud Service (extended support) to Service Fabric managed cluster requires architecture and deployment modifications, there's no one-click migration tooling available. Each application must determine how its code runs on the VM/node and make custom adjustments accordingly.

