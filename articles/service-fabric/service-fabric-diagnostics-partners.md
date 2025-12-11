---
title: Azure Service Fabric Monitoring Partners 
description: Learn how to monitor Azure Service Fabric applications, clusters, and infrastructure with partner monitoring solutions.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/14/2022
# Customer intent: As a cloud application developer, I want to select and integrate a third-party monitoring solution for my Service Fabric applications, so that I can effectively monitor performance, gather diagnostics, and enhance observability across my deployment.
---

# Azure Service Fabric Monitoring Partners

This article illustrates how one can monitor their Service Fabric applications, clusters, and infrastructure with a handful of partner solutions. We worked with each of the following partners to create integrated offerings for Service Fabric.

## Dynatrace

Our integration with Dynatrace provides many out of the box features to monitor your Service Fabric clusters. Installing the Dynatrace OneAgent on your Azure Virtual Machine Scale Sets instances gives you performance counters and a topology of your Service Fabric deployment down to the App level. Dynatrace is also a great choice for on-premises monitoring. Check out more of the features listed in the [announcement](https://www.dynatrace.com/news/blog/automatic-end-to-end-service-fabric-monitoring-with-dynatrace/) and [instructions](https://www.dynatrace.com/news/blog/automatic-end-to-end-service-fabric-monitoring-with-dynatrace/) to enable Dynatrace on your cluster. 

## Datadog

Datadog has an extension for Virtual Machine Scale Sets for both Windows and Linux instances. Using Datadog you can collect Windows event logs and thereby collect Service Fabric platform events on Windows. Check out the instructions on how to send your diagnostics data to Datadog [here](https://www.datadoghq.com/blog/azure-monitoring-enhancements/#integrate-with-azure-service-fabric).

## AppDynamics

The Service Fabric integration with AppDynamics is at the application level. By updating environment variables and using App Dynamics NuGets, you can send application telemetry to AppDynamics. Refer to these [instructions](https://docs.appdynamics.com/display/AZURE/Install+AppDynamics+for+Azure+Service+Fabric) for how to integrate your .NET Service Fabric applications with AppDynamics.

## New Relic

New Relic is another Application Performance Management tool that integrates well with Service Fabric applications. You can install the New Relic NuGet packages and add specific environment variables in your manifest files to send your application telemetry to New Relic. Check out these [instructions](https://docs.newrelic.com/docs/agents/net-agent/azure-installation/install-net-agent-azure-service-fabric) to enable New Relic telemetry for your .NET Service Fabric applications.

## Elasticsearch 

Elasticsearch offers a powerful and flexible platform for observing and monitoring your Service Fabric infrastructure. By leveraging Elasticsearch's robust search and analytics engine, you can ingest, store, and analyze diverse telemetry data, including logs, metrics, and traces, from your Service Fabric applications and the underlying cluster. This comprehensive approach allows for deep insights into the health, performance, and operational state of your services. Refer to these [instructions](/azure/partner-solutions/elastic/create?pivots=elastic-search#logs--metrics-tab-optional) on how to establish your Elastic deployment and begin forwarding your Service Fabric and all other Azure resource logs for evaluation and insights.

## Humio

Humio is a log collection service that can gather logs from your applications and events from Service Fabric in the cloud or on-premises in real time. In addition to live observability, Humio offers state-of-the-art analysis and visualization capabilities for viewing and collecting information from your diagnostics. Humio has cost effective pricing plans and is built to scale while retaining it's lightening fast speed. It directly integrates with Service Fabric platform events and Application telemetry. You can read more about the Humio and Service Fabric integration [here](https://github.com/humio/service-fabric-humio).

## Next steps

* Get an [overview of monitoring and diagnostics](monitor-service-fabric.md) in Service Fabric
* Learn how to [diagnose common scenarios](service-fabric-diagnostics-common-scenarios.md) with our first party tools
