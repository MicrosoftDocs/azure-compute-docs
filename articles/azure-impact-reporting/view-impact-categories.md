---
title: 'Get Impact Categories: Azure Impact Reporting' 
description: Learn how to get a valid list of impact categories. 
author: rolandnyamo
ms.author: ronyamo
ms.topic: overview
ms.service: azure 
ms.date: 10/30/2022
ms.custom: template-overview
# Customer intent: As a cloud user, I want to access a list of impact categories for Azure services so that I can accurately report issues related to resource operations and performance.
---

# Impact categories (preview)

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Review the full list of categories in the [REST API reference](https://aka.ms/ImpactRP/APIDocs).

## Category list

|Category name|Problem description|
|----------------------------------|------------------------------------------------------------------------------------------------------------------------|
|`ARMOperation.CreateOrUpdate`|To report problems related to creating a new Azure virtual machine, such as provisioning or allocation failures.|
|`ARMOperation.Delete`|To report failures in deleting a resource.|
|`ARMOperation.Get`|To report failures in querying resource metadata.|
|`ARMOperation.Start`|To report failures in starting a resource.|
|`ARMOperation.Stop`|To report failures in stopping a resource.|
|`ARMOperation.Other`|To report control plane operation failures that don't fall into other `ARMOperation` categories.|
|`Resource.Performance`|To report general performance issues. For example, high usage of CPU, IOPS, disk space, or memory.|
|`Resource.Performance.Network`|To report performance issues that are networking related. For example, degraded network throughput.|
|`Resource.Performance.Disk`|To report performance issues that are disk related. For example, degraded IOPS.|
|`Resource.Performance.CPU`|To report performance issues that are CPU related.|
|`Resource.Performance.Other`|To report issues that don't fall under other `Resource.Performance` subcategories.|
|`Resource.Connectivity`|To report general connectivity issues to or from a resource.|
|`Resource.Connectivity.Inbound`|To report inbound connectivity issues to a resource.|
|`Resource.Connectivity.Outbound`|To report outbound connectivity issues from a resource.|
|`Resource.Connectivity.Other`|To report issues that don't fall under other `Resource.Connectivity` subcategories.|
|`Resource.Availability`|To report general unavailability issues.|
|`Resource.Availability.Restart`|To report if an unexpected virtual machine restarts.|
|`Resource.Availability.Boot`|To report virtual machines that are in a nonbootable state, not booting at all, or on a reboot loop.|
|`Resource.Availability.Disk`|To report availability issues related to disks.|
|`Resource.Availability.UnResponsive`|To report a resource that isn't responsive now or for a time in the past.|
|`Resource.Availability.Storage`|To report availability issues related to storage.|
|`Resource.Availability.Network`|To report network availability issues.|
|`Resource.Availability.DNS`|To report Domain Name System availability issues.|
|`Resource.Availability.Other`|To report issues that don't fall under other `Resource.Availability` subcategories.|

## Related content

- [View previously filed impact reports](view-impact-insights.md)
