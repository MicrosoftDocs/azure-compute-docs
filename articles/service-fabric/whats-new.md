---
title: What's new for Service Fabric
description: Learn about what's new for Service Fabric.
author: tomvcassidy
ms.service: azure-service-fabric
ms.topic: whats-new
ms.date: 10/22/2025
ms.author: tomcassidy
# Customer intent: "As a system administrator, I want to stay informed about the latest updates and features for Service Fabric, so that I can ensure my applications utilize the most current and efficient functionalities for improved performance and reliability."
---

# What's new for Service Fabric

Learn what's new for Service Fabric managed clusters and classic.

This article's coverage begins in January 2023. For all Service Fabric runtime release announcements, including versions older than January 2023, see [Service Fabric releases](release-notes.md).

This article provides information about:
* The latest releases
* Known issues
* Bug fixes
* Retired functionality

## Announcements

### September 2025
* Service Fabric 11 CU2 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_112.md).
* Service Fabric 10.1 CU8 released.

### July 2025
* Service Fabric 11 CU1 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_111.md).
* Service Fabric 10.1 CU8 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU8.md).

### June 2025
* Service Fabric 11 RTO released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_110.md).
* Service Fabric 10.1 CU7 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU7.md).
* Service Fabric 10.0 CU8 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU8.md).
* Service Fabric 9.1 CU14 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU14.md).

### October 2024
* Service Fabric 10.1 CU6 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU6.md).
* Service Fabric 10.0 CU7 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU7.md).
* Service Fabric 9.1 CU13 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU13.md).

### September 2024

* Service Fabric 10.1 CU5 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU5.md).
  * User-assigned managed identity support for Backup and Restore
* Service Fabric 10.0 CU6 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU6.md).
  * User-assigned managed identity support for Backup and Restore
* Service Fabric 9.1 CU12 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU12.md).
  * User-assigned managed identity support for Backup and Restore

### August 2024

* Service Fabric 10.1 CU4 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU4.md).
  * Reliable Collections with string keys can switch comparison behavior from CurrentCulture to Ordinal
  * Retired ReplicatedStore settings
* Service Fabric 10.0 CU5 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU5.md).
  * Reliable Collections with string keys can switch comparison behavior from CurrentCulture to Ordinal
  * Retired ReplicatedStore settings
* Service Fabric 9.1 CU11 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU11.md).

### June 2024

* Service Fabric 10.1 CU3 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU3.md).
  * Programming model support for **.NET 8.0**
  * Improvements to health reporting
  * Improvements to Key Value Store
  * CertificateManager class added to Service Fabric SDK
* Service Fabric 10.0 CU4 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU4.md).
  * Improvements to health reporting
  * Improvements to Key Value Store
* Service Fabric 9.1 CU10 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU10.md).
  * Improvements to health reporting
  * Improvements to Key Value Store

### April 2024

* Service Fabric 10.1 CU2 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101CU2.md).
  * Improvements to Key Value Store
  * ElevatedAdmin role dSTS support
  * ElevatedAdmin role REST API call support
* Service Fabric 10.0 CU3 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU3.md).
* Service Fabric 9.1 CU9 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU9.md).
  * Improvements to Key Value Store

### January 2024

* Support for Service Fabric 9.0 ends.

### November 2023

* Service Fabric 10.1 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_101RTO.md).
  * New client role, ElevatedAdmin, to improve tenant operations in shared clusters
  * Improvements to Disable-ServiceFabricNode command
  * Health events now visible in SFX/SFE
  * Sensitivity feature for Cluster Resource Manager
  * Auxiliary replica throttling
* Service Fabric 10.0CU1 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_100CU1.md).
* Service Fabric 9.1CU7 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU7.md).
  * Improvements to Key Value Store
  * Modifications to placement load balancing to improve replica build times and resource consumption
* Service Fabric 9.0CU12 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_90CU12.md).
  * Improvements to Key Value Store

### September 2023

* Service Fabric 10.0 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_10.md).
  * Enhances container image pruning
  * Balancing of a cluster per node type
  * Exposes health check phase and timer for application and cluster upgrade
  * Supports ESE.dll version compatibility in the replica building process
  * Enables Lease probes
  * Extends the FabricClient constructor to include "SecurityCredentials" without "HostEndpoints"
  * Security audit of cluster management endpoint settings

### August 2023

* Service Fabric 9.1CU6 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU6.md).
  * Improvements to Reliable Collections

### July 2023

Service Fabric managed clusters adds support for:
* [Using NAT gateways to route network traffic](how-to-managed-cluster-nat-gateway.md)
* [Enabling public IPv4 on secondary node types](how-to-managed-cluster-networking.md#enable-public-ip)
* [Using public IP prefixes to reserve ranges of public IP addresses for your endpoints in Azure](how-to-managed-cluster-public-ip-prefix.md)

### June 2023

* Service Fabric 9.1CU5 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU5.md).
  * Improvements to Key Value Store

### May 2023

* Service Fabric 9.1CU4 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU4.md).
  * Improvements to Key Value Store
  * Application model improved to address issue with invalid characters
  * Improvements to Failover Manager to support upgrade process
* Service Fabric 9.0CU9 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_90CU9.md).
  * Improvements to Key Value Store

### April 2023

* Service Fabric 9.1CU3 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU3.md).
  * Improvements to Key Value Store
* Service Fabric 9.0CU8 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_90CU8.md).
  * Improvements to Key Value Store

### March 2023

* Service Fabric 9.1CU2 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_91CU2.md).
  * Programming model support for **.NET 7.0**
  * Policy added to validate that the minimum virtual machine count configuration meets "Silver" and "Gold" durability requirements
* Service Fabric 9.0CU7 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_90CU7.md).
* Service Fabric 8.2CU9 released. For more information, see the [release notes](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_82CU9.md).
* Support for Service Fabric 8.2 ends.

### January 2023

Service Fabric managed clusters adds support for:
* Creating different subnets for each node type using custom virtual networks.

## Next steps

For updates and announcements about Azure, see the [Microsoft Azure Blog](https://azure.microsoft.com/blog/).
