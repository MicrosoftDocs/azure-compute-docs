---
title: Ignore metrics
description: An introduction to the mechanism of ignoring certain metrics in service fabric. 
author: tracygooo

ms.topic: concept-article
ms.date: 1/26/2026
ms.author: majovanovic
# Customer intent: As a cloud service administrator, I want to enable/disable a certain set of metrics quickly, without editing the service description.
---

# Ignore metrics
Service Fabric provides a way for customers to set custom metrics for their services, which the Cluster Resource Manager (CRM) uses for placement of said services and for keeping the cluster balanced. These metrics don't need to be physical cluster constraints, like CPU or memory, but can be any logical constraints that the customer wishes to impose upon the cluster.

CRM introduces a way where the user can ignore a set of these metrics by enabling the feature through a dynamic config. When this feature is enabled, CRM considers capacities for the stated metrics on all of the nodes infinite, so they don't pose any constraint in placing services. This feature might come in handy when a large portion of the cluster nodes goes down. In this scenario, capacities need to be expanded in order to prioritize placing all of the services from the downed nodes. An example of such a scenario is when a whole availability zone (AZ) goes down.

## Example scenario: Large-scale node failure requiring capacity expansion

This scenario demonstrates what happens when a large portion of cluster nodes goes down, and how ignoring metrics effectively expands capacity to prioritize placing all services from the downed nodes.

### Initial cluster state (six nodes healthy)

**Node capacities:**

| Node Name | CPU Capacity | Memory Capacity | Custom Metric Capacity |
|-----------|--------------|-----------------|------------------------|
| Node1 | 100 units | 8192 MB | 50 units |
| Node2 | 100 units | 8192 MB | 50 units |
| Node3 | 100 units | 8192 MB | 50 units |
| Node4 | 100 units | 8192 MB | 50 units |
| Node5 | 100 units | 8192 MB | 50 units |
| Node6 | 100 units | 8192 MB | 50 units |

**Service definitions:**

| Service Name | Total Replicas | CPU per Replica | Memory per Replica | Custom Metric per Replica |
|--------------|----------------|-----------------|---------------------|---------------------------|
| ServiceA | 6 replicas | 10 units | 1024 MB | 13 units |
| ServiceB | 6 replicas | 10 units | 1024 MB | 13 units |
| ServiceC | 6 replicas | 10 units | 1024 MB | 13 units |

**Initial placement (balanced across all 6 nodes):**

| Node | ServiceA | ServiceB | ServiceC | Total CPU | Total Memory | Total Custom Metric | Status |
|------|----------|----------|----------|-----------|--------------|---------------------|--------|
| Node1 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |
| Node2 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |
| Node3 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |
| Node4 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |
| Node5 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |
| Node6 | 1 replica | 1 replica | 1 replica | 30/100 | 3072/8192 MB | 39/50 units | ✓ Healthy |

All 18 replicas (6 from each service) are distributed evenly. Each node hosts 3 replicas.

### Large-scale failure: Four out of six nodes go down

**Catastrophic scenario:** Node3, Node4, Node5, and Node6 fail simultaneously (67% of the cluster is down). CRM must redistribute twelve replicas from the failed nodes to the remaining two healthy nodes.

**Remaining nodes after failure:**

| Node Name | CPU Capacity | Memory Capacity | Custom Metric Capacity | Status |
|-----------|--------------|-----------------|------------------------|--------|
| Node1 | 100 units | 8192 MB | 50 units | Active |
| Node2 | 100 units | 8192 MB | 50 units | Active |
| Node3 | — | — | — | ✗ Down |
| Node4 | — | — | — | ✗ Down |
| Node5 | — | — | — | ✗ Down |
| Node6 | — | — | — | ✗ Down |

**Attempting to place all services from downed nodes:**

CRM needs to place twelve replicas (four of each service) onto just two nodes:

| Node | ServiceA Replicas Needed | ServiceB Replicas Needed | ServiceC Replicas Needed | Required Custom Metric |
|------|--------------------------|--------------------------|--------------------------|------------------------|
| Node1 | 1 original + 2 from failed nodes | 1 original + 2 from failed nodes | 1 original + 2 from failed nodes | Nine replicas × 13 = 117 units |
| Node2 | 1 original + 2 from failed nodes | 1 original + 2 from failed nodes | 1 original + 2 from failed nodes | 9 replicas × 13 = 117 units |

**Actual placement result (constrained by Custom Metric):**

| Node | ServiceA | ServiceB | ServiceC | Total CPU | Total Memory | Total Custom Metric | Status |
|------|----------|----------|----------|-----------|--------------|---------------------|--------|
| Node1 | 2 replicas | 1 replica | 1 replica | 40/100 | 4096/8192 MB | 52/50 units | ✗ Blocked at capacity |
| Node2 | 2 replicas | 1 replica | 1 replica | 40/100 | 4096/8192 MB | 52/50 units | ✗ Blocked at capacity |

**Services from downed nodes - placement summary:**

| Service | Replicas on Downed Nodes | Successfully Redistributed | Still Pending | Status |
|---------|--------------------------|----------------------------|---------------|--------|
| ServiceA | 4 replicas | 2 replicas | 2 replicas unplaced | ⚠️ Partial recovery (50%) |
| ServiceB | 4 replicas | 0 replicas | 4 replicas unplaced | ✗ Failed to place any |
| ServiceC | 4 replicas | 0 replicas | 4 replicas unplaced | ✗ Failed to place any |
| **Total** | **12 replicas lost** | **2 placed** | **10 unplaced** | **83% placement failure** |

**Problem:** The Custom Metric capacity of 50 units per node is the bottleneck. You could manually edit each node description to disable the metric, but this approach becomes impractical with many nodes or multiple custom metrics. The ignoring metrics feature solves this problem with a single cluster-wide configuration change.

### Ignore the Custom Metric

By ignoring the Custom Metric, you effectively expand the capacity from 50 units to infinite, allowing CRM to prioritize placing all services from the downed nodes.

**Updated node capacities with ignored metric:**

| Node Name | CPU Capacity | Memory Capacity | Custom Metric Capacity | Status |
|-----------|--------------|-----------------|------------------------|--------|
| Node1 | 100 units | 8192 MB | ∞ (ignored) | Active |
| Node2 | 100 units | 8192 MB | ∞ (ignored) | Active |
| Node3 | — | — | — | ✗ Down |
| Node4 | — | — | — | ✗ Down |
| Node5 | — | — | — | ✗ Down |
| Node6 | — | — | — | ✗ Down |

**Successful placement of all services from downed nodes:**

| Node | ServiceA | ServiceB | ServiceC | Total CPU | Total Memory | Total Custom Metric | Status |
|------|----------|----------|----------|-----------|--------------|---------------------|--------|
| Node1 | 3 replicas | 3 replicas | 3 replicas | 90/100 | 9216/8192 MB | 117 units (ignored) | ✓ All replicas placed |
| Node2 | 3 replicas | 3 replicas | 3 replicas | 90/100 | 9216/8192 MB | 117 units (ignored) | ✓ All replicas placed |

**Services from downed nodes - recovery summary:**

| Service | Replicas on Downed Nodes | Successfully Redistributed | Still Pending | Status |
|---------|--------------------------|----------------------------|---------------|--------|
| ServiceA | 4 replicas | 4 replicas | 0 replicas | ✓ Full recovery (100%) |
| ServiceB | 4 replicas | 4 replicas | 0 replicas | ✓ Full recovery (100%) |
| ServiceC | 4 replicas | 4 replicas | 0 replicas | ✓ Full recovery (100%) |
| **Total** | **12 replicas lost** | **12 placed** | **0 unplaced** | **100% placement success** |

**Result:** By ignoring the Custom Metric, the capacity constraint was removed and all 12 replicas from the downed nodes were successfully placed on the 2 remaining nodes. This restored full service redundancy and ensured high availability during the cluster failure.

## 1. Specify metrics to ignore

Configure which metrics can be ignored as part of your cluster configuration. You must specify these metrics in advance before you need to ignore them. This configuration is a global cluster setting that identifies individual metrics that can be ignored. The following examples show how to configure metrics. In this case, Metric1 is ignored (node capacities for that metric become infinite), while Metric2 continues to be enforced normally.

> [!NOTE]
> Set the value to `true` only for metrics that should be ignored. Metrics not listed in the `ExpandedMetricsDuringZoneDownMode` section continue to be enforced normally.

ClusterManifest.xml:

```xml
<Section Name="ExpandedMetricsDuringZoneDownMode">
    <Parameter Name="Metric1" Value="true" />
    <Parameter Name="Metric2" Value="false" />
</Section>
```

Via ClusterConfig.json for Standalone deployments or Template.json for Azure hosted clusters:

```json
"fabricSettings": [
  {
    "name": "ExpandedMetricsDuringZoneDownMode",
    "parameters": [
      {
          "name": "Metric1",
          "value": "true"
      },
      {
          "name": "Metric2",
          "value": "false"
      }
    ]
  }
]
```

## 2. Enable/Disable the feature

Users must manually detect the scenario when metrics need to be ignored. Ignoring metrics feature is turned on/off by setting config `EnableZoneDownModeNodeCapacityExpansion` in `PlacementAndLoadBalancing` section of cluster manifest either using XML or JSON:

In ClusterManifest.xml:
``` xml
<Section Name="PlacementAndLoadBalancing">
     <Parameter Name="EnableZoneDownModeNodeCapacityExpansion" Value="true" />
</Section>
```

Via ClusterConfig.json for Standalone deployments or Template.json for Azure hosted clusters:

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "EnableZoneDownModeNodeCapacityExpansion",
          "value": "true"
      }
    ]
  }
]
```

Once `EnableZoneDownModeNodeCapacityExpansion` is set to `true`, metrics specified in the `ExpandedMetricsDuringZoneDownMode` section are ignored. Setting `EnableZoneDownModeNodeCapacityExpansion` to `false` turns off this feature, and all metrics are enforced normally.

## Next steps
Learn more about [custom metrics](service-fabric-cluster-resource-manager-metrics.md).
