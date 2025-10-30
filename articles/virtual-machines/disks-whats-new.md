---
title: What's new in Azure Disk Storage
description: Learn about new features and enhancements in Azure Disk Storage.   
author: roygara
ms.author: rogarana
ms.date: 09/04/2025
ms.topic: concept-article
ms.service: azure-disk-storage
ms.custom: references_regions
# Customer intent: "As a cloud administrator, I want to stay informed about new features and enhancements in Azure Disk Storage, so that I can leverage the latest capabilities to optimize performance, cost efficiency, and availability for my virtual machine workloads."
---

# What's new for Azure Disk Storage

Azure Disk Storage regularly receives updates for new features and enhancements. This article provides information about what's new in Azure Disk Storage.

## Update summary

- [What's new in 2025](#whats-new-in-2025)
    -  [Quarter 3 (July, August, September)](#quarter-3-july-august-september)
        - [Generally available: Live Resize for Premium SSD v2 and Ultra Disks using NVMe controllers](#generally-available-live-resize-for-premium-ssd-v2-and-ultra-disks-using-nvme-controllers)
        - [Generally available: Ultra Disk price reduction in West US 2, Central US, and UK South](#generally-available-ultra-disk-price-reduction-in-west-us-2-central-us-and-uk-south)
  - [Quarter 2 (April, May, June)](#quarter-2-april-may-june)
    - [Generally available: Troubleshoot Disk Performance with Copilot in Azure](#generally-available-troubleshoot-disk-performance-with-copilot-in-azure)
    - [Generally Available: Availability Set support for Premium SSD v2 Disk Storage](#generally-available-availability-set-support-for-premium-ssd-v2-disk-storage)
    - [Expanded regional availability for Premium SSD v2](#expanded-regional-availability-for-premium-ssd-v2)
  - [Quarter 1 (January, February, March)](#quarter-1-january-february-march-2)
    - [Public Preview: Troubleshoot Disk Performance with Copilot in Azure](#public-preview-troubleshoot-disk-performance-with-copilot-in-azure)
- [What's new in 2024](#whats-new-in-2024)
  - [Quarter 4 (October, November, December)](#quarter-4-october-november-december)
    - [Generally available: Convert existing disks to Premium SSD v2 disks](#generally-available-convert-existing-disks-to-premium-ssd-v2-disks)
    - [Generally available: Expand Ultra Disks and Premium SSD v2 without downtime](#generally-available-expand-ultra-disks-and-premium-ssd-v2-without-downtime)
    - [Expanded regional availability for Premium SSD v2](#expanded-regional-availability-for-premium-ssd-v2)
  - [Quarter 2 (April, May, June)](#quarter-2-april-may-june)
    - [Generally available: New Property for Disks-LastOwnershipUpdateTime](#generally-available-new-property-for-disks-lastownershipupdatetime)
  - [Quarter 1 (January, February, March)](#quarter-1-january-february-march)
      - [Generally available: Azure VM Backup support for Ultra Disks and Premium SSD v2](#generally-available-azure-vm-backup-support-for-ultra-disks-and-premium-ssd-v2)
      - [Generally available: Trusted launch support for Ultra Disks and Premium SSD v2](#generally-available-trusted-launch-support-for-ultra-disks-and-premium-ssd-v2)
      - [Expanded regional availability for Ultra Disks](#expanded-regional-availability-for-ultra-disks)
      - [Expanded regional availability for zone-redundant storage disks](#expanded-regional-availability-for-zone-redundant-storage-disks)

## What's new in 2025

### Quarter 3 (July, August, September)

#### Generally available: Live Resize for Premium SSD v2 and Ultra Disks using NVMe controllers

You can dynamically expand the storage capacity of your Premium SSD v2 and Ultra Disks using [NVMe controllers](/azure/virtual-machines/nvme-overview) without any disruption to your applications. To optimize costs, you can start with smaller disks and gradually increase their storage capacity as needed, without experiencing downtime. See [expand with Ultra Disks and Premium SSD v2](/azure/virtual-machines/windows/expand-disks#expand-with-ultra-disks-and-premium-ssd-v2) for details.

#### Generally available: Ultra Disk price reduction in West US 2, Central US, and UK South

To better support performance sensitive and mission critical workloads on Azure, Ultra Disk offers improved cost efficiency in West US 2, Central US, and UK South. These enhancements gives you access to the same high-performance storage at a lower cost. For details, see [West US 2](https://azure.microsoft.com/updates?id=499401), [Central US](https://azure.microsoft.com/updates?id=499406), and [UK South](https://azure.microsoft.com/updates?id=499411).

### Quarter 2 (April, May, June)

#### Public preview: Azure Site Recovery for Virtual Machines with Premium SSD v2 and Ultra disks

Azure Site Recovery support for Virtual Machines with [Premium SSD v2](https://azure.microsoft.com/updates?id=495231) and [Ultra](https://azure.microsoft.com/updates?id=495843) disks is in public preview. Azure Site Recovery provides seamless disaster recovery for Virtual Machines across Azure Regions and from on-premises to Azure, helping organizations maintain business continuity. It offers cost-effective replication, automated failover, and easy disaster recovery simulation, ensuring minimal production impact during disaster events. With built-in security, compliance support, and native integration with Azure services, Azure Site Recovery helps your organization stay resilient and minimize downtime. [Learn more](/azure/site-recovery/azure-to-azure-support-matrix).

#### Generally available: Troubleshoot Disk Performance with Copilot in Azure

The Disk Performance Troubleshooting Capability for Copilot in Azure is now [Generally Available](https://azure.microsoft.com/updates?id=474649) as a part of the Copilot in Azure General Availability. Now, you can use Copilot in Azure to analyze your [disk metrics](disks-metrics.md) and resolve any performance degredation issues when your application requires higher performance than what you have configured for your VMs and disks. To learn more, see [Troubleshoot Disk Performance using Microsoft Copilot in Azure](/azure/copilot/troubleshoot-disk-performance).

#### Generally Available: Availability Set support for Premium SSD v2 Disk Storage

Availability Set support for Premium SSD v2 disk storage is now [generally available](https://azure.microsoft.com/updates?id=494088). Availability Set enhances application availability by distributing virtual machines and their associated Premium SSD v2 disks across multiple fault domains, reducing the risk of a single point of failure. Premium SSD v2 provides low latency, consistent performance, flexible scalability, and cost efficiency making it an ideal choice for enterprise workloads such as SAP, SQL Server, and Oracle. The combination of Availability Set and Premium SSD v2 enables customers to achieve higher availability, performance, and cost optimization for their critical applications. To learn more, see [Availability Sets with Premium SSD v2 documentation](https://aka.ms/AvSetWithPv2). 

#### Expanded regional availability for Premium SSD v2

Premium SSD v2 disks were made available in Australia Central 2, Australia Southeast, Canada East, Indonesia Central, Japan West, Malaysia West, New Zealand North, North Central US, Norway West, UK West, US West, and West Central US.

### Quarter 1 (January, February, March)

#### Public Preview: Troubleshoot Disk Performance with Copilot in Azure

The Disk Performance Troubleshooting Capability for Copilot in Azure is now available in [Public Preview](https://azure.microsoft.com/updates?id=474649). Now, you can use Copilot in Azure to analyze your [disk metrics](disks-metrics.md) and resolve any performance degredation issues when your application requires higher performance than what you have configured for your VMs and disks. To learn more, see [Troubleshoot Disk Performance using Microsoft Copilot in Azure](/azure/copilot/troubleshoot-disk-performance).

## What's new in 2024

### Quarter 4 (October, November, December)

#### Generally available: Convert existing disks to Premium SSD v2 disks

Directly converting a disk to a Premium SSD v2 disk is [generally available](https://azure.microsoft.com/updates/?id=466729). Making it easier to move your workloads from Standard HDD, Standard SSD, and Premium Disks, to Premium SSD v2 disks, and take advantage of its balance of price and performance capabilities. To learn more, see [Convert Premium SSD v2 disks](disks-convert-types.md#convert-premium-ssd-v2-disks).

#### Generally available: Expand Ultra Disks and Premium SSD v2 without downtime

Expanding Ultra Disks and Premium SSD v2 disks without downtime is [generally available](https://azure.microsoft.com/updates?id=466724). This allows you to dynamically increase the capacity of your storage without causing disruptions to existing applications. To learn more, see the [Windows](windows/expand-os-disk.md#expand-without-downtime) or [Linux](linux/expand-disks.md#expand-without-downtime) articles.

#### Expanded regional availability for Premium SSD v2

Premium SSD v2 disks were made available in Germany West Central, Israel Central, Italy North, Spain Central, Mexico Central regions. For more information, see the [Azure Update](https://azure.microsoft.com/updates/v2/generally-available-azure-premium-ssd-v2-disk-storage-is-now-available-in-more-regions).

### Quarter 2 (April, May, June)

#### Generally Available: New Property for Disks-LastOwnershipUpdateTime

The `LastOwnershipUpdateTime` property is now available for disks in the Azure portal, Azure PowerShell module, and Azure CLI. This property reflects the time when a disk’s state was last changed. This property can be used with the `diskState` to identify the current state of a disk, and when it was last updated. To learn more, see the [Azure Update](https://azure.microsoft.com/updates/ga-new-property-for-diskslastownershipupdatetime/) post or [the documentation.](/azure/virtual-machines/windows/find-unattached-disks)

### Quarter 1 (January, February, March)

#### Generally available: Azure VM Backup Support for Ultra Disks and Premium SSD v2

Azure Backup enabled support on Azure VMs using Ultra Disks and Premium SSD v2 that offers high throughput, high IOPS, and low latency. Azure VM Backup support allows you to ensure business continuity for your virtual machines and to recover from any disasters or ransomware attacks. Enabling backup on VMs using Ultra Disks and Premium SSD v2 is available in all regions where Ultra disks and Premium SSD v2 disks are supported. To learn more, refer to the [documentation](/azure/backup/backup-support-matrix-iaas#vm-storage-support) and enable backup on your Azure VMs. 


#### Generally available: Trusted launch support for Ultra Disks and Premium SSD v2

Trusted launch VMs added support for Ultra Disks and Premium SSD v2, allowing you to combine the foundational compute security of Trusted Launch with the high throughput, high IOPS, and low latency of Ultra Disks and Premium SSD v2. For more information, see [Trusted launch for Azure virtual machines](trusted-launch.md) or the [Azure Update](https://azure.microsoft.com/updates/premium-ssd-v2-and-ultra-disks-support-with-trusted-launch-vm/).

#### Expanded regional availability for Ultra Disks

Ultra Disks were made available in the UK West and Poland Central regions.

#### Expanded regional availability for zone-redundant storage disks

Zone-redundant storage (ZRS) disks were made available in West US 3 and Germany Central regions.

## What's new in 2023

### Quarter 4 (October, November, December)

#### Encryption at host GA for Premium SSD v2 and Ultra Disks

Encryption at host was previously only available for Standard HDDs, Standard SSDs, and Premium SSDs. Encryption at host is now also available as a GA offering for Premium SSD v2 and Ultra Disks. For more information on encryption at host, see [Encryption at host - End-to-end encryption for your VM data](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

There are some additional restrictions for Premium SSD v2 and Ultra Disks that enable encryption at host. For more information, see [Restrictions](disk-encryption.md#restrictions-1).

#### New latency metrics (preview)

Metrics dedicated to monitoring latency are now available as a preview feature. To learn more, see either the [metrics article](disks-metrics.md#disk-io-throughput-queue-depth-and-latency-metrics) or the [Azure Update](https://azure.microsoft.com/updates/latency-metrics-for-azure-disks-and-performance-metrics-for-temporary-disks-on-azure-virtual-machines/).

#### Expanded regional availability for Premium SSD v2

Premium SSD v2 disks were made available in Poland Central, China North 3, and US Gov Virginia. For more information, see the [Azure Update](https://azure.microsoft.com/updates/generally-available-azure-premium-ssd-v2-disk-storage-is-now-available-in-more-regions-pcu/).


#### Expanded regional availability for ZRS disks

ZRS disks were made available in the Norway East and UAE North regions. For more information, see the [Azure Update](https://azure.microsoft.com/updates/generally-available-zone-redundant-storage-for-azure-disks-is-now-available-in-norway-east-uae-north-regions/).

### Quarter 3 (July, August, September)

#### Expanded regional availability for ZRS disks

In quarter 3, ZRS disks were made available in the China North 3, East Asia, India Central, Switzerland North, South Africa North, and Sweden Central regions.

#### Expanded regional availability for Premium SSD v2

In Quarter 3, Premium SSD v2 were made available in the Australia East, Brazil South, Canada Central, Central India, Central US, East Asia, France Central, Japan East, Korea Central, Norway East, South Africa North, Sweden Central, Switzerland North, and UAE North regions.

#### General Availability - Incremental Snapshots for Premium SSD v2 and Ultra Disks

Incremental snapshots for Premium SSD v2 and Ultra Disks were made available as a general availability (GA) feature. For more information, see either the [documentation](disks-incremental-snapshots.md#incremental-snapshots-of-premium-ssd-v2-and-ultra-disks) or the [Azure Update](https://azure.microsoft.com/updates/general-availability-incremental-snapshots-for-premium-ssd-v2-disk-and-ultra-disk-storage-3/).

### Quarter 2 (April, May, June)

#### Expanded regional availability for Premium SSD v2

In quarter 2, Premium SSD v2 disks were made available in the Southeast Asia, UK South, South Central US, and West US 3 regions.

#### Expanded regional availability for ZRS disks

In quarter 2, ZRS disks were made available in the Australia East, Brazil South, Japan East, Korea Central, Qatar Central, UK South, East US, East US 2, South Central US, and Southeast Asia regions.

#### Azure Backup support (preview) for Premium SSD v2

Azure Backup added preview support for Azure virtual machines using Premium SSD v2 disks in the East US and West Europe regions. For more information, see the [Azure Update](https://azure.microsoft.com/updates/premium-ssd-v2-backup-support/).

### Quarter 1 (January, February, March)

#### Expanded regional availability for Premium SSD v2

In quarter 1, Premium SSD v2 disks were made available in the East US 2, North Europe, and West US 2 regions.

#### Preview - Performance plus

Azure Disk Storage added a new preview feature, performance plus. Performance plus enhances the IOPS and throughput performance for Premium SSDs, Standard SSDs, and Standard HDDs that are 513 GiB and larger. For details, see [Increase IOPS and throughput limits for Azure Premium SSDs and Standard SSD/HDDs](disks-enable-performance.md)

#### Expanded regional availability for Ultra Disks

In quarter 1, Ultra Disks were made available in the Brazil Southeast, China North 3, Korea South, South Africa North, Switzerland North, and UAE North regions.

#### More transactions at no extra cost - Standard SSDs

In quarter 1, we added an hourly limit to the number of transactions that can occur a billable cost. Any transactions beyond that limit don't occur a cost. For more information, see the [blog post](https://aka.ms/billedcapsblog) or [Standard SSD transactions](disks-types.md#standard-ssd-transactions).

#### GA: Create disks from snapshots encrypted with customer-managed keys across subscriptions

In quarter 1, support for creating disks from snapshots or other disks encrypted with customer-managed keys in different subscriptions while within the same tenant was added. For more information, see either the [Azure Update](https://azure.microsoft.com/updates/ga-create-disks-from-cmkencrypted-snapshots-across-subscriptions-and-in-the-same-tenant/) or [the documentation](disk-encryption.md#customer-managed-keys).

#### GA: Entra ID support for managed disks

In quarter 1, support for using Entra ID to secure uploads and downloads of managed disks was added. For details, see [Secure downloads with Microsoft Entra ID](linux/download-vhd.md#secure-downloads-and-uploads-with-microsoft-entra-id) or [Secure uploads with Microsoft Entra ID](windows/disks-upload-vhd-to-managed-disk-powershell.md#secure-uploads-with-microsoft-entra-id).

## Next steps

- [Azure managed disk types](disks-types.md)
- [Introduction to Azure managed disks](managed-disks-overview.md)
