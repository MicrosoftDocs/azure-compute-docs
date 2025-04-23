---
title: Canonical Ubuntu LTS end of standard support guidance 
description: Understand end of standard support for Ubuntu
author: fossygirl
ms.service: azure-virtual-machines
ms.subservice: vm-linux-setup-configuration
ms.custom: linux-related-content, linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.date: 02/21/2025
ms.author: carols
---

# Canonical Ubuntu LTS end of standard support guidance 

Long-term support (LTS) releases of Ubuntu Server receive standard security updates for five years by default. Every six months, interim releases bring new features, while hardware enablement updates add support for the latest machines to all supported LTS releases. 

When an Ubuntu release reaches the end of its standard support, the following applies: You can continue to use your existing virtual machines; however, security, feature, and maintenance updates will no longer be provided by Canonical. This may leave your systems vulnerable.  

You'll want to consider either migrating to the next Ubuntu LTS release, or upgrading to Ubuntu Pro to gain access to expanded security and maintenance from Canonical.   

## Upgrading to a newer Ubuntu LTS 

Transitioning to the latest operating system, is important for performance, hardware enablement, new technology benefits, and is recommended for new instances. It could be a complex process for existing deployments and should be properly scoped and tested with your workloads.   

You can only directly upgrade one version of Ubuntu LTS to the next version. For instance, if you're running Ubuntu 18.04 LTS, you can't upgrade directly to Ubuntu 22.04 LTS. You would have to first upgrade your Ubuntu 18.04 LTS to Ubuntu 20.04 LTS and then you could upgrade to Ubuntu 22.04 LTS. 

See the [Ubuntu Server upgrade guide](https://ubuntu.com/server/docs/how-to-upgrade-your-release) for more information.  

## Ubuntu Pro – expanded security maintenance for LTS releases

Ubuntu Pro includes security patching for all Ubuntu packages due to Expanded Security Maintenance (ESM) for Infrastructure and Applications and optional 24/7 phone and ticket support. Ubuntu Pro adds an extra five years of support to an Ubuntu LTS release, for a total of 10 years of support. 

New virtual machines can be deployed with Ubuntu Pro from the Azure Marketplace. You can also upgrade Ubuntu Server (version 16.04 or higher) virtual machines to Ubuntu Pro without redeployment or downtime. See the [Ubuntu Server to Ubuntu Pro in-place upgrade on Azure guide](ubuntu-pro-in-place-upgrade.md) for more information.


| **Ubuntu LTS Release** | **End of Standard Support** | **End of Ubuntu Pro Support** |
|---|---|---|
| 24.04   | April 2029 | April 2034|
| 22.04   |  May 2027 | April 2032 |
| 20.04   | May 2025  | April 2030 |
| 18.04   | May 2023  | April 2028 |


## More information  

More information covering the Ubuntu LTS lifecycle can be found here: https://ubuntu.com/about/release-cycle
