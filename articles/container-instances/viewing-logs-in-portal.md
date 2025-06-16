---
title: View container group logs in a log analytics workspace via Azure portal 
description: How to view container group logs in a log analytics workspace via Azure portal 
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: how-to
ms.date: 08/27/2024
---
# View container group logs in a log analytics workspace via Azure portal 

Log Analytics workspaces provide a centralized location for storing and querying log data not only from Azure resources, but also on-premises resources and resources in other clouds. Azure Container Instances includes built-in support for sending logs and event data to Azure Monitor logs. While creating a container instance within Azure portal, you are able to optionally set up Log Analytics. Once your container group is created with Log Analytics enabled, you can view logs within the page for your Azure Container Instance by navigating to “Logs” on the left panel. If you didn’t set up Log Analytics initially, you can also navigate to “Logs” to set it up after you create your container instance also. 

The following sections describe how to set up logging and how to view logs for Azure Container Instances. 

## Create Azure container instance 

This section shows how to create a container instance with Log Analytics. 

> [!NOTE]
> Container instance logs are enabled by default. You can also set up Log Analytics later after you create your container instance by deselecting “Enable container instance logs”. The View Logs section has details on how to do this. 

1. Navigate to the create container instance page in Azure portal.

2. Fill out the relevant information in the Basic and Networking tabs.

3. In the Monitoring tab, enable container instance logs.

If you have a Log Analytics workspace, you can enter the information for it. 

 :::image type="content" source="./media/viewing-logs-in-portal/monitor-tabi.png" alt-text="Screenshot of log analytics screen." lightbox="./media/viewing-logs-in-portal/monitor-tabi.png":::

If you don't have a Log Analytics workspace, you can select “Create New” to create a new one. 

:::image type="content" source="./media/viewing-logs-in-portal/monitor-tab-create-new.png" alt-text="Screenshot of log analytics create new screen." lightbox="./media/viewing-logs-in-portal/monitor-tab-create-new.png":::

Lastly, fill out the following tabs then go to the “Review + Create” tab and click “Create” to create your container instance. 

## View logs 

This section outlines how to view logs for your container instance that you created. 

1. Once your container instance is already created, click “Logs” on the left panel. 

:::image type="content" source="./media/viewing-logs-in-portal/logs-in-portal-running.png" alt-text="Screenshot of logs in left panel." lightbox="./media/viewing-logs-in-portal/logs-in-portal-running.png":::

2. You’ll see the Queries Hub where you can find queries to run in Log Analytics. If these aren't needed, then you can close this screen. 

After closing the previous screen, if you created the ACI with the log-generating image, you should see data for your container group here. 

:::image type="content" source="./media/viewing-logs-in-portal/logs-in-portal.png" alt-text="Screenshot of logs in portal." lightbox="./media/viewing-logs-in-portal/logs-in-portal.png":::
 
3. If you created a container instance without Log Analytics enabled, then you'll see this screen and be able to enable it. 

You can enter an already existing Log Analytics workspace: 

 :::image type="content" source="./media/viewing-logs-in-portal/connect-logs.png" alt-text="Screenshot of create existing log analytics workspace screen." lightbox="./media/viewing-logs-in-portal/connect-logs.png":::

You can also view logs for your container instances by navigating to “Containers” on the left pane, then selecting “Logs” in the middle pane. 

If your container instance is in running state, you’ll be able to see the logs: 

 :::image type="content" source="./media/viewing-logs-in-portal/logs-in-portal-running.png" alt-text="Screenshot of logs in running state." lightbox="./media/viewing-logs-in-portal/logs-in-portal-running.png":::

Now you’re ready to run your container group with Log Analytics. 

## Next Steps 

### Azure Monitor logs 

For more information about querying logs and configuring alerts in Azure Monitor logs, see: 

* [Understanding log searches in Azure Monitor logs](/azure/azure-monitor/logs/log-query-overview)

* [Unified alerts in Azure Monitor](/azure/azure-monitor/alerts/alerts-overview)

### Monitor container CPU and memory 

For information about monitoring container instance CPU and memory resources, see: 

* [Monitor Azure Container Instances](/azure/container-instances/monitor-azure-container-instances)

 

 

 

 
