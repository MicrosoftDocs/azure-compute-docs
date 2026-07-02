---
title: Performance Monitoring with Azure Monitor logs (retired)
description: This article is retired. Use Azure Monitor Agent (AMA) and data collection rules (DCRs) for Service Fabric telemetry instead of Log Analytics agent (MMA/OMS).
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 03/22/2026
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: As a DevOps engineer, I want to use supported telemetry configuration for Service Fabric so that monitoring remains supported.
---

# Performance monitoring with Azure Monitor logs (retired)

> [!WARNING]
> This article is retired and kept only for historical reference. Don't use Log Analytics agent (MMA/OMS) for new or existing Service Fabric telemetry onboarding.

Use [Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules](service-fabric-diagnostics-azure-monitor-agent-data-collection-rules.md) for supported guidance.

For Azure Monitor migration details, see [Migrate from Azure Diagnostics extension to Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-migration).

## Why this article was retired

Log Analytics agent (MMA/OMS) is deprecated and unsupported. Configure Service Fabric cluster telemetry by using Azure Monitor Agent (AMA) with Data Collection Rules (DCRs).

## What to do instead

1. Install Azure Monitor Agent on cluster nodes.
1. Create and associate Data Collection Rules for required logs and counters.
1. Validate ingestion in `Event`, `Syslog`, and `Perf` tables as applicable.
1. Remove deprecated agents after cutover validation.

## Next steps

* Configure telemetry by using [Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules](service-fabric-diagnostics-azure-monitor-agent-data-collection-rules.md).
* Review Service Fabric counters in [Service Fabric monitoring data reference](monitor-service-fabric-reference.md#performance-metrics).
* Configure alerting by using [Azure Monitor alerts overview](/azure/azure-monitor/alerts/alerts-overview).
