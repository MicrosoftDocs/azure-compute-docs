---
title: Troubleshoot problems with Maintenance Configurations
description: This article provides details on known and fixed issues and how to troubleshoot problems with Maintenance Configurations.
author: ApnaLakshay
ms.service: azure-virtual-machines
ms.subservice: maintenance
ms.topic: concept-article
ms.date: 10/13/2023
ms.author: lnagpal
# Customer intent: "As a cloud administrator, I want to troubleshoot issues with Maintenance Configurations for virtual machines, so that I can ensure scheduled patches and updates are applied efficiently without errors."
---

# Troubleshoot problems with Maintenance Configurations

This article outlines common problems and errors that might arise during the deployment or use of Maintenance Configurations for scheduled patching on virtual machines (VMs), along with strategies to address them.

### A VM shuts down and is unresponsive when you use a static scope in guest maintenance

#### Problem

A maintenance configuration doesn't install a scheduled patch on the VMs and gives a `ShutdownOrUnresponsive` error.

#### Resolution

In a static scope, it's essential to avoid relying on outdated VM configurations. Therefore, it's imperative to ensure that the VM is up and running while the patch is being installed. **If the VM instance is recreated with the same name, it's crucial to prioritize reassigning the configuration after the VM instance is recreated.**

**The system will automatically perform this action within 12 hours of recreation if not performed by the user.** If the maintenance configuration of the attached recreated VM is triggered within 12 hours of recreating the VM with the same name, a `ShutdownOrUnresponsive` error is displayed. Therefore, it's recommended to either perform the previously mentioned action or wait for 12 hours before applying the patch to the machine via maintenance configuration.  

### Scheduled patching times out or fails

#### Problem

Scheduled patching fails with a `TimeOut` or `Failed` error after you move a VM by re-creating it with the same name in a different region. The portal might show the same VM twice because the previously created VM is removed from the back end.

#### Resolution

If the VM instance is recreated with the same name, it's crucial to prioritize reassigning the configuration after the VM instance is recreated in a different region.

**The system will automatically perform this action within 12 hours of recreation if not performed by the user.** If the maintenance configuration of the attached recreated VM is triggered within 12 hours of recreating the VM with the same name, a `TimeOut` or `Failed` error is displayed. Therefore, it's recommended to either perform the previously mentioned action or wait for 12 hours before applying the patch to the machine via maintenance configuration.  


### Unable to delete configuration assignment

#### Problem

Configuration assignment couldn't be removed or deleted from a particular maintenance configuration

#### Resolution

Use the following steps to mitigate this issue:

1. Delete the existing maintenance configuration in which you are encountering this issue.
1. Create a new maintenance configuration and assign the required set of dynamic scope and VMs as attached in the deleted maintenance configuration.

If you want to create new maintenance configuration with the same name as the deleted maintenance configuration, you will have to wait 20 minutes for the cleanup to take place in the backend. The system doesn't allow the creation of maintenance configuration with the same name if cleanup isn't performed in the backend. 

### Scheduled patching stops working after the resource is moved

#### Problem

If you move a resource to a different resource group or subscription, scheduled patching for the resource stops working.

#### Resolution

The system currently doesn't support moving resources across resource groups or subscriptions. As a workaround, use the following steps for the resource that you want to move. **As a prerequisite, first remove the assignment before following the steps.** 

If you're using a `static` scope:

1. Move the resource to a different resource group or subscription.
1. Re-create the resource assignment.

If you're using a `dynamic` scope:

1. Initiate or wait for the next scheduled run. This action prompts the system to completely remove the assignment, so you can proceed with the next steps.
1. Move the resource to a different resource group or subscription.
1. Re-create the resource assignment.

If any of the steps are missed, move the resource to the previous resource group or subscription ID and reattempt the steps.

> [!NOTE]
> If the resource group is deleted, recreate it with the same name. If the subscription ID is deleted, reach out to the support team for mitigation.

### Maintenance Configuration didn't trigger on the configured date time

#### Problem

After creating a Maintenance Configuration with a repeat value of either week or month, you expect the schedule to start at the specified date and time and then recur based on the chosen interval. However, the schedule didn't trigger at the start date and time.

#### Resolution

The Maintenance Configuration first run occurs on the first recurrence value following the specified start date, not necessarily on the start date itself. For example, if the maintenance configuration starts on January 17 (Wednesday) and is set to recur every Monday, the first run of the schedule will be on the first Monday after January 17, which is January 22.

You can view first 4 instances of the scheduled run when creating a new maintenance configuration from Azure Portal.

### Creation of a dynamic scope fails

#### Problem

You can't create a dynamic scope because of role-based access control (RBAC).

#### Resolution

To create a dynamic scope, you must have the permission at the subscription level or at the resource group level. Specifically, following are the requirements you need to take care of.

1. The subscription under which dynamic scope is getting created should be registered to Maintenance RP.
1. It's recommended to have the 'Scheduled Patching Contributor' role to be assigned to the following scopes:
    1. The subscription/resource group at which the dynamic scope is being created.
    1. The maintenance configuration scope.

For more information, see the [list of permissions list for various resources here](/azure/update-manager/roles-permissions#permissions).

### An update is stuck and not progressing

#### Problem

**Applies to:** :heavy_check_mark: Dedicated Hosts :heavy_check_mark: VMs

If you redeploy a resource to a different cluster, and you create a pending update request by using the old cluster value, the request becomes stuck indefinitely.

#### Resolution

If the status of an operation to apply an update is closed or not found, retry after 120 hours. If the problem persists, contact the support team for assistance.

### A dedicated host is updated after a maintenance configuration is attached

#### Problem

A maintenance configuration doesn't block the update of a dedicated host, and the host is updated even after you attach a maintenance configuration.

#### Resolution

If you re-create a dedicated host with the same name, Maintenance Configurations retains the old dedicated host ID, which prevents it from blocking updates. You can resolve this problem by removing the maintenance configuration and reassigning it.
**The system will automatically perform this action within 12 hours of recreation if not performed by the user.** It's recommended to either perform the previously mentioned action or wait for 12 hours before this action is performed automatically.

### A schedule isn't triggered

#### Problem

If a resource has two maintenance configurations with the same trigger time and patch installation configuration, and both are assigned to the same VM or resource, only one maintenance configuration is triggered.

#### Resolution

Modify the start time of one of the maintenance configurations to mitigate the problem. This is a workaround to a current system limitation in which Maintenance Configurations can't identify which maintenance configuration to trigger.

### You can't create a dynamic scope for a resource group

#### Problem

Dynamic scope validation fails because of a null value in the location.

#### Resolution

This problem with dynamic scope validation causes regression in the validation process. We recommend that you provide the required set of locations for a dynamic scope at the resource group level.

### A dynamic scope isn't executed and no resources are patched

#### Problem

Dynamic scope flattening fails because of throttling, and the service can't determine which VMs are associated with the VM.

#### Resolution

Make sure that the number of subscriptions per dynamic scope isn't more than 200. [Learn more about the service limits of dynamic scoping](../virtual-machines/maintenance-configurations.md#service-limits).

### Configuration assignment of a dedicated host isn't cleaned up after the host's removal

#### Problem

After you delete the dedicated hosts, configuration assignments that are attached to dedicated hosts still exist.

#### Resolution

Before you delete a dedicated host, be sure to delete the maintenance configuration that's associated with it. If the dedicated host is deleted but still appears in the portal, contact the support team for assistance. Cleanup processes are currently in place for dedicated hosts, to help prevent any impact on customers.

### Can't provide multiple tag values for dynamic scopes

#### Problem

If you use the Azure portal, you can't provide multiple tag values for dynamic scopes.

#### Resolution

This feature currently isn't available in the portal. As a workaround, you can use the Azure CLI or Azure PowerShell to create a dynamic scope. The system accepts multiple values for tags when you use the Azure CLI or Azure PowerShell option.

### A maintenance configuration is triggered again with an older trigger time

#### Problem

There's a known issue in Maintenance Configurations related to the caching of old maintenance policies. If an old policy is cached, and a new instance moves the new policy processing, the old machine might trigger the schedule with the outdated start time.

#### Resolution

We recommend that you update the maintenance configuration at least 1 hour before the scheduled time. If the problem persists, contact the support team for assistance.

### A maintenance configuration times out while waiting for an ongoing update to finish on a resource

#### Problem

In rare cases, if the host update window happens to coincide with the VM guest patching window, and if the guest patching window doesn't have sufficient time to run after the host update, the system shows this error message: "Schedule timeout, waiting for an ongoing update to complete the resource." The reason is that the platform allows only one update at a time.

#### Resolution

Change the maintenance configuration schedule for the guest update for a time after the ongoing update is finished.

### New maintenance configuration retains attributes of deleted configuration

#### Problem

A newly created maintenance configuration with the same name as a previously deleted configuration is inheriting properties from the deleted configuration.

#### Resolution

Ensure a minimum wait time of 20 minutes between deleting a maintenance configuration and creating a new one with the same name. If the problem persists, contact the support team for assistance.

### Maintenance assignment fails with `InternalServerError` error

#### Problem

Maintenance assignment fails with `InternalServerError` when using either bicep template or MRP API. The assignment can be done successfully using either portal or CLI or PowerShell.

#### Resolution

It is recommended to use the location/region in a normalized form for the bicep template and MRP API. The normalized form involves removing whitespace and converting the text to lowercase. For example, `EAST US 2` will result in an internal server error, whereas `eastus2` will be processed successfully.

### Workload isolation in maintenance configuration

#### Problem

Unexpected outage due to assumption of maintenance configuration respecting workload isolation

#### Resolution

Maintenance configurations will initiate updates on all resources associated with it simultaneously. Customers have the flexibility to decide on the isolation and maintenance windows for their workloads. They can create single or multiple maintenance configurations based on the isolation that is optimal for their workloads.

For example, if a customer wants to isolate updates for their development environment resources from the production environment resources, they can create separate maintenance configurations for each environment. This way, workloads in maintenance configuration with development resources are updated first, while the remaining workloads are updated in another configuration that does not overlap. This approach allows customers to control workload isolation and manage updates independently. 

### Maintenance Configurations doesn't support an API

The feature currently doesn't support the following APIs:

+ Get Apply Update at Subscription Level
+ Get Apply Update at Resource Group Level
+ Get Pending Update at Subscription Level
+ Get Pending Update at Resource Group Level
