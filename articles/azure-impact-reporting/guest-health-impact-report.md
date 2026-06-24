---
title: Azure HPC Guest Health Reporting - Report Node Health 
description: Share the health status of a supercomputing virtual machine with Azure. 
author: bryantruong 
ms.author: bryantruong 
ms.service: azure 
ms.topic: overview 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# Report node health by using Guest Health Reporting (preview)

This article shows how to use Guest Health Reporting to share the health status of a supercomputing virtual machine (VM) with Azure. Before you begin, follow the instructions for onboarding and access management in the [feature overview](guest-health-overview.md).

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## REST client reporting

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Impact/workloadImpacts/{impactName}?api-version=2025-01-01-preview
```

Descriptions of URI parameters are as follows:

| Field name           | Description                                                                                          |
|----------------------|------------------------------------------------------------------------------------------------------|
| `subscriptionId`     | Subscription previously added to an allow list.                                                      |
| `impactName`         | Name or ID that you choose to identify the impact. This value must be unique — if you reuse an existing name in a new PUT request, you won't receive a 200 OK response. |
| `api-version`        | API version to be used for this operation. Use `2025-01-01-preview` or later to ensure compatibility with the `/getUploadToken` endpoint. |

### [Healthy node](#tab/healthy/)

```json
{
  "properties": {
      "startDateTime": "2025-05-13T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Healthy",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/{rg_name}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
      "additionalProperties": {
            "PhysicalHostName": "GGBB90904476"
      }
   }
}

```

### [Missing GPU](#tab/missingGPU/)

```json
{
  "properties": {
      "startDateTime": "2026-05-13T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.HpcMissingGpu",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/{rg_name}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
      "additionalProperties": {
            "LogUrl": "https://ghrloguploadprod.blob.core.windows.net/exampleCustomer/20260513150912_5273ea32.gz?",
            "PhysicalHostName": "GGBB90904476",
            "Manufacturer": "Nvidia",
            "SerialNumber": "12345679",
            "ModelNumber": "NV3LB225"
      }
   }
}

```

### [Investigate node](#tab/investigate/)

```json
{
  "properties": {
      "startDateTime": "2026-05-13T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Investigate.NVLink",
      "impactDescription": "NvLink may be down",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/{rg_name}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
      "additionalProperties": {
            "LogUrl": "https://ghrloguploadprod.blob.core.windows.net/exampleCustomer/20260513150912_5273ea32.gz"
      }
   }
}

```

### [Unhealthy non-GPU](#tab/unhealthynongpu/)

```json
{
  "properties": {
      "startDateTime": "2026-05-13T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.IBPerformance",
      "impactDescription": "IB low bandwidth",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/{rg_name}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
      "additionalProperties": {
            "LogUrl": "https://ghrloguploadprod.blob.core.windows.net/exampleCustomer/20260513150912_5273ea32.gz",
            "PhysicalHostName": "GGBB90904476"
      }
   }
}

```

> [!WARNING]
> The field names in GHR request bodies ARE case SENSITIVE. As a general rule-of-thumb, top-level fields within `properties` (`startDateTime`, `impactCategory`, etc.) are camelCase, while fields nested within `additionalProperties` (`LogUrl`, `PhysicalHostName`, etc.) are PascalCase.

---

| Field name           | Required | Data type  | Description                                                                  |
|----------------------|----------|------------|------------------------------------------------------------------------------|
| `startDateTime`      | Yes      | `datetime` | Time (in UTC) when the impact happened.                                      |
| `impactCategory`     | Yes      | `string`   | Observation type or fault scenario. Only an approved string list is allowed. |
| `impactDescription`  | Yes      | `string`   | Description of the reported impact.                                          |
| `impactedResourceId` | Yes      | `string`   | Fully qualified URI for the Azure resource.                                  |
| `PhysicalHostName`   | Yes      | `string`   | Node identifier, available in metadata.                                      |
| `LogUrl`             | No       | `string`   | URL to saved logs.                                                           |
| `Manufacturer`       | No       | `string`   | GPU manufacturer.                                                            |
| `SerialNumber`       | No       | `string`   | GPU serial number.                                                           |
| `ModelNumber`        | No       | `string`   | Model number.                                                                |
| `Location`           | No       | `string`   | Peripheral Component Interconnect Express (PCIe) location.                   |

> [!NOTE]
> Providing optional information can speed up the node recovery time. You can retrieve `PhysicalHostName` from within the VM by using [this script](https://github.com/jeseszhang1010/Utilities/blob/main/kvp_client.c).

Use the following command to get the `PhysicalHostName` value:

```shell
timeout 100 gcc -o /root/scripts/GPU/kvp_client /root/scripts/GPU/kvp_client.c
timeout 60 sudo /root/scripts/GPU/kvp_client | grep "PhysicalHostName;" | awk '{print$4}' | tee PhysicalHostName.txt
```

## HTTP response status codes for impact creation

When you submit a Guest Health Reporting impact, the REST API response indicates whether the request was accepted, not whether a repair action will occur.

| HTTP Status | Code | Description |
|---|---|---|
| 200 | OK | The impact submission was accepted. This status doesn't guarantee that Azure performs a repair action. |
| 4xx | Code can vary depending on the failure | The impact failed validation. Check the response `message` field to confirm the reason for the failure. |
| 404 | NotFound | The impacted resource wasn't found. Verify the resource provided as part of `additionalProperties` exists. |
| 429 | TooManyRequests | The request was rate limited. Retry later by using exponential backoff. |

> [!NOTE]
> A successful response suggests that the API received and processed the request. To track downstream evaluation and actions, continue to [query workload impact insights](#list-insights-for-a-workload-impact).

## Additional HPC properties

To aid Guest Health Reporting in taking the correct action, you can provide more information about the issue by using the `additionalProperties` field for high-performance computing (HPC).

### Resource HPC

`Resource.Hpc.*` fields:

* `LogUrl` (string): URL to the relevant log file.
* `PhysicalHostName` (string): Physical host name of the node (REQUIRED, alphanumeric).


`Resource.Hpc.Unhealthy.*` fields specific to GPUs:

* `Manufacturer` (string): Manufacturer of the GPU.
* `SerialNumber` (string): Serial number of the GPU.
* `ModelNumber` (string): Model number of the GPU.
* `Location` (string): Physical location of the GPU.

`Resource.Hpc.Investigate.*` field:

* `CollectTelemetry` (Boolean, `0`/`1`): Tell HPC to collect telemetry from the affected VM.

### GPU row remapping

`gpu_row_remap_failure` field:

* `SerialNumber` (string): Serial number of the GPU.
* Flag: `gpu_row_remap_failure: GPU # (SXM# SN:#): row remap failure. This is an official end of life condition: decommission the GPU`

`gpu_row_remap_*` fields:

* `UCE` (string): Count of uncorrectable errors in histogram data.
* `SerialNumber` (string): Serial number of the GPU.
* Flag: `gpu_row_remap_*: GPU # (SXM# SN:#): bank with multiple row remaps: partial 1, low 0, none 0. CE: 0, UCE: #`

> [!IMPORTANT]
> We advise you to include detailed row-remapping fields with the specified information in their claims to expedite node restoration.

## Query workload impact insights

After reporting a workload impact, Azure may generate a sequence of insights that describe how the event was detected, processed, acknowledged, and resolved. These insights can be queried programmatically through the Azure Resource Manager (ARM) API.

### List insights for a workload impact


To retrieve all insights for a specific workload impact, use the following REST API command:

```bash
GET "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Impact/workloadImpacts/{impactName}/insights?api-version=2025-01-01-preview"
```

#### Example Response

```json
{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22/insights/1454aa86bd412eca",
      "name": "1454aa86bd412eca",
      "type": "microsoft.impact/workloadimpacts/insights",
      "systemData": {
        "createdBy": "00000000-0000-0000-0000-000000000000",
        "createdByType": "Application",
        "createdAt": "2026-06-03T21:06:02.8556075Z",
        "lastModifiedBy": "00000000-0000-0000-0000-000000000000",
        "lastModifiedByType": "Application",
        "lastModifiedAt": "2026-06-03T21:06:02.8556075Z"
      },
      "properties": {
        "category": "MitigationAction",
        "status": "Acknowledged",
        "eventTime": "2026-06-03T21:05:59.65Z",
        "insightUniqueId": "573c176b-2d98-4e04-ad39-df5029d92036",
        "impact": {
          "impactedResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/dummy_rg/providers/Microsoft.Compute/virtualMachines/VM1",
          "startTime": "2026-06-02T01:06:21.3886467Z",
          "endTime": "0001-01-01T00:00:00Z",
          "impactId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22"
        },
        "additionalDetails": {
          "statusCode": "AcknowledgedUnhealthy",
          "terminalInsight": false
        },
        "content": {
          "title": "Customer reports investigate state - HPC Acknowledge",
          "description": "<h2><strong>Customer reports investigate state - HPC Acknowledge</strong></h2>\n<!--issueDescription-->\n<p>The Azure monitoring and diagnostics systems identified that your resource <strong></strong> may have been impacted by <strong></strong>; a <strong></strong> platform change.</p>\n<!--/issueDescription-->\n<!--rcaDescription-->\n<h3><strong>Root Cause</strong></h3>\n<blockquote>\n<p>The Host Node where the resource was running, or part of the resource dependency stack may have been impacted by this change, resulting in a brief impact.</p>\n</blockquote>\n<!--fb6919eb-a1ab-4bab-ab90-0e5596197e22-->\n<!--resolutionDetails-->\n<h3><strong>Resolution</strong></h3>\n<blockquote>\n<p>This state was transient.</p>\n</blockquote>\n<!--/resolutionDetails-->\n<!--additionalInfo-->\n<h3><strong>Additional Information</strong></h3>\n<blockquote>\n<p>Customers can control distribution of their resources at risk of effects from an infrastructure failure. For more details and example for VMs, see: <a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability\">Availability Options for Azure Virtual Machines</a>.</p>\n</blockquote>\n<blockquote>\n<p>See also, <a href=\"https://docs.microsoft.com/azure/service-health/alerts-activity-log-service-notifications-portal\">Create activity log alerts on service notifications using the Azure portal</a>.</p>\n</blockquote>\n<!--/additionalInfo-->\n<!--/rcaDescription-->\n<!--recommendedActions-->\n<h3><strong>Recommended Documents</strong></h3>\n<blockquote>\n<ul>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/maintenance-and-updates\">Maintenance for virtual machines in Azure</a></li>\n<li><a href=\"https://azure.microsoft.com/blog/service-healing-auto-recovery-of-virtual-machines\">Service Healing - Auto-recovery of Virtual Machines</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets\">Create and deploy virtual machines in an availability set using Azure PowerShell</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/storage/storage-managed-disks-overview\">Introduction to Azure managed disks</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/scheduled-events\">Azure Metadata Service: Scheduled Events for Windows VMs</a></li>\n</ul>\n</blockquote>\n<!--/recommendedActions-->\n<!--salutation-->\n<p>We apologize for any inconvenience this may have caused you.</p>\n<p>Microsoft Azure Team</p>\n<!--/salutation-->\n"
        },
        "provisioningState": "Succeeded"
      }
    },
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22/insights/53ec2fa0aafddc99",
      "name": "53ec2fa0aafddc99",
      "type": "microsoft.impact/workloadimpacts/insights",
      "systemData": {
        "createdBy": "00000000-0000-0000-0000-000000000000",
        "createdByType": "Application",
        "createdAt": "2026-06-03T21:06:04.907233Z",
        "lastModifiedBy": "00000000-0000-0000-0000-000000000000",
        "lastModifiedByType": "Application",
        "lastModifiedAt": "2026-06-03T21:06:04.907233Z"
      },
      "properties": {
        "category": "MitigationAction",
        "status": "Processing",
        "eventTime": "2026-06-03T21:06:00.693Z",
        "insightUniqueId": "9333c97f-a943-437d-b11e-e42a81bd8eab",
        "impact": {
          "impactedResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/dummy_rg/providers/Microsoft.Compute/virtualMachines/VM1",
          "startTime": "2026-06-02T01:06:21.3886467Z",
          "endTime": "0001-01-01T00:00:00Z",
          "impactId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22"
        },
        "additionalDetails": {
          "statusCode": "UpdatingNodeHealthState",
          "terminalInsight": false
        },
        "content": {
          "title": "Customer reports unhealthy state - HPC Unhealthy Pipeline Starts",
          "description": "<h2><strong>Customer reports unhealthy state - HPC Unhealthy Pipeline Starts</strong></h2>\n<!--issueDescription-->\n<p>The Azure monitoring and diagnostics systems identified that your resource <strong></strong> may have been impacted by <strong></strong>; a <strong></strong> platform change.</p>\n<!--/issueDescription-->\n<!--rcaDescription-->\n<h3><strong>Root Cause</strong></h3>\n<blockquote>\n<p>The Host Node where the resource was running, or part of the resource dependency stack may have been impacted by this change, resulting in a brief impact.</p>\n</blockquote>\n<!--ba951ba4-b416-44f5-b5e1-df7eed2670cd-->\n<!--resolutionDetails-->\n<h3><strong>Resolution</strong></h3>\n<blockquote>\n<p>This state was transient.</p>\n</blockquote>\n<!--/resolutionDetails-->\n<!--additionalInfo-->\n<h3><strong>Additional Information</strong></h3>\n<blockquote>\n<p>Customers can control distribution of their resources at risk of effects from an infrastructure failure. For more details and example for VMs, see: <a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability\">Availability Options for Azure Virtual Machines</a>.</p>\n</blockquote>\n<blockquote>\n<p>See also, <a href=\"https://docs.microsoft.com/azure/service-health/alerts-activity-log-service-notifications-portal\">Create activity log alerts on service notifications using the Azure portal</a>.</p>\n</blockquote>\n<!--/additionalInfo-->\n<!--/rcaDescription-->\n<!--recommendedActions-->\n<h3><strong>Recommended Documents</strong></h3>\n<blockquote>\n<ul>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/maintenance-and-updates\">Maintenance for virtual machines in Azure</a></li>\n<li><a href=\"https://azure.microsoft.com/blog/service-healing-auto-recovery-of-virtual-machines\">Service Healing - Auto-recovery of Virtual Machines</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets\">Create and deploy virtual machines in an availability set using Azure PowerShell</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/storage/storage-managed-disks-overview\">Introduction to Azure managed disks</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/scheduled-events\">Azure Metadata Service: Scheduled Events for Windows VMs</a></li>\n</ul>\n</blockquote>\n<!--/recommendedActions-->\n<!--salutation-->\n<p>We apologize for any inconvenience this may have caused you.</p>\n<p>Microsoft Azure Team</p>\n<!--/salutation-->\n"
        },
        "provisioningState": "Succeeded"
      }
    },
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22/insights/64ec403cfebdd3da",
      "name": "64ec403cfebdd3da",
      "type": "microsoft.impact/workloadimpacts/insights",
      "systemData": {
        "createdBy": "00000000-0000-0000-0000-000000000000",
        "createdByType": "Application",
        "createdAt": "2026-06-03T21:36:00.3626409Z",
        "lastModifiedBy": "00000000-0000-0000-0000-000000000000",
        "lastModifiedByType": "Application",
        "lastModifiedAt": "2026-06-03T21:36:00.3626409Z"
      },
      "properties": {
        "category": "MitigationAction",
        "status": "Resolved",
        "eventTime": "2026-06-03T21:35:56.923Z",
        "insightUniqueId": "9899ba08-d6f5-4bf5-88ea-b3ed8d3d0cff",
        "impact": {
          "impactedResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/dummy_rg/providers/Microsoft.Compute/virtualMachines/VM1",
          "startTime": "2026-06-02T01:06:21.3886467Z",
          "endTime": "0001-01-01T00:00:00Z",
          "impactId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/myImpact22"
        },
        "additionalDetails": {
          "statusCode": "NodeRemovedFromService",
          "terminalInsight": true
        },
        "content": {
          "title": "Customer reports unhealthy state - HPC Unhealthy Pipeline Successful",
          "description": "<h2><strong>Customer reports unhealthy state - HPC Unhealthy Pipeline Successful</strong></h2>\n<!--issueDescription-->\n<p>The Azure monitoring and diagnostics systems identified that your resource <strong></strong> may have been impacted by <strong></strong>; a <strong></strong> platform change.</p>\n<!--/issueDescription-->\n<!--rcaDescription-->\n<h3><strong>Root Cause</strong></h3>\n<blockquote>\n<p>The Host Node where the resource was running, or part of the resource dependency stack may have been impacted by this change, resulting in a brief impact.</p>\n</blockquote>\n<!--5f7cb092-ecd1-4c3e-89de-59e99a4ae137-->\n<!--resolutionDetails-->\n<h3><strong>Resolution</strong></h3>\n<blockquote>\n<p>This state was transient.</p>\n</blockquote>\n<!--/resolutionDetails-->\n<!--additionalInfo-->\n<h3><strong>Additional Information</strong></h3>\n<blockquote>\n<p>Customers can control distribution of their resources at risk of effects from an infrastructure failure. For more details and example for VMs, see: <a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability\">Availability Options for Azure Virtual Machines</a>.</p>\n</blockquote>\n<blockquote>\n<p>See also, <a href=\"https://docs.microsoft.com/azure/service-health/alerts-activity-log-service-notifications-portal\">Create activity log alerts on service notifications using the Azure portal</a>.</p>\n</blockquote>\n<!--/additionalInfo-->\n<!--/rcaDescription-->\n<!--recommendedActions-->\n<h3><strong>Recommended Documents</strong></h3>\n<blockquote>\n<ul>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/maintenance-and-updates\">Maintenance for virtual machines in Azure</a></li>\n<li><a href=\"https://azure.microsoft.com/blog/service-healing-auto-recovery-of-virtual-machines\">Service Healing - Auto-recovery of Virtual Machines</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets\">Create and deploy virtual machines in an availability set using Azure PowerShell</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/storage/storage-managed-disks-overview\">Introduction to Azure managed disks</a></li>\n<li><a href=\"https://docs.microsoft.com/azure/virtual-machines/windows/scheduled-events\">Azure Metadata Service: Scheduled Events for Windows VMs</a></li>\n</ul>\n</blockquote>\n<!--/recommendedActions-->\n<!--salutation-->\n<p>We apologize for any inconvenience this may have caused you.</p>\n<p>Microsoft Azure Team</p>\n<!--/salutation-->\n"
        },
        "provisioningState": "Succeeded"
      }
    }
  ]
}
```

| Name                | Type                  | Description                                                                                             |
|---------------------|-----------------------|---------------------------------------------------------------------------------------------------------|
| `additionalDetails` | object                | Additional details of the insight.                                                                      |
| `category`          | string                | Category of the insight.                                                                                |
| `content`           | object                | Contains title and description for the insight.                                                         |
| `eventId`           | string                | Identifier of the event correlated with this insight. Used to aggregate insights for the same event.    |
| `eventTime`         | string (date-time)    | Time of the event correlated with the impact.                                                           |
| `groupId`           | string                | Identifier that can be used to group similar insights.                                                  |
| `impact`            | object                | Details of the impact for which the insight has been generated.                                         |
| `insightUniqueId`   | string                | Unique identifier of the insight.                                                                       |
| `provisioningState` | string                | Resource provisioning state.                                                                            |
| `status`            | string                | Status of the insight (e.g., *Resolved*, *Repaired*, other).                                            |

### Additional Processing Fields

Some insights include extra processing metadata under `additionalDetails`. These fields help you understand how the impact request progressed through the Guest Health Reporting pipeline.

| Name                                | Type    | Description                                                                                                                                         |
|-------------------------------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `additionalDetails.statusCode`      | string  | Detailed reason code explaining why this insight was generated (for example: `AcknowledgedUnhealthy`, `NodeRemovedFromService`, `TooManyRequests`). |
| `additionalDetails.terminalInsight` | boolean | Indicates whether this is the final insight for the impact. If `true`, no further updates will follow.                                              |

These fields should be interpreted together:  
- **`statusCode`** = tells you the specific condition or reason for the insight. 
- **`terminalInsight`** = whether the pipeline has completed  

Example:  
`statusCode = "NodeRemovedFromService"` and `terminalInsight = true` tells you whether additional updates should be expected.

Example:  
 `statusCode = AcknowledgedUnhealthy`, `terminalInsight = false`  
→ The node health update is still in progress.

## Related content

* [What is Guest Health Reporting?](guest-health-overview.md)
* [Impact categories for Guest Health Reporting](guest-health-impact-categories.md)

