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
# Customer intent: "As an IT administrator managing Ubuntu LTS virtual machines, I want to understand the end of standard support implications, so that I can ensure my systems remain secure and consider upgrading or enabling extended security maintenance options."
---

# Canonical Ubuntu LTS end of standard support guidance 

Long-term support (LTS) releases of Ubuntu receive standard security maintenance for packages in the Ubuntu Main repository for five years from the release date. Every six months between LTS releases, interim releases bring new features for testing and development. Hardware enablement (HWE) updates, providing newer kernel and graphics stacks, are made available for supported LTS releases. 

When an Ubuntu LTS release reaches the end of its standard support period, the following applies: You can continue to use your existing virtual machines; however, security, feature, and maintenance updates will no longer be provided by Canonical under standard support. This may leave your systems vulnerable.  

You should consider either migrating to a supported Ubuntu LTS release, or enabling Ubuntu Pro to gain access to expanded security and maintenance and other features from Canonical.   

## Upgrading to a newer Ubuntu LTS 

Transitioning to the latest operating system is important for performance, hardware enablement, access to new technology, and is recommended for new instances. Upgrading could be a complex process for existing deployments and should be properly scoped and tested with your workloads.   

You can only directly upgrade one version of Ubuntu LTS to the next version. For instance, upgrading from Ubuntu 20.04 LTS to Ubuntu 24.04 LTS requires a two-step process: first upgrade Ubuntu 20.04 LTS to Ubuntu 22.04 LTS, and then upgrade from Ubuntu 22.04 LTS to Ubuntu 24.04 LTS. 

See the [Ubuntu Server upgrade guide](https://ubuntu.com/server/docs/how-to-upgrade-your-release) for more information.  

## Ubuntu Pro – expanded security maintenance for LTS releases

Ubuntu Pro is a subscription that expands Canonical's security maintenance coverage and provides optional support. It includes Expanded Security Maintenance (ESM) providing security patching for critical, high, and selected medium CVEs for packages in both the Ubuntu Main and Universe repositories. Ubuntu Pro extends the security coverage period for an Ubuntu LTS release up to a total of 12 years. Optional 24/7 phone and ticket support plans are also available with Ubuntu Pro.

New virtual machines can be deployed with Ubuntu Pro from the Azure Marketplace. You can also upgrade existing Ubuntu Server (version 16.04 LTS or higher) virtual machines to Ubuntu Pro without redeployment or downtime. See the [Ubuntu Server to Ubuntu Pro in-place upgrade on Azure guide](ubuntu-pro-in-place-upgrade.md) for more information.


| **Ubuntu LTS Release** | **End of Standard Support** | **End of Ubuntu Pro Support** |
|---|---|---|
| 24.04   | May 2029 | April 2034|
| 22.04   |  May 2027 | April 2032 |
| 20.04   | May 2025  | April 2030 |
| 18.04   | May 2023  | April 2028 |
| 16.04   | May 2021  | April 2026 |


## More information  

More information covering the Ubuntu LTS lifecycle can be found here: https://ubuntu.com/about/release-cycle
