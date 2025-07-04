---
title: Collect & analyze resource logs
description: Learn how to send resource logs and event data from container groups in Azure Container Instances to Azure Monitor logs
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.date: 08/29/2024
# Customer intent: As a cloud administrator, I want to configure logging for my Azure Container Instances so that I can collect and analyze resource logs and event data in a centralized logging workspace for better monitoring and troubleshooting.
---

# Container group and instance logging with Azure Monitor logs

Log Analytics workspaces provide a centralized location for storing and querying log data not only from Azure resources, but also on-premises resources and resources in other clouds. Azure Container Instances includes built-in support for sending logs and event data to Azure Monitor logs.

To send container group log and event data to Azure Monitor logs, specify an existing Log Analytics workspace ID and workspace key when configuring a container group. 

The following sections describe how to create a logging-enabled container group and how to query logs. You can also [update a container group](container-instances-update.md) with a workspace ID and workspace key to enable logging.

[!INCLUDE [azure-monitor-log-analytics-rebrand](~/reusable-content/ce-skilling/azure/includes/azure-monitor-log-analytics-rebrand.md)]

## Prerequisites

To enable logging in your container instances, you need the following prerequisites:

* [Log Analytics workspace](/azure/azure-monitor/logs/quick-create-workspace)
* [Azure CLI](/cli/azure/install-azure-cli) (or [Cloud Shell](/azure/cloud-shell/overview))

## Get Log Analytics credentials

Azure Container Instances needs permission to send data to your Log Analytics workspace. To grant this permission and enable logging, you must provide the Log Analytics workspace ID and one of its keys (either primary or secondary) when you create the container group.

To obtain the log analytics workspace ID and primary key:

1. Navigate to your Log Analytics workspace in the Azure portal
1. Under **Settings**, select **Agents management**
1. Take note of:
   * **Workspace ID**
   * **Primary key**

## Create container group

Now that you have the log analytics workspace ID and primary key, you're ready to create a logging-enabled container group.

The following examples demonstrate two ways to create a container group that consists of a single [fluentd][fluentd] container: Azure CLI, and Azure CLI with a YAML template. The fluentd container produces several lines of output in its default configuration. Because this output is sent to your Log Analytics workspace, it works well for demonstrating the viewing and querying of logs.

### Deploy with Azure CLI

To deploy with the Azure CLI, specify the `--log-analytics-workspace` and `--log-analytics-workspace-key` parameters in the [az container create][az-container-create] command. Replace the two workspace values with the values you obtained in the previous step (and update the resource group name) before running the following command.

> [!NOTE]
> The following example pulls a public container image from Docker Hub. We recommend that you set up a pull secret to authenticate using a Docker Hub account instead of making an anonymous pull request. To improve reliability when working with public content, import and manage the image in a private Azure container registry. [Learn more about working with public images.](/azure/container-registry/buffer-gate-public-content)

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainergroup001 \
    --image fluent/fluentd:v1.3-debian-1 \
    --log-analytics-workspace <WORKSPACE_ID> \
    --log-analytics-workspace-key <WORKSPACE_KEY>
```

### Deploy with YAML

Use this method if you prefer to deploy container groups with YAML. The following YAML defines a container group with a single container. Copy the YAML into a new file, then replace `LOG_ANALYTICS_WORKSPACE_ID` and `LOG_ANALYTICS_WORKSPACE_KEY` with the values you obtained in the previous step. Save the file as **deploy-aci.yaml**.

> [!NOTE]
> The following example pulls a public container image from Docker Hub. We recommend that you set up a pull secret to authenticate using a Docker Hub account instead of making an anonymous pull request. To improve reliability when working with public content, import and manage the image in a private Azure container registry. [Learn more about working with public images.](/azure/container-registry/buffer-gate-public-content)

```yaml
apiVersion: 2019-12-01
location: eastus
name: mycontainergroup001
properties:
  containers:
  - name: mycontainer001
    properties:
      environmentVariables: []
      image: fluent/fluentd:v1.3-debian-1
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
  diagnostics:
    logAnalytics:
      workspaceId: LOG_ANALYTICS_WORKSPACE_ID
      workspaceKey: LOG_ANALYTICS_WORKSPACE_KEY
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Next, execute the following command to deploy the container group. Replace `myResourceGroup` with a resource group in your subscription (or first create a resource group named "myResourceGroup"):

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainergroup001 --file deploy-aci.yaml
```

You should receive a response from Azure containing deployment details shortly after issuing the command.

## View logs

After you deploy the container group, it can take several minutes (up to 10) for the first log entries to appear in the Azure portal. 

To view the container group's logs in the `ContainerInstanceLog_CL` table:

1. Navigate to your Log Analytics workspace in the Azure portal
1. Under **General**, select **Logs**  
1. Type the following query: `ContainerInstanceLog_CL | limit 50`
1. Select **Run**

You should see several results displayed by the query. If at first you don't see any results, wait a few minutes, then select the **Run** button to execute the query again. By default, log entries are displayed in **Table** format. You can then expand a row to see the contents of an individual log entry.

![Log Search results in the Azure portal][log-search-01]

## View events

You can also view events for container instances in the Azure portal. Events include the time the instance is created and when it starts. To view the event data in the `ContainerEvent_CL` table:

1. Navigate to your Log Analytics workspace in the Azure portal
1. Under **General**, select **Logs**  
1. Type the following query: `ContainerEvent_CL | limit 50`
1. Select **Run**

You should see several results displayed by the query. If at first you don't see any results, wait a few minutes, then select the **Run** button to execute the query again. By default, entries are displayed in **Table** format. You can then expand a row to see the contents of an individual entry.

![Event Search results in the Azure portal][log-search-02]

## Query container logs

Azure Monitor logs includes an extensive [query language][query_lang] for pulling information from potentially thousands of lines of log output.

The basic structure of a query is the source table (in this article, `ContainerInstanceLog_CL` or `ContainerEvent_CL`) followed by a series of operators separated by the pipe character (`|`). You can chain several operators to refine the results and perform advanced functions.

To see example query results, paste the following query into the query text box, and select the **Run** button to execute the query. This query displays all log entries whose "Message" field contains the word "warn":

```query
ContainerInstanceLog_CL
| where Message contains "warn"
```

More complex queries are also supported. For example, this query displays only those log entries for the "mycontainergroup001" container group generated within the last hour:

```query
ContainerInstanceLog_CL
| where (ContainerGroup_s == "mycontainergroup001")
| where (TimeGenerated > ago(1h))
```

## Log schema

To learn about the Azure Monitor legacy schema for Log Analytics, see [Legacy Log Analytics tables](monitor-azure-container-instances-reference.md#legacy-log-analytics-tables).

## Using Diagnostic Settings

The newer tables require use of diagnostic settings to route information to Log Analytics. The diagnostic settings feature for Container Instances is in public preview, and it can be enabled through preview features options in the Azure portal.

To learn more, see [Azure Monitor Log Analytics tables](monitor-azure-container-instances-reference.md#azure-monitor-log-analytics-tables).

## Next steps

### Azure Monitor logs

For more information about querying logs and configuring alerts in Azure Monitor logs, see:

* [Understanding log searches in Azure Monitor logs](/azure/azure-monitor/logs/log-query-overview)
* [Unified alerts in Azure Monitor](/azure/azure-monitor/alerts/alerts-overview)


### Monitor container CPU and memory

For information about monitoring container instance CPU and memory resources, see:

* [Monitor Azure Container Instances](monitor-azure-container-instances.md).

<!-- IMAGES -->
[log-search-01]: ./media/container-instances-log-analytics/portal-query-01.png
[log-search-02]: ./media/container-instances-log-analytics/portal-query-02.png

<!-- LINKS - External -->
[fluentd]: https://hub.docker.com/r/fluent/fluentd/
[query_lang]: /azure/data-explorer/

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az_container_create
