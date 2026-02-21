---
title: Set up IP tags on a Service Fabric managed cluster 
description: Learn how to set IP tags on a Service Fabric managed cluster.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 10/30/2024
# Customer intent: As a cloud administrator, I want to set up IP tags on a Service Fabric managed cluster, so that I can enhance the security of network access controls without the complexity of frequent updates to network security rules.
---

# Set up IP tags on a Service Fabric managed cluster (SFMC)

An IP tag represents a group of IP address prefixes from a given Azure service. Microsoft manages the address prefixes encompassed by the IP tag and automatically updates the service tag as addresses change, minimizing the complexity of frequent updates to network security rules. You can use IP tags to define network access controls on [network security groups](/azure/virtual-network/network-security-groups-overview#security-rules), [Azure Firewall](/azure/firewall/service-tags), and user-defined routes. By adding IP tags, an SFMC cluster has an added layer of security. 
 
> [!NOTE]
> The `IPTag` property only applies to the public IPv4 and IPv6 addresses of the default cluster load balancer.

## Limitations

You shouldn't implement IP tags on existing clusters. 

## Prerequisites

Ensure that you [provisioned IPTags beforehand](/powershell/module/az.network/new-azpublicipaddress#example-3-create-a-new-public-ip-address-with-iptag).

## Modify your ARM template

When creating a new Service Fabric managed cluster, you need to add to the ARM template with the following property: 


```json
{
  "type": "Microsoft.ServiceFabric/managedClusters",
  "apiVersion": "2021-07-01-preview",
  "properties": {
    "ipTags": [
      {
        "ipTagType": "string",
        "tag": "string"
      }
    ]
  }
}
```

Then, deploy your SFMC cluster as normal.

## Next steps

* To see more about Service Fabric managed cluster network settings, see [Configure network settings for Service Fabric managed clusters](how-to-managed-cluster-networking.md).
* To learn about using public IP prefixes in managed clusters, see [Use a Public IP address prefix in a Service Fabric managed cluster](how-to-managed-cluster-public-ip-prefix.md).
