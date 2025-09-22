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

## HPC additional properties

To aid Guest Health Reporting in taking the correct action, you can provide more information about the issue by using the `additionalProperties` field for high-performance computing (HPC).

Use these `Resource.Hpc.*` fields:

* `LogUrl` (string): URL to the relevant log file.
* `PhysicalHostName` (string): Physical host name of the node (alphanumeric).
* `VmUniqueId` (string):  Unique ID of the VM (GUID).

> [!IMPORTANT]
> All HPC impact requests must include either `PhysicalHostName` (preferred) or `VmUniqueId`. The VM in question can be from any subscription. It isn't limited to the VMs in the subscription that you're reporting from.

These `Resource.Hpc.Unhealthy.*` fields are specific to GPUs:

* `Manufacturer` (string): Manufacturer of the GPU.
* `SerialNumber` (string): Serial number of the GPU.
* `ModelNumber` (string): Model number of the GPU.
* `Location` (string): Physical location of the GPU.

Use this `Resource.Hpc.Investigate.*` field:

* `CollectTelemetry` (Boolean, `0`/`1`): Tell HPC to collect telemetry from the affected VM.

Use this `gpu_row_remap_failure` field:

* `SerialNumber` (string): Serial number of the GPU.

The following flag relates to `gpu_row_remap_failure`:

`gpu_row_remap_failure: GPU # (SXM# SN:#): row remap failure. This is an official end of life condition: decommission the GPU`

Use these `gpu_row_remap_*` fields:

* `UCE` (string): Count of uncorrectable errors in histogram data.
* `SerialNumber` (string): Serial number of the GPU.

The following flag relates to `gpu_row_remap_*`:

`gpu_row_remap_*: GPU # (SXM# SN:#): bank with multiple row remaps: partial 1, low 0, none 0. CE: 0, UCE: #`

> [!IMPORTANT]
> We advise you to include detailed row-remapping fields with the specified information in their claims to expedite node restoration.

## Related content

* [What is Guest Health Reporting?](guest-health-overview.md)
* [Guest Health Reporting impact categories](guest-health-impact-categories.md)
