---
title: Set up monitoring with Azure Monitor logs (retired)
description: This article is retired. Use Azure Monitor Agent (AMA) and data collection rules (DCRs) for Service Fabric telemetry instead of OMS-era setup.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
ms.custom: devx-track-arm-template, devx-track-azurepowershell
services: service-fabric
ms.date: 03/22/2026
# Customer intent: As a DevOps engineer, I want to use supported Service Fabric monitoring setup so that telemetry stays supported.
---

# Set up monitoring with Azure Monitor logs (retired)

> [!WARNING]
> This article is retired and kept only for historical reference. Don't use this OMS-era setup path for new onboarding.

Use [Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules](service-fabric-diagnostics-azure-monitor-agent-data-collection-rules.md) for supported guidance.

For Azure Monitor migration details, see [Migrate from Azure Diagnostics extension to Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-migration).

## Why this article was retired

This article describes legacy patterns that depend on deprecated diagnostics onboarding and old workspace integration steps. Service Fabric monitoring should use Azure Monitor Agent (AMA) and data collection rules (DCRs).

## What to do instead

1. Install AMA on cluster nodes.
1. Create and associate DCRs for required logs and performance counters.
1. Validate data flow in `Event`, `Syslog`, and `Perf` tables.
1. Remove deprecated WAD/LAD and Log Analytics agent configurations after cutover.

## Next steps

* Configure telemetry with [Configure Service Fabric cluster telemetry with Azure Monitor Agent and data collection rules](service-fabric-diagnostics-azure-monitor-agent-data-collection-rules.md).
* Get familiar with [Log Analytics query language overview](/azure/azure-monitor/logs/log-query-overview).
* See [Monitor Azure Service Fabric](monitor-service-fabric.md) for end-to-end monitoring architecture.
