---
title: Use compute placement scores to plan VM deployments
description: Learn how to use compute placement scores to evaluate the likelihood of allocating regular VMs across VM sizes, Azure regions, and availability zones.
author: bartoszp
ms.author: bartoszp
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 10/30/2026
ms.update-cycle: 180-days
ms.custom: devx-track-azurecli
ms.reviewer: wwilliams
ai-usage: ai-generated
# Customer intent: As a cloud architect, I want to compare allocation likelihood across VM sizes, regions, and availability zones so that I can choose a suitable configuration for my regular VM deployment.
---

# Use compute placement scores to plan VM deployments

Compute placement scores help you evaluate the likelihood of successfully allocating a requested number of regular, on-demand virtual machines (VMs). Specify candidate VM sizes, Azure regions, and whether to evaluate regional or zonal deployments. The API returns a score for each evaluated VM size and location combination.

Scores indicate a **High**, **Medium**, or **Low** likelihood of successful allocation, or identify a restriction, quota issue, or unavailable scoring data. Each result also includes a `placementPreference` from 0 to 9, where a higher value is better.

> [!IMPORTANT]
> **Access requirements**
>
> This API is generally available (GA) and supported for production use. However, access is restricted to approved customers.
>
> To request access, contact your Microsoft account representative. Microsoft reviews access requests and enables the API for approved Azure subscriptions.
>
> Access to this API isn't enabled by default.

This article uses API version `2026-10-05-preview`.

## When to use compute placement scores

Use the API when you want to:

- Compare allocation likelihood across VM sizes that support your workload.
- Evaluate regions or availability zones before you deploy regular VMs.
- Identify quota or VM size restrictions that affect your deployment options.

Compute placement scores evaluate each VM size and location combination independently. They don't recommend how to split a workload across multiple VM sizes or zones. For a combined distribution of capacity, use [SKU mix placement scores](sku-mix-placement-score.md). For Spot VM deployment planning, use [Spot Placement Score](../virtual-machine-scale-sets/spot-placement-score.md).

## Prerequisites

- An Azure subscription and an identity with permission to generate compute recommendations in that subscription. For information about assigning access, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).
- Candidate VM sizes and regions that meet your workload requirements.
- If you use the Azure CLI example, [install Azure CLI](/cli/azure/install-azure-cli) or use [Azure Cloud Shell](/azure/cloud-shell/overview).

## Considerations

-  The API uses subscription-specific data, including available quota and VM size restrictions. Always set `subscriptionId` in the request URL to the subscription where you plan to deploy. Results generated for a different subscription might not reflect your deployment conditions.
-  Placement scores are point-in-time assessments. Use them as soon as possible after the API call. The API aims to help you identify configurations with a higher likelihood of deployment success, but capacity and quota conditions can change at any time. The API doesn't deploy VMs or reserve capacity, and a high score doesn't guarantee that a subsequent deployment succeeds.
- Use the same subscription, VM size, instance count, region, and zonal or regional scope when you deploy. A score doesn't validate your complete deployment configuration.
- `desiredCount` is evaluated independently for each VM size and region or zone. It isn't a total that's divided across the returned results.
- Review each result's `score`, not just the HTTP status. A successful API call can return results that indicate insufficient quota, a restricted VM size, or unavailable scoring data.
- Don't interpret `placementPreference` as a probability or a percentage. It's separate from the `High`, `Medium`, and `Low` score categories.

## Configure your request

Specify the candidate VM sizes and deployment scope in the request body. For the complete schema, see the [compute placement scores REST API reference](/rest/api/recommenderrp/compute-placement-scores/post?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true).

| Property | Required | Description |
| --- | --- | --- |
| `desiredSizes` | Yes | Array of VM size objects. Set each object's `sku` to a VM size name, such as `Standard_D2_v5`. |
| `desiredLocations` | No | Array of Azure region names, such as `["eastus", "eastus2"]`. If omitted, the region in the request URL is used. Each entry must be a valid Azure region. |
| `desiredCount` | No | Number of VM instances to evaluate per region or zone, depending on the scope. Defaults to 1 if omitted. |
| `availabilityZones` | No | Set to `true` for zonal scores or `false` for regional scores. Specify this property explicitly to select your intended scope. |

For example, with `desiredCount` set to 10 and `availabilityZones` set to `true`, each zonal result evaluates allocating 10 VMs of that size in that zone. It doesn't evaluate dividing 10 VMs across all zones.

## Generate placement scores

Send an authenticated `POST` request to the following Azure Resource Manager endpoint. Replace `{subscriptionId}` with your subscription ID and `{location}` with an Azure region.

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/locations/{location}/placementScores/regular/generate?api-version=2026-10-05-preview
Content-Type: application/json
```

The `regular` resource selects scoring for regular VMs. Use `desiredLocations` in the body to evaluate regions other than, or in addition to, the region in the URL.

### Example request

The following request evaluates allocating 10 regular VMs of size `Standard_D2_v5` in each availability zone returned for East US and East US 2.

```json
{
  "availabilityZones": true,
  "desiredCount": 10,
  "desiredLocations": ["eastus", "eastus2"],
  "desiredSizes": [
    { "sku": "Standard_D2_v5" }
  ]
}
```

To call the API with Azure CLI:

1. Save the example request body in a file named `request.json`.

1. Sign in and select the target subscription.

    ```azurecli-interactive
    az login
    az account set --subscription "{subscriptionId}"
    ```

1. Submit the request. Replace the subscription ID with your value.

    ```azurecli-interactive
    az rest --method post --url "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/locations/eastus/placementScores/regular/generate?api-version=2026-10-05-preview" --body "@request.json"
    ```

### Example response

The following illustrative response contains six zonal results. Five indicate a high likelihood of allocation. One indicates that scoring data isn't available or is stale. Actual scores and placement preferences vary.

```json
{
  "availabilityZones": true,
  "desiredCount": 10,
  "desiredLocations": ["eastus", "eastus2"],
  "desiredSizes": [
    { "sku": "Standard_D2_v5" }
  ],
  "placementScores": [
    {
      "availabilityZone": "1",
      "placementPreference": 9,
      "region": "eastus",
      "score": "High",
      "sku": "Standard_D2_v5"
    },
    {
      "availabilityZone": "2",
      "placementPreference": 8,
      "region": "eastus",
      "score": "High",
      "sku": "Standard_D2_v5"
    },
    {
      "availabilityZone": "3",
      "placementPreference": 7,
      "region": "eastus",
      "score": "High",
      "sku": "Standard_D2_v5"
    },
    {
      "availabilityZone": "1",
      "placementPreference": 0,
      "region": "eastus2",
      "score": "DataNotFoundOrStale",
      "sku": "Standard_D2_v5"
    },
    {
      "availabilityZone": "2",
      "placementPreference": 7,
      "region": "eastus2",
      "score": "High",
      "sku": "Standard_D2_v5"
    },
    {
      "availabilityZone": "3",
      "placementPreference": 7,
      "region": "eastus2",
      "score": "High",
      "sku": "Standard_D2_v5"
    }
  ]
}
```

## Interpret the response

The response includes the evaluated request configuration and a `placementScores` array. Use the configuration fields to confirm the instance count, sizes, regions, and scope associated with the results.

Each entry in `placementScores` contains these fields:

| Field | Description |
| --- | --- |
| `sku` | VM size evaluated for this result. |
| `region` | Azure region evaluated for this result. |
| `availabilityZone` | Availability zone associated with a zonal result. |
| `score` | Allocation likelihood category or a value identifying why a normal likelihood score isn't available. |
| `placementPreference` | Integer from 0 to 10, inclusive. Higher is better. |

### Score values

The API defines the following `score` values:

| Value | Meaning | Action |
| --- | --- | --- |
| `High` | High likelihood of successful allocation. | Consider this configuration while accounting for your workload requirements and the point-in-time nature of the score. |
| `Medium` | Medium likelihood of successful allocation. | Compare other sizes, regions, or zones before you deploy. |
| `Low` | Low likelihood of successful allocation. | Consider another configuration or evaluate a smaller instance count. |
| `RestrictedSkuNotAvailable` | The requested VM size is restricted and unavailable. | Check VM size availability for your subscription and target location, or choose another size or location. |
| `InsufficientQuota` | Available quota is insufficient to fulfill the request. | Review your [VM quotas](quotas.md), request an increase if needed, or reduce the requested count. |
| `DataNotFoundOrStale` | Scoring data wasn't found or isn't sufficiently current. | Don't interpret this value as a low allocation likelihood. Retry later or evaluate another configuration. |

A `placementPreference` of 10 isn't a guarantee of success, and a value of 0 doesn't replace the explanation in `score`. Review both fields. The numeric preference isn't the same as the 0-to-9 score returned by the SKU mix placement scores API. Score refers to an *absolute* probability of allocation success. PlacementPreference is a relative preference helping you distinguish between otherwise equivalent options. 

## Evaluate a regional deployment

For regional scoring, set `availabilityZones` to `false`. The following request also demonstrates the defaults for omitted fields: the API evaluates one VM in the region specified in the request URL.

```json
{
  "availabilityZones": false,
  "desiredSizes": [
    { "sku": "Standard_D2_v5" }
  ]
}
```

For example, when you submit this body to the `eastus` endpoint, the evaluated `desiredLocations` is `["eastus"]` and `desiredCount` is 1. Regional scores evaluate allocation at regional scope rather than within a particular availability zone.

## Get scoring metadata

Use the following `GET` request to retrieve metadata for the compute placement scoring resource:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/locations/{location}/placementScores/regular?api-version=2026-10-05-preview
```

The response includes `properties.supportedResourceTypes`. This operation returns metadata, not generated placement scores. Use the `POST` operation to evaluate your deployment configuration.

For details, see the [compute placement scores metadata REST API reference](/rest/api/recommenderrp/compute-placement-scores/get?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true).

## Troubleshoot requests

If the API returns an error, inspect `error.code`, `error.message`, and any `error.details` in the response.

| Issue | Action |
| --- | --- |
| Authentication or authorization failure. | Confirm that you're signed in to the correct tenant and that your identity has permission to generate recommendations in the target subscription. |
| Invalid request parameters. | Check the API version, region names, VM size names, and JSON property types. `desiredSizes` is an array of objects with a `sku` property, `desiredCount` is an integer, and `availabilityZones` is a boolean. |
| Successful call with a restriction, quota, or data issue. | Inspect each entry's `score` and follow the guidance in the score values table. HTTP `200 OK` means the scoring operation succeeded, not that every evaluated deployment can succeed. |
| Deployment fails after a high score. | Review the deployment error and request fresh scores. Capacity and quota conditions can change after the scoring call, and deployment constraints outside the scoring request can affect allocation. |

## Next steps

- Review the [compute placement scores REST API reference](/rest/api/recommenderrp/compute-placement-scores/post?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true).
- Learn how to distribute capacity with [SKU mix placement scores](sku-mix-placement-score.md).
- Review [VM quotas](quotas.md) before you deploy.
