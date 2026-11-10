---
title: Use SKU mix placement scores to plan VM deployments
description: Learn how to use SKU mix placement scores to distribute VM capacity across VM sizes and availability zones based on capacity, quota, and your preferences.
author: bartoszp
ms.author: bartoszp
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 10/27/2026
ms.update-cycle: 180-days
ms.custom: devx-track-azurecli
ms.reviewer: wwilliams
ai-usage: ai-generated
# Customer intent: As a cloud architect, I want recommendations for distributing capacity across VM sizes and availability zones so that I can plan deployments that meet my workload requirements.
---

# Use SKU mix placement scores to plan VM deployments

SKU mix placement scores help you plan deployments for workloads that can run on multiple virtual machine (VM) sizes. The API recommends how to distribute your requested capacity across VM sizes and, optionally, availability zones within a region. Recommendations account for available capacity, subscription restrictions, available quota, and the allocation preferences in your request.

Each deployment choice includes a placement score from 0 to 9, where a higher score is better, and a `skuSplit` that describes the recommended allocations. You can request recommendations for regular or [Azure Spot VMs](spot-vms.md).

> [!IMPORTANT]
> **Access requirements**
>
> This API is generally available (GA) and supported for production use. However, access is restricted to approved customers.
>
> To request access, contact your Microsoft account representative. Microsoft reviews access requests and enables the API for approved Azure subscriptions.
>
> Access to this API isn't enabled by default.

This article uses API version `2026-10-05-preview`.

## When to use SKU mix placement scores

Use the API when you want to:

- Find a distribution of capacity across several VM sizes that your workload supports.
- Compare deployment choices that favor lower prices, your preferred VM sizes, or lower Spot eviction rates.
- Plan a regional deployment or control how capacity is distributed across availability zones.
- Identify capacity or quota constraints before you deploy.

Unlike [Compute Placement Score](/rest/api/recommenderrp/compute-placement-scores?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true), which evaluates individual VM size and location combinations, SKU Mix Placement Scores return deployment choices that can combine multiple VM sizes and zones into a single recommendation.

## Prerequisites

- An Azure subscription and an identity with permission to generate compute recommendations in that subscription. For information about assigning access, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).
- A list of VM sizes that support your workload's operating system, architecture, storage, and security requirements.
- A target Azure region and, for zonal requests, the logical availability zones to consider.
- If you use the Azure CLI example, [install Azure CLI](/cli/azure/install-azure-cli) or use [Azure Cloud Shell](/azure/cloud-shell/overview).

## Considerations

-  The API uses subscription-specific data, including available quota and VM size restrictions. Always set `subscriptionId` in the request URL to the subscription where you plan to deploy. Results generated for a different subscription might not reflect your deployment conditions.
-  Recommendations are point-in-time assessments based on capacity and quota conditions when you call the API. Use them as soon as possible after the call. The API aims to improve the likelihood of deployment success while optimizing for the criteria in your request. However, capacity can change at any time, including before the `validUntil` timestamp. The API doesn't deploy VMs or reserve capacity, and recommendations don't guarantee deployment success.
- Use the same subscription, region, VM sizes, priority, and zone requirements when you deploy. A recommendation doesn't validate your complete deployment configuration.
- Inspect `partialFulfillmentReason` and the recommended allocations before you deploy. A successful API response doesn't necessarily mean that all requested capacity can be fulfilled.
- Request fresh recommendations after the response's `validUntil` timestamp. Don't reuse expired recommendations.
- A Spot recommendation doesn't guarantee that VMs won't be evicted.
- Each request targets one region. To evaluate another region, make a separate request with that region in the URL.

## Configure your request

Specify the candidate VM sizes in `instanceDescription`, the capacity requirements in `capacityProfile`, and optional availability zones in `zones`. For the complete schema, see the [SKU mix placement scores REST API reference](/rest/api/recommenderrp/sku-mix-placement-scores/post?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true).

### Capacity and VM sizes

Use these properties to describe your workload:

| Property | Description |
| --- | --- |
| `instanceDescription.vmSizes` | List of candidate VM sizes. Set each entry's `name` to a VM size name, such as `Standard_D4as_v5`. |
| `instanceDescription.vmSizes[].rank` | VM size preference for the `Prioritized` allocation strategy. Lower values indicate higher priority, starting at 0. Use this property only with `Prioritized`. |
| `capacityProfile.capacity` | Requested capacity: 1 to 1,000 VMs, or 1 to 10,000 vCPUs. |
| `capacityProfile.capacityType` | Unit for the requested capacity: `VM` or `VCpu`. |
| `capacityProfile.priority` | VM priority: `Regular` or `Spot`. |
| `capacityProfile.osType` | Operating system: `Linux` or `Windows`. Required with `LowestPrice` because pricing varies by operating system. |
| `capacityProfile.spotPriorityProfile.maxPricePerVm` | Maximum price per VM for a Spot request. The default is `-1`, which applies no price restriction. |
| `zones` | Logical availability zones to consider, such as `["1", "2", "3"]`. Omit this property or pass an empty array for a regional, nonzonal request. |

### VM size allocation strategy

Set `capacityProfile.allocationStrategy` to select an allocation preference:

| Strategy | Behavior |
| --- | --- |
| `LowestPrice` | Optimizes the VM size distribution for the lowest price. Specify `osType` with this strategy. |
| `Prioritized` | Uses the `rank` values on your candidate VM sizes. Lower ranks have higher priority. |
| `EvictionOptimized` | Optimizes for the lowest eviction rate. Use this strategy only with `Spot` priority. |

The strategy values in this table belong to the placement scores API. Don't substitute allocation strategy values from a scale set deployment API.

### Zone distribution strategy

For a zonal request, configure `capacityProfile.zoneAllocationPolicy`:

| `distributionStrategy` | Behavior |
| --- | --- |
| `BestEffortBalanced` | Distributes capacity across the requested zones on a best-effort balanced basis. This strategy is the default. |
| `Prioritized` | Fills higher-priority zones first, based on `zonePreferences`. |
| `BestEffortSingleZone` | Tries to place capacity in one zone, but can use multiple zones if a single zone has insufficient capacity. |

For `Prioritized` zone distribution, set `zonePreferences` entries with a `zone` and a `rank`. Lower ranks have higher priority, with 0 the highest. You can also specify `targetMaxCapacity`, a best-effort limit on the count allocated within that zone.

VM size priority and zone priority are separate settings. Use `instanceDescription.vmSizes[].rank` to rank VM sizes, and `capacityProfile.zoneAllocationPolicy.zonePreferences[].rank` to rank zones.

## Generate placement recommendations

Send an authenticated `POST` request to the following Azure Resource Manager endpoint. Replace `{subscriptionId}` and `{location}` with your subscription ID and target region.

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/locations/{location}/skuMixPlacementScores/recommendations/generate?api-version=2026-10-05-preview
Content-Type: application/json
```

### Example request

The following request asks for 10 regular VMs across two candidate sizes and three zones in the region specified in the URL. It favors the lowest price and best-effort balancing across zones.

```json
{
  "zones": ["1", "2", "3"],
  "capacityProfile": {
    "capacity": 10,
    "capacityType": "VM",
    "priority": "Regular",
    "allocationStrategy": "LowestPrice",
    "osType": "Linux",
    "zoneAllocationPolicy": {
      "distributionStrategy": "BestEffortBalanced"
    }
  },
  "instanceDescription": {
    "vmSizes": [
      { "name": "Standard_D4as_v5" },
      { "name": "Standard_D4s_v5" }
    ]
  }
}
```

To call the API with Azure CLI:

1. Save the example request body in a file named `request.json`.

1. Sign in and select the target subscription.

    ```azurecli-interactive
    az login
    az account set --subscription "{subscriptionId}"
    ```

1. Submit the request. This example uses `eastus`. Replace the subscription ID and region with your values.

    ```azurecli-interactive
    az rest --method post --url "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/locations/eastus/skuMixPlacementScores/recommendations/generate?api-version=2026-10-05-preview" --body "@request.json"
    ```

### Example response

The following illustrative response contains one deployment choice with a score of 8. Its allocations total 10 VMs: four in zone 1, three in zone 2, and three in zone 3. Actual recommendations, scores, limits, and expiration times vary.

```json
{
  "id": "8b0e5f2c-6d41-4a7e-9c3b-1f5a2d8e4b60",
  "placementChoices": [
    {
      "score": 8,
      "skuSplit": [
        {
          "name": "Standard_D2as_v5",
          "priority": "Regular",
          "capacity": 4,
          "zone": "1"
        },
        {
          "name": "Standard_D4as_v5",
          "priority": "Regular",
          "capacity": 3,
          "zone": "2"
        },
        {
          "name": "Standard_D4as_v5",
          "priority": "Regular",
          "capacity": 3,
          "zone": "3"
        }
      ]
    }
  ],
  "partialFulfillmentReason": "None",
  "capacityLimits": [
    {
      "name": "Standard_D4as_v5",
      "priority": "Regular",
      "zone": "1",
      "limit": 10,
      "reason": "None"
    },
    {
      "name": "Standard_D4as_v5",
      "priority": "Regular",
      "zone": "2",
      "limit": 10,
      "reason": "None"
    },
    {
      "name": "Standard_D4as_v5",
      "priority": "Regular",
      "zone": "3",
      "limit": 10,
      "reason": "None"
    },
    {
      "name": "Standard_D4s_v5",
      "priority": "Regular",
      "zone": "1",
      "limit": 6,
      "reason": "InsufficientCapacity"
    },
    {
      "name": "Standard_D4s_v5",
      "priority": "Regular",
      "zone": "2",
      "limit": 10,
      "reason": "None"
    },
    {
      "name": "Standard_D4s_v5",
      "priority": "Regular",
      "zone": "3",
      "limit": 4,
      "reason": "InsufficientQuota"
    }
  ],
  "validUntil": "2026-10-27T21:15:00Z"
}
```

## Interpret the response

Review these fields before you use a recommendation:

| Field | How to use it |
| --- | --- |
| `id` | Identifies the entire response, including a response with no deployment choices. It isn't a separate identifier for each choice. |
| `placementChoices` | Alternative deployment choices. Select a choice rather than combining allocations from different choices. |
| `placementChoices[].score` | Score from 0 to 9, where a higher score is better. Don't interpret the score as a percentage or an allocation guarantee. |
| `placementChoices[].skuSplit` | Allocations that make up a choice. Each entry identifies a VM size, priority, recommended capacity, and zone, if applicable. |
| `partialFulfillmentReason` | Indicates whether the requested capacity was fully fulfilled. Possible values are `None`, `InsufficientCapacity`, and `InsufficientQuota`. |
| `capacityLimits` | Independent availability limits for every requested VM size and zone combination, including combinations not used in the recommendations. |
| `validUntil` | Expiration timestamp. Request fresh recommendations after this time. |

### Capacity limits

Use `capacityLimits` to further optimize your deployment for additional criteria that aren't reflected in the recommended `skuSplit`.

For the best chance of deployment success, use a recommended `skuSplit` without modification whenever possible. The API optimizes each split to balance the likelihood of successful allocation with the criteria in your request.

Each `capacityLimits` entry contains a `limit` in VMs and a `reason`. A limit of 0 means no capacity is available for that combination. The limit accounts for the requested capacity, available capacity, and available quota, and isn't below the allocation already recommended for that combination.

These limits describe individual combinations independently. Don't add them together as a guarantee that you can allocate the combined total.

| Reason | Meaning | Action |
| --- | --- | --- |
| `None` | The full requested capacity is available for this combination. | Review the deployment choices and their scores. |
| `InsufficientCapacity` | Available capacity limits this combination. | Consider other candidate sizes, zones, or regions, or reduce the requested capacity. |
| `InsufficientQuota` | Available quota limits this combination. This reason takes precedence if both capacity and quota are insufficient. | Review your [VM quotas](quotas.md) and request an increase if needed. |
| `SkuNotAvailable` | The VM size isn't available in this zone for the subscription. | Choose a VM size and zone combination that's available to your subscription. |

In the example response, some individual combinations have insufficient capacity or quota, but the recommended split still fulfills all 10 VMs. The top-level `partialFulfillmentReason` is therefore `None`.

### Partial fulfillment

An HTTP `200 OK` response can contain recommendations that fulfill only part of your request. Check `partialFulfillmentReason` even when the API call succeeds.

If the value is `InsufficientCapacity` or `InsufficientQuota`, review the returned allocations and `capacityLimits` before deciding whether a smaller deployment meets your needs. Adjust the request or address quota constraints, and then request new recommendations. Don't treat an empty `placementChoices` array as a deployment plan.

## Other request configurations

Adapt the example request to match your workload:

- **Regional deployment:** Omit `zones` or set it to `[]`. Regional response entries omit `zone` or return an empty value.
- **vCPU target:** Set `capacityType` to `VCpu` and `capacity` to your requested vCPU count.
- **Preferred VM sizes:** Set `allocationStrategy` to `Prioritized` and add a `rank` to each VM size entry.
- **Spot deployment:** Set `priority` to `Spot`. Use `EvictionOptimized` if your goal is to favor lower eviction rates, and optionally configure `spotPriorityProfile.maxPricePerVm`.
- **Preferred zones:** Set `zoneAllocationPolicy.distributionStrategy` to `Prioritized` and provide `zonePreferences` with zone ranks and optional best-effort capacity limits.

For example, the following properties favor `Standard_D4as_v5` over `Standard_D4s_v5`, and zone 1 over zone 2:

```json
{
  "zones": ["1", "2"],
  "capacityProfile": {
    "capacity": 10,
    "capacityType": "VM",
    "priority": "Regular",
    "allocationStrategy": "Prioritized",
    "zoneAllocationPolicy": {
      "distributionStrategy": "Prioritized",
      "zonePreferences": [
        { "zone": "1", "rank": 0 },
        { "zone": "2", "rank": 1 }
      ]
    }
  },
  "instanceDescription": {
    "vmSizes": [
      { "name": "Standard_D4as_v5", "rank": 0 },
      { "name": "Standard_D4s_v5", "rank": 1 }
    ]
  }
}
```

## Troubleshoot requests

If the API returns an error, inspect `error.code`, `error.message`, and any `error.details` in the response.

| Issue | Action |
| --- | --- |
| Authentication or authorization failure. | Confirm that you're signed in to the correct tenant and that your identity has access to generate recommendations in the target subscription. |
| Invalid request parameters. | Check the API version, property names, enum casing, and capacity range against the REST API reference. Specify `osType` for `LowestPrice`, and use VM size ranks only with `Prioritized`. |
| Partial or empty recommendations. | Review `partialFulfillmentReason` and `capacityLimits`. Address quota constraints or try different sizes, zones, a smaller capacity, or another region. |
| Expired recommendations. | Generate a new response instead of deploying from a response whose `validUntil` timestamp has passed. |

## Next steps

- Review the [SKU mix placement scores REST API reference](/rest/api/recommenderrp/sku-mix-placement-scores/post?view=rest-recommenderrp-2026-10-05-preview&preserve-view=true).
- Learn about [instance mix in Virtual Machine Scale Sets](../virtual-machine-scale-sets/instance-mix-overview.md).
- Review [VM quotas](quotas.md) before you deploy.