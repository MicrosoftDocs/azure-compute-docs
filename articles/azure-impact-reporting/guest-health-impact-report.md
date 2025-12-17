---
title: Azure HPC Guest Health Reporting - Report Node Health 
description: Share the health status of a supercomputing virtual machine with Azure. 
author: rolandnyamo 
ms.author: ronyamo 
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
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Impact/workloadImpacts/{workloadImpactName}?api-version=2023-02-01-preview
```

Descriptions of URI parameters are as follows:

| Field name       | Description       |
|---------------------|--------------------|
| `subscriptionId`  | Subscription previously added to an allow list. |
| `subscriptionId`   | Unique name that identifies a specific impact. You can also use a globally unique identifier (GUID).  |
| `api-version`   | API version to be used for this operation. Use `2023-02-01-preview`.   |

### [Healthy node](#tab/healthy/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Healthy",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "PhysicalHostName": "GGBB90904476",
      }
   }
}

```

### [Missing GPU](#tab/missingGPU/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.HpcMissingGpu",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "PhysicalHostName": "GGBB90904476",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j", //GUID
            "Manufacturer": "Nvidia",
            "SerialNumber": "12345679",
            "ModelNumber": "NV3LB225",
            "Location": "0"
      }
   }
}

```

### [Investigate node](#tab/investigate/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Investigate.NVLink",
      "impactDescription": "NvLink may be down",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j", //GUID
            "CollectTelemtery": "0"
      }
   }
}

```

### [Unhealthy non-GPU](#tab/unhealthynongpu/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.IBPerformance",
      "impactDescription": "IB low bandwidth",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "PhysicalHostName": "GGBB90904476",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j"
      }
   }
}

```

---

| Field name       | Required | Data type | Description                                                                 |
|-----------------------|--------------|---------------|---------------------------------------------------------------------------------|
| `startDateTime`         | Yes            | `datetime`      | Time (in UTC) when the impact happened.                                           |
| `impactCategory`        | Yes            | `string`        | Observation type or fault scenario. Only an approved string list is allowed.           |
| `impactDescription`     | Yes            | `string`        | Description of the reported impact.                                            |
| `impactedResourceId`    | Yes            | `string`        | Fully qualified URI for the Azure resource.                             |
| `physicalHostName`      | Yes            | `string`        | Node identifier, available in metadata.                                        |
| `VmUniqueId`            | Yes            | `string`        | Unique ID of the VM. Queryable inside the VM.                                |
| `logUrl`                | No           | `string`        | URL to saved logs.                                                             |
| `manufacturer`          | No           | `string`        | GPU manufacturer.                                                              |
| `serialNumber`          | No           | `string`        | GPU serial number.                                                             |
| `modelNumber`           | No           | `string`        | Model number.                                                                  |
| `location`              | No           | `string`        | Peripheral Component Interconnect Express (PCIe) location.                                                                 |

> [!NOTE]
> Providing optional information can speed up the node recovery time. You can retrieve `PhysicalHostName` from within the VM by using [this script](https://github.com/jeseszhang1010/Utilities/blob/main/kvp_client.c).

Use the following command to get the `PhysicalHostName` value:

```shell
timeout 100 gcc -o /root/scripts/GPU/kvp_client /root/scripts/GPU/kvp_client.c
timeout 60 sudo /root/scripts/GPU/kvp_client | grep "PhysicalHostName;" | awk '{print$4}' | tee PhysicalHostName.txt
```

## Additional HPC properties

To aid Guest Health Reporting in taking the correct action, you can provide more information about the issue by using the `additionalProperties` field for high-performance computing (HPC).

### Resource HPC

`Resource.Hpc.*` fields:

* `LogUrl` (string): URL to the relevant log file.
* `PhysicalHostName` (string): Physical host name of the node (alphanumeric).
* `VmUniqueId` (string):  Unique ID of the VM (GUID).

> [!IMPORTANT]
> All HPC impact requests must include either `PhysicalHostName` (preferred) or `VmUniqueId`. The VM in question can be from any subscription. It isn't limited to the VMs in the subscription that you're reporting from.

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
GET "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Impact/workloadImpacts/{impactId}/insights?api-version=2025-01-01-preview"
```

#### Example Response

```json
{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/impactid22/insights/insightId12",
      "name": "insightId12",
      "properties": {
        "additionalDetails": {
          "statusCode": "AcknowledgedUnhealthy",
          "terminalInsight": false
        },
        "category": "MitigationAction",
        "content": {
          "description": "Your resource experienced a brief, transient impact due to a platform change on the host node or its dependencies. No further action is required, though customers may reduce exposure by using Azure availability features and service notifications.",
          "title": "Customer reports investigate state - HPC Acknowledge"
        },
        "eventTime": "2025-06-15T04:00:00.009223Z",
        "impact": {
          "endTime": "0001-01-01T00:00:00Z",
          "impactId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/impactid22",
          "impactedResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AA_RG/providers/Microsoft.Compute/virtualMachineScaleSets/AA_VMSS/00001111",
          "startTime": "2025-06-15T17:36:21Z"
        },
        "insightUniqueId": "000111222-2233-4455-6677-66778897c74cb",
        "provisioningState": "Succeeded",
        "status": "Resolved"
      },
      "systemData": {
        "createdAt": "2025-06-15T04:00:00.009223Z",
        "createdBy": "000111222-2233-4455-6677-66778897c74",
        "createdByType": "Application",
        "lastModifiedAt": "2025-07-15T17:50:16.7183274Z",
        "lastModifiedBy": "000111222-2233-4455-6677-66778897c74",
        "lastModifiedByType": "Application"
      },
      "type": "microsoft.impact/workloadimpacts/insights"
    },
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/impactid22/insights/insightId13",
      "name": "insightId13",
      "properties": {
        "additionalDetails": {
          "statusCode": "NodeRemovedFromService",
          "terminalInsight": true
        },
        "category": "MitigationAction",
        "content": {
          "description": "Azure monitoring detected that your resource entered an unhealthy state due to a platform event affecting the host node or its dependency stack. The unhealthy pipeline completed successfully and the node was removed from service for repair or investigation. Availability options and service notifications can help reduce exposure to similar infrastructure events."
        },
        "eventTime": "2025-07-15T04:00:00.009223Z",
        "impact": {
          "endTime": "0001-01-01T00:00:00Z",
          "impactId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Impact/workloadImpacts/impactid22",
          "impactedResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AA_RG/providers/Microsoft.Compute/virtualMachineScaleSets/AA_VMSS/00001111",
          "startTime": "2025-07-15T17:36:21Z"
        },
        "insightUniqueId": "000111222-2233-4455-6677-66778897c74cb",
        "provisioningState": "Succeeded",
        "status": "Resolved"
      },
      "systemData": {
        "createdAt": "2025-07-15T04:00:00.009223Z",
        "createdBy": "000111222-2233-4455-6677-66778897c74",
        "createdByType": "Application",
        "lastModifiedAt": "2025-07-15T17:50:16.7183274Z",
        "lastModifiedBy": "000111222-2233-4455-6677-66778897c74",
        "lastModifiedByType": "Application"
      },
      "type": "microsoft.impact/workloadimpacts/insights"
    }
  ]
}
```

| Name               | Type                  | Description                                                                                             |
|--------------------|-----------------------|---------------------------------------------------------------------------------------------------------|
| `additionalDetails` | object                | Additional details of the insight.                                                                      |
| `category`         | string                | Category of the insight.                                                                               |
| `content`          | object             | Contains title and description for the insight.                                                        |
| `eventId`          | string                | Identifier of the event correlated with this insight. Used to aggregate insights for the same event.   |
| `eventTime`        | string (date-time)    | Time of the event correlated with the impact.                                                           |
| `groupId`          | string                | Identifier that can be used to group similar insights.                                                  |
| `impact`           | object       | Details of the impact for which the insight has been generated.                                         |
| `insightUniqueId`  | string                | Unique identifier of the insight.                                                                       |
| `provisioningState`| string   | Resource provisioning state.                                                                            |
| `status`           | string                | Status of the insight (e.g., *Resolved*, *Repaired*, other).                                            |
### Additional Processing Fields

Some insights include extra processing metadata under `additionalDetails`. These fields help you understand how the impact request progressed through the Guest Health Reporting pipeline.

| Name | Type | Description |
|------|------|-------------|
 `additionalDetails.statusCode` | string | Detailed reason code explaining why this insight was generated (for example: `AcknowledgedUnhealthy`, `NodeRemovedFromService`, `TooManyRequests`). |
| `additionalDetails.terminalInsight` | boolean | Indicates whether this is the final insight for the impact. If `true`, no further updates will follow. |

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

