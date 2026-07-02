---
title: Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules
description: Learn how to collect Service Fabric cluster telemetry by using Azure Monitor Agent (AMA) and data collection rules (DCRs), and migrate from deprecated WAD/LAD and Log Analytics agent configurations.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 06/08/2026
# Customer intent: As a Service Fabric operator, I want to configure supported telemetry collection for my cluster so that I can monitor platform events, syslog or Windows events, and performance data in Azure Monitor Logs.
---

# Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules

This article describes a supported telemetry approach for Service Fabric clusters by using Azure Monitor Agent (AMA) and data collection rules (DCRs).

Use this guidance if you're onboarding new monitoring, or if you're migrating from older diagnostics configurations such as Windows Azure Diagnostics (WAD), Linux Azure Diagnostics (LAD), or Log Analytics agent (also known as MMA/OMS).

## Why this approach

- WAD and LAD are deprecated and retire on March 31, 2026.
- Log Analytics agent (MMA/OMS) is deprecated and unsupported.
- AMA + DCR is the supported ingestion model for guest logs and performance data in Azure Monitor.

For Azure Monitor migration guidance, see [Migrate from Azure Diagnostics extension to Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-migration).

## Recommended rollout

Use a phased rollout to reduce risk.

1. Validate AMA + DCR pipeline.
1. Add Service Fabric specific signals.
1. Cut over and remove deprecated agents.

### Phase 1: Validate AMA + DCR pipeline

1. Install AMA on cluster VM scale sets or machines.
1. Create and associate at least one DCR. Data collection starts only after association.
1. Validate that telemetry is flowing into Azure Monitor Logs.

- Windows event logs into the `Event` table.
- Linux syslog into the `Syslog` table.
- Performance counters into the `Perf` table.

Key references:

- [Manage Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-manage)
- [Create and edit data collection rules](/azure/azure-monitor/data-collection/data-collection-rule-create-edit)
- [Data collection rule samples](/azure/azure-monitor/data-collection/data-collection-rule-samples)
- [Monitor data collection operations](/azure/azure-monitor/data-collection/data-collection-monitor)

### Phase 2: Add Service Fabric signals

Map your required Service Fabric signals and update DCR scope incrementally (for example, by node type canary).

- Windows clusters: collect relevant Service Fabric event channels from Windows Event Log.
- Linux clusters: collect syslog facilities and severities used for Service Fabric events.
- Add required OS and process performance counters for infrastructure monitoring.

Use [List of Service Fabric events](service-fabric-diagnostics-event-generation-operational.md) to determine event IDs and channels that are required for alerting and troubleshooting.

### Phase 3: Cut over and remove deprecated agents

After required signals are available through AMA + DCR:

1. Remove WAD/LAD extensions.
1. Remove Log Analytics agent (MMA/OMS) extensions.
1. Validate that alerts, dashboards, and queries continue to work.
1. Confirm no persistent DCR ingestion, transformation, or delivery errors.

## Minimal validation queries

### Windows events

```kusto
Event
| where TimeGenerated > ago(30m)
| where Source has "ServiceFabric" or RenderedDescription has "Service Fabric"
| take 50
```

### Linux syslog

```kusto
Syslog
| where TimeGenerated > ago(30m)
| where ProcessName has "ServiceFabric" or SyslogMessage has "ServiceFabric"
| take 50
```

### Performance counters

```kusto
Perf
| where TimeGenerated > ago(30m)
| summarize AvgValue = avg(CounterValue) by ObjectName, CounterName, InstanceName
| top 50 by AvgValue desc
```

## Next steps

- Review platform diagnostics concepts in [Monitor Azure Service Fabric](monitor-service-fabric.md).
- Review available counters and signals in [Service Fabric monitoring data reference](monitor-service-fabric-reference.md).
- For Linux event flow details, see [Service Fabric Linux cluster events in Syslog](service-fabric-diagnostics-oms-syslog.md).
