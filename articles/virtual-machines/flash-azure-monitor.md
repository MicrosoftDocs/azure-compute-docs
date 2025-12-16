---
title: Project Flash - Use Azure Monitor to monitor Azure Virtual Machine availability
description: This article covers important concepts for monitoring Azure virtual machine availability using the Azure Monitor VM availability metric.
ms.service: azure-virtual-machines
ms.author: tomcassidy
author: tomvcassidy
ms.date: 01/31/2024
ms.topic: concept-article
ms.custom:
  - subject-monitoring
  - sfi-image-nochange
# Customer intent: "As a system administrator, I want to utilize the VM availability metric in Azure Monitor so that I can effectively track and manage the health and performance of my virtual machines."
---

# Project Flash - Use Azure Monitor to monitor Azure Virtual Machine availability

Azure Monitor is one solution offered by Flash. Flash is the internal name for a project dedicated to building a robust, reliable, and rapid mechanism for customers to monitor virtual machine (VM) health.

This article covers the use of the Azure Monitor VM availability metric to monitor Azure Virtual Machine availability. For a general overview of Flash solutions, see the [Flash overview](flash-overview.md).

For documentation specific to the other solutions offered by Flash, choose from the following articles:
* [Use Azure Resource Health to monitor Azure Virtual Machine availability](flash-azure-resource-health.md)
* [Use Azure Resource Graph to monitor Azure Virtual Machine availability](flash-azure-resource-graph.md)
* [Use Event Grid system topics to monitor Azure Virtual Machine availability](flash-event-grid-system-topic.md)

## Azure monitor - VM availability metric

Currently in public preview. The VM availability metric is well-suited for tracking trends, aggregating platform metrics (such as CPU and disk usage), and configuring precise threshold-based alerts. Customers can utilize this out-of-the-box [VM availability metric](monitor-vm-reference.md#vm-availability-metric-preview) in [Azure Monitor](/azure/azure-monitor/platform/alerts-overview). This metric displays the trend of VM availability over time, so users can:

- Set up [threshold-based metric alerts](/azure/azure-monitor/alerts/alerts-create-new-alert-rule?tabs=metric) on dipping VM availability to quickly trigger appropriate mitigation actions.
- Correlate the VM availability metric with existing [platform metrics](/azure/azure-monitor/essentials/data-platform-metrics) like memory, network, or disk for deeper insights into concerning changes that impact the overall performance of workloads.
- Easily interact with and chart metric data during any relevant time window on [Metrics Explorer](/azure/azure-monitor/essentials/metrics-getting-started), for quick and easy debugging.
- Route metrics to downstream tooling like [Grafana dashboards](/azure/azure-monitor/visualize/grafana-plugin), for constructing custom visualizations and dashboards.

### Get started

Users can either consume the metric programmatically via the [Azure Monitor REST API](/rest/api/monitor/metrics) or directly from the [Azure portal](https://portal.azure.com/). The following steps highlight metric consumption from the Azure portal.

Once on the Azure portal, navigate to the VM overview blade. The new metric is displayed as VM Availability (Preview), along with other platform metrics under the Monitoring tab.

   :::image type="content" source="media/flash/virtual-machine-availability-metric.png" alt-text="Screenshot of virtual machine availability metric on a virtual machine's overview page on the Azure portal." lightbox="media/flash/virtual-machine-availability-metric.png" :::

To navigate to the [metrics explorer](/azure/azure-monitor/essentials/metrics-getting-started) for further analysis, select the VM availability metric chart on the overview page.

   :::image type="content" source="media/flash/metrics-explorer-virtual-machine-availability.png" alt-text="Screenshot of the newly added VM availability Metric on Metrics Explorer on Azure portal." lightbox="media/flash/metrics-explorer-virtual-machine-availability.png" :::

For a description of the metric's display values, see [VM availability metric](monitor-vm-reference.md#vm-availability-metric-preview).

Users can split the VM availability by the 'Context' property.
   :::image type="content" source="./media/flash/context-property-filter.png" alt-text="Screenshot of the newly added Context property of VM availability Metric on Azure portal." lightbox="./media/flash/context-property-filter.png" :::

> [!NOTE]
> The context for Service Healing and Live Migration is currently unknown by default.

Users can create an alert rule based on dimension values. Under the Condition tab, choose VM Availability Metric as the Signal name. In the Split by dimensions section, enter Context as the dimension name and select the corresponding dimension values.
   :::image type="content" source="./media/flash/metric-alert-rule.png" alt-text="Screenshot of alert rule creation for VM availability Metric split by dimensions on Azure portal." lightbox="./media/flash/metric-alert-rule.png" :::

### Useful links

- [How to filter events for Azure Event Grid - Azure Event Grid | Microsoft Learn](/azure/event-grid/how-to-filter-events)
- [Event filtering for Azure Event Grid - Azure Event Grid | Microsoft Learn](/azure/event-grid/event-filtering#advanced-filtering)

## Next steps

To learn more about the solutions offered, proceed to corresponding solution article:
* [Use Azure Resource Health to monitor Azure Virtual Machine availability](flash-azure-resource-health.md)
* [Use Azure Resource Graph to monitor Azure Virtual Machine availability](flash-azure-resource-graph.md)
* [Use Event Grid system topics to monitor Azure Virtual Machine availability](flash-event-grid-system-topic.md)

For a general overview of how to monitor Azure Virtual Machines, see [Monitor Azure virtual machines](monitor-vm.md) and the [Monitoring Azure virtual machines reference](monitor-vm-reference.md).
