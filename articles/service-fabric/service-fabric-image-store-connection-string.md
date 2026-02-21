---
title: Azure Service Fabric image store connection string 
description: Learn about the image store connection string, including its uses and applications to a Service Fabric cluster.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/14/2022
ms.update-cycle: 1095-days
# Customer intent: "As a cloud architect, I want to understand the ImageStoreConnectionString setting for a Service Fabric cluster, so that I can effectively configure and manage application deployments while ensuring optimal storage management."
---

# Understand the ImageStoreConnectionString setting

In some of our documentation, we briefly mention the existence of an "ImageStoreConnectionString" parameter without describing what it really means. And after going through an article like [Deploy and remove applications using PowerShell][10], it looks like all you do is copy/paste the value as shown in the cluster manifest of the target cluster. So the setting must be configurable per cluster, but when you create a cluster through the [Azure portal][11], there's no option to configure this setting and it's always "fabric:ImageStore". What's the purpose of this setting then?

![Cluster Manifest][img_cm]

Service Fabric started off as a platform for internal Microsoft consumption by many diverse teams, so some aspects of it are highly customizable - the "Image Store" is one such aspect. Essentially, the Image Store is a pluggable repository for storing application packages. When your application is deployed to a node in the cluster, that node downloads the contents of your application package from the Image Store. The ImageStoreConnectionString is a setting that includes all the necessary information for both clients and nodes to find the correct Image Store for a given cluster.

There are currently two possible kinds of Image Store providers and their corresponding connection strings are as follows:

1. Image Store Service: "fabric:ImageStore"

2. File System: "file:[file system path]"

The provider type used in production is the Image Store Service, which is a stateful persisted system service that you can see from Service Fabric Explorer. 

![Image Store Service][img_is]

Hosting the Image Store in a system service within the cluster itself eliminates external dependencies for the package repository and gives us more control over the locality of storage. Future improvements around the Image Store are likely to target the Image Store provider first, if not exclusively. The connection string for the Image Store Service provider doesn't have any unique information since the client is already connected to the target cluster. The client only needs to know that protocols targeting the system service should be used.

The File System provider is used instead of the Image Store Service for local one-box clusters during development to bootstrap the cluster slightly faster. The difference is typically small, but it's a useful optimization for most folks during development. It's possible to deploy a local one-box cluster with the other storage provider types as well, but there's usually no reason to do so since the develop/test workflow remains the same regardless of provider.

Furthermore, the File System provider should not be used as a method of sharing an Image Store between multiple clusters - this will result in corruption of cluster configuration data as each cluster can write conflicting data to the Image Store. To share provisioned application packages between multiple clusters, use [sfpkg][12] files instead, which can be uploaded to any external store with a download URI.

So while the ImageStoreConnectionString is configurable, you just use the default setting. When publishing to Azure through Visual Studio, the parameter is automatically set for you accordingly. For programmatic deployment to clusters hosted in Azure, the connection string is always "fabric:ImageStore". Though when in doubt, its value can always be verified by retrieving the cluster manifest by [PowerShell](/powershell/module/servicefabric/get-servicefabricclustermanifest), [.NET](/previous-versions/azure/reference/mt161375(v=azure.100)), or [REST](/rest/api/servicefabric/get-a-cluster-manifest). Both on-premises test and production clusters should always be configured to use the Image Store Service provider as well.

### Next steps
[Deploy and remove applications using PowerShell][10]

<!--Image references-->
[img_is]: ./media/service-fabric-image-store-connection-string/image_store_service.png
[img_cm]: ./media/service-fabric-image-store-connection-string/cluster_manifest.png

[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-cluster-creation-via-portal.md
[12]: service-fabric-package-apps.md#create-an-sfpkg
