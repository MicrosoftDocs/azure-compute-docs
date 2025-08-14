---
title: NGroups Rolling Update - Customer Documentation
description: Introduction to NGroups Rolling Update
ms.author: shivg
author: shivg
ms.service: azure-container-instances
ms.custom: devx-track-azurecli
services: container-instances
ms.topic: how-to
ms.date: 11/07/2024
# Customer intent: As a cloud administrator, I want to perform rolling updates on container groups in NGroups, so that I can ensure minimal workload disruption and maintain service availability during updates.
---

# NGroups Rolling Update

## Introduction

As requirements change, you may need to keep updating your NGroups and its container groups (CGs).

There are two update modes available for updating an NGroups – **Manual** (default option) and **Rolling**.  

Within **Rolling Update (RU)**, there are 2 options – [*in-place*](#in-place-rolling-update) update and [*replace*](#replace-rolling-update) update. Replace RU is the default option.

This document describes RU in detail. Manual update is covered in the NGroups documentation link [here](container-instances-about-ngroups.md).

## Feature Description

Consider a basic example of updating a CG profile reference from *cgprofile1* to *cgprofile2*.

### In-place Rolling Update

With in-place RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, *existing CGs* are updated with *cgprofile2*. The update to existing CGs happens in small batches (and not all at once). It ensures that there's a minimal impact on your workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because in-place RU updates existing CGs, there are certain limitations to the CG properties that Azure Container Instances (ACI) enforces.  

See ACI’s [limitations](../container-instances-update.md) around updating CGs. Also, see [here](../container-instances-update.md) for properties on CGs that require a delete (versus an update).

### Replace Rolling Update

With replace RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, *new CGs* are created with *cgprofile2*. Existing CGs with *cgprofile1* are deleted. This creation and deletion happen in small batches (and not all at once). It ensures that there's a minimal impact on your workload since only a small percentage of CGs is impacted during the update.  

Like in-place RU, we can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because replace RU creates new CGs, there are fewer limitations enforced by ACI. As a result, replace RU is a more powerful option and is the default option when a customer selects RU.

## Usage

### Triggering a Rolling Update

Rolling update is triggered when an NGroups PUT call is made and the CG profile in the PUT call is different from the CG profile currently referenced in the NGroups.  

NGroups then automatically group instances into batches and updates one batch at a time. The *maxBatchPercent* parameter determines the size of the batch.

#### Updating a Batch

- An **in-place** update invokes a CG PUT call to update each CG of the batch.

- A **replace** update invokes a CG PUT call to create new CGs and delete existing CGs of the batch. There exists a 1:1 correspondence between the CGs being created and the CGs being deleted. However, the CG names will be different.

If a sufficient number of CGs in the batch provide healthy signals after the pauseTimeBetweenBatches period, NGroups automatically starts the next batch for the update. Otherwise, it stops the rollout. The *maxUnhealthyPercent* parameter specifies the total number of unhealthy CGs, while the *maxUnhealthyUpdatedPercent* parameter specifies the total number of unhealthy CGs after the update.

Here is an example to issue a rolling update request to NGroups:

``` json
{ 
    "location": "{{location}}", 
    "properties": { 
        "updateProfile": { 
            "updateMode": "Rolling", 

            "rollingUpdateProfile": { 
                // Maximum percentage of total CGs which can be updated  
                // simultaneously by rolling update in one batch. 
                “maxBatchPercent”: “10”, // optional, defaults to 20% 

                // Maximum percentage of the total CGs across the whole NGroup  
                // that can be unhealthy at a time either by rolling update or health 
                // checks by liveness probes. If there are more unhealthy CGs than this,  
                // the current rolling update is marked as failed. 
                // This check is a prerequisite to start any batch. 
                “maxUnhealthyPercent”: “10”, // optional, defaults to 20% 

                // Maximum percentage of the updated CGs which can be in unhealthy state  
                // after each batch is updated. If there are more unhealthy CGs than this,  
                // the current rolling update is marked as failed. 
                “maxUnhealthyUpdatedPercent”: 10, // optional, defaults to 20% 

                // The wait time between batches after completing one batch of the rolling 
                // update and before starting the next batch. The time duration should  
                // be specified in ISO 8601 format for duration. 
                "pauseTimeBetweenBatches": "PT2M", // optional, defaults to PT1M 

                // A nullable boolean property. Default is null 
                // Sets the mode to either in-place RU (when true) or replace (default) RU. 
                “inPlaceUpdate”: null/false/true 
            } 
        }, 
        "containerGroupProfiles": [
            { 
                "resource": { 
                    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/cgp1" 
                } 
            } 
        ] 
    } 
} 
```

If image version is set to the **latest** tag for container images within the CG profile, then NGroups automatically picks up the latest image version during the RU. To prevent unexpected behavior in your application, it is recommended to not use the *latest* tag for images. Instead, use specific versions.  


> [!NOTE]
> In order to use replace RU, set the tag "rollingupdate.replace.enabled: true" for the NGroups resource.
> This tag is temporary, and it will not be required in the future.

``` json
“tags”: { 
    “rollingupdate.replace.enabled”: true 
} 
```

### Getting Status of a Running Rolling Update

For getting the latest status of your rolling update, you can use this REST API:

`GET /subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/latestRollingUpdate`

This returns a response containing relevant information about the RU.

### Canceling a Rolling Update

To cancel a rolling update, use the following API. Once canceled, the RU cannot be resumed/restarted. A new RU needs to be triggered.

`POST /subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/cancelRollingUpdate`

You do not need to provide a request body when calling this API.

It's also important to know that canceling an RU won't automatically roll it back to the previous state, but it will remain in the state that it was at the moment it was canceled.  For example, in case during an RU one wants to update a CG profile reference from *cgprofile1* to *cgprofile2*, and at some point the operation is canceled, some CGs might have the new profile *cgprofile2* and others remain with *cgprofile1*.

### Boundary of a Batch in a Rolling Update

The CGs of a specific batch in an RU do not cross a fault model boundary. A fault model represents a zone/fault-domain (FD) combination. For example, zone 1 / FD 0 is a fault model boundary, zone 1 / FD 1 is another fault model boundary, and zone 2 / FD 0 is yet another fault model boundary.

If a customer has a multi-zonal NGroups set up with three zones, a batch is confined to CGs belonging to only one zone at most. A batch never consists of CGs spread across multiple zones.

If a customer has a multi-zonal and multi-FD NGroups setup, a batch still consists of CGs belonging to only one FD in a single zone at most.

NGroups maintains this fault model boundary in a batch, even when the number of CGs selected for a batch is much less than the maxBatchPercent setting. It reflects that NGroups prefers safe updates over faster (and thus riskier) updates.

The only time a fault model boundary is crossed when the RU selects unhealthy CGs for the first batch. In this batch, the RU attempts to update all unhealthy CGs to improve the overall availability of NGroups. As a result, when updating unhealthy CGs, the RU may exceed the maxBatchPercent setting.

## Learn Related Topics

- [Azure Container Instances](../container-instances-overview.md)
- [About NGroups](container-instances-about-ngroups.md)
