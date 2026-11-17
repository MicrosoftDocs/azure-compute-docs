---
title: Spot Rebalance Recommendation signals (preview Spot Plus)
description: Learn how to receive Spot Plus rebalance recommendations through in-VM Scheduled Events or out-of-VM Azure Event Grid notifications.
ms.date: 11/05/2026
ms.topic: concept-article
ms.reviewer: mattmcinnes
ai-usage: ai-assisted
---

# Spot Rebalance Recommendation signals (preview Spot Plus)

The Spot rebalance recommendation is a feature that provides an early alert when a Spot Instance faces a heightened risk of interruption. When you receive this recommendation, proactively save your state and shift your workload to another Spot Instance to mitigate potential disruptions. Scheduled events deliver rebalance recommendations. You can consume them in two ways: as an **in-VM notification** or as an **out-of-VM notification**.

To receive these notifications for Spot Plus, register `Microsoft.Compute/SpotPlus` by using [Feature Registration](/rest/api/resources/features/register?tabs=HTTP).

> [!NOTE]
> This feature makes a best-effort attempt and doesn't always emit a signal before eviction.

## In-VM rebalance notifications

In-VM rebalance notifications are a scheduled event delivered to the virtual machine via the metadata service. The metadata service provides this information through a REST endpoint that you can access from inside the VM.

> [!IMPORTANT]
> This endpoint uses the same endpoint for preempt notifications, but it uses a different version.

Rebalance notifications are only advisory. Unlike preempt notifications, acknowledging them has no effect. To onboard to Scheduled Events and listen to these events, follow [Scheduled Events for Windows VMs in Azure](windows/scheduled-events.md) or [Scheduled Events for Linux VMs in Azure](linux/scheduled-events.md).

## Out-of-VM rebalance notifications

The following steps explain the onboarding experience for out-of-VM rebalance notifications.

### Create a system topic by using the Azure CLI

For the full command reference, see [az eventgrid system-topic](/cli/azure/eventgrid/system-topic).

1. Set the account to the Azure subscription where you want to create the system topic. Replace the placeholders in the commands with your values.

   ```azurecli
   az account set -s <subscription-id>
   ```

1. Create the system topic. For more information, see [az eventgrid system-topic](/cli/azure/eventgrid/system-topic).

   ```azurecli
   az eventgrid system-topic create --name spotRebalanceTopic --resource-group sampletestrg --source /subscriptions/<subscription-id> --topic-type microsoft.resourcenotifications.MaintenanceResources --location Global
   ```

1. Create the event subscription. For more information, see [az eventgrid system-topic event-subscription create](/cli/azure/eventgrid/system-topic/event-subscription#az-eventgrid-system-topic-event-subscription-create).

   ```azurecli
   az eventgrid system-topic event-subscription create --name firstEventSubscription --resource-group sampletestrg --system-topic-name spotRebalanceTopic --included-event-types Microsoft.ResourceNotifications.MaintenanceResources.ScheduledEventEmitted --endpoint <event-hub-endpoint> --endpoint-type eventhub
   ```

   > [!NOTE]
   > If you don't specify `--included-event-types`, the command includes all event types by default. Ensure that the entity creating the event subscription has read permissions on the entire Azure subscription.

1. To delete the event subscription:

   ```azurecli
   az eventgrid system-topic event-subscription delete --name firstEventSubscription --resource-group sampletestrg --system-topic-name spotRebalanceTopic
   ```

1. To delete the system topic:

   ```azurecli
   az eventgrid system-topic delete --name spotRebalanceTopic --resource-group sampletestrg
   ```

### Create and validate system topics by using the Azure portal

After you create a system topic by using the Azure CLI or PowerShell, use the Azure portal to create event subscriptions and configure filters.

For more information, see [Create, view, and manage system topics](/azure/event-grid/create-view-manage-system-topics).

1. Follow the preceding steps to create the system topic by using the CLI.

1. In the portal, in the search box at the top, type **Event Grid System Topics**, and then press **ENTER**.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/search-event-grid-system-topics.png" alt-text="Screenshot of the Azure portal search box with Event Grid System Topics entered." lightbox="media/spot-rebalance-recommendation-signals-onboarding-guide/search-event-grid-system-topics.png":::

1. On the **Event Grid System Topics** page, view the list of system topics.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/event-grid-system-topics-list.png" alt-text="Screenshot of the Event Grid System topics page listing the system topics." lightbox="media/spot-rebalance-recommendation-signals-onboarding-guide/event-grid-system-topics-list.png":::

1. Select a **system topic** from the list to see details about it.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/event-grid-system-topic-overview.png" alt-text="Screenshot of the system topic details page.":::

   This page shows you details about the system topic, such as the following information:

   - **Source**: Name of the resource on which the system topic was created.
   - **Source type**: Type of the system topic resource. For example: microsoft.resourcenotifications.MaintenanceResources, Microsoft.ResourceNotifications.HealthResources, Microsoft.ResourceNotifications.Resources, Microsoft.Storage.StorageAccounts, Microsoft.EventHub.Namespaces, and Microsoft.Resources.ResourceGroups.
   - Any subscriptions created for the system topic.

   This page allows operations such as the following ones:

   - Create an event subscription: select **+ Event Subscription** on the toolbar.
   - Delete an event subscription: select **Delete** on the toolbar.

1. On the **Event Grid System Topic** page, select **+ Event Subscription** from the toolbar.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/add-event-subscription.png" alt-text="Screenshot of the system topic page with the + Event Subscription button highlighted.":::

1. Confirm that the **Topic Type**, **Source Resource**, and **Topic Name** are automatically populated. Enter a name for the event subscription, select the event types, and specify the endpoint details.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/create-event-subscription-basics.png" alt-text="Screenshot of the Create Event Subscription page showing the Basics tab.":::

1. Select the **Filters** tab to provide subject or advanced filtering for your scenario.

   :::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/event-subscription-filters.png" alt-text="Screenshot of the Filters tab on the Create Event Subscription page.":::

### Example scenario: Subscribe to rebalance signals for a particular resource group

#### Azure portal

Select **Enable subject filtering** under **Subject Filters**. For **Subject Begins With**, enter the value of the resource group in this format: `/subscriptions/{subscription-id}/resourceGroups/{resourceGroup-id}`.

For example, set **Subject Begins With** to `/subscriptions/<subscription-id>/resourceGroups/TestRG`.

:::image type="content" source="media/spot-rebalance-recommendation-signals-onboarding-guide/resource-group-subject-filter.png" alt-text="Screenshot of the event subscription Filters tab with subject filtering enabled for a resource group.":::

#### Azure CLI

```azurecli
az eventgrid system-topic event-subscription create --name firstEventSubForRebalance --resource-group sampletestrg --system-topic-name spotRebalanceTopic --included-event-types Microsoft.ResourceNotifications.MaintenanceResources.ScheduledEventEmitted --endpoint <event-hub-endpoint> --endpoint-type eventhub --subject-begins-with /subscriptions/<subscription-id>/resourceGroups/TestRG/
```

## Related content

- [Scheduled Events for Windows VMs in Azure](windows/scheduled-events.md).

- [Azure Event Grid event schema](/azure/event-grid/event-schema).

- [az eventgrid system-topic](/cli/azure/eventgrid/system-topic).
