---
title: What's new in Azure Disk Storage
description: Learn about new features and enhancements in Azure Disk Storage.   
author: roygara
ms.author: rogarana
ms.date: 09/09/2026
ms.topic: concept-article
ms.service: azure-disk-storage
ms.custom: references_regions
ai-usage: ai-assisted
# Customer intent: "As a cloud administrator, I want to stay informed about new features and enhancements in Azure Disk Storage, so that I can leverage the latest capabilities to optimize performance, cost efficiency, and availability for my virtual machine workloads."
---

# What's new for Azure Disk Storage

Azure Disk Storage regularly receives updates for new features and enhancements. This article provides information about what's new in Azure Disk Storage.

## Update summary

- [What's new in 2026](#whats-new-in-2026)
  - [Quarter 3 (July, August, September)](#quarter-3-july-august-september)
    - [Expanded regional availability for Ultra Disks in Australia Southeast in Q3 2026](#expanded-regional-availability-for-ultra-disks-in-australia-southeast-in-q3-2026)
  - [Quarter 1 (January, February, March)](#quarter-1-january-february-march)
    - [Expanded regional availability for Premium SSD v2 in Q1 2026](#expanded-regional-availability-for-premium-ssd-v2-in-q1-2026)
- [What's new in 2025](#whats-new-in-2025)
  - [Quarter 4 (October, November, December)](#quarter-4-october-november-december)
    - [Expanded regional availability for Premium SSD v2 in Q4 2025](#expanded-regional-availability-for-premium-ssd-v2-in-q4-2025)
    - [Public preview: Instant Access Snapshot for Premium SSD v2 and Ultra Disks in Q4 2025](#public-preview-instant-access-snapshot-for-premium-ssd-v2-and-ultra-disks-in-q4-2025)
    - [Generally available: Azure Site Recovery for virtual machines with Premium SSD v2 and Ultra Disks in Q4 2025](#generally-available-azure-site-recovery-for-virtual-machines-with-premium-ssd-v2-and-ultra-disks-in-q4-2025)
  - [Quarter 3 (July, August, September)](#quarter-3-july-august-september-1)
    - [Generally available: Live resize for Premium SSD v2 and Ultra Disks using NVMe controllers in Q3 2025](#generally-available-live-resize-for-premium-ssd-v2-and-ultra-disks-using-nvme-controllers-in-q3-2025)
    - [Ultra Disk price reduction in West US 2, Central US, and UK South in Q3 2025](#ultra-disk-price-reduction-in-west-us-2-central-us-and-uk-south-in-q3-2025)
  - [Quarter 2 (April, May, June)](#quarter-2-april-may-june)
    - [Public preview: Azure Site Recovery for virtual machines with Premium SSD v2 and Ultra Disks in Q2 2025](#public-preview-azure-site-recovery-for-virtual-machines-with-premium-ssd-v2-and-ultra-disks-in-q2-2025)
    - [Generally available: Troubleshoot disk performance with Microsoft Copilot in Azure in Q2 2025](#generally-available-troubleshoot-disk-performance-with-microsoft-copilot-in-azure-in-q2-2025)
    - [Generally available: Availability set support for Premium SSD v2 in Q2 2025](#generally-available-availability-set-support-for-premium-ssd-v2-in-q2-2025)
    - [Expanded regional availability for Premium SSD v2 in Q2 2025](#expanded-regional-availability-for-premium-ssd-v2-in-q2-2025)
  - [Quarter 1 (January, February, March)](#quarter-1-january-february-march-1)
    - [Public preview: Troubleshoot disk performance with Microsoft Copilot in Azure in Q1 2025](#public-preview-troubleshoot-disk-performance-with-microsoft-copilot-in-azure-in-q1-2025)
- [What's new in 2024](#whats-new-in-2024)
  - [Quarter 4 (October, November, December)](#quarter-4-october-november-december-1)
    - [Generally available: Convert existing disks to Premium SSD v2 disks in Q4 2024](#generally-available-convert-existing-disks-to-premium-ssd-v2-disks-in-q4-2024)
    - [Generally available: Expand Ultra Disks and Premium SSD v2 without downtime in Q4 2024](#generally-available-expand-ultra-disks-and-premium-ssd-v2-without-downtime-in-q4-2024)
    - [Expanded regional availability for Premium SSD v2 in Q4 2024](#expanded-regional-availability-for-premium-ssd-v2-in-q4-2024)
  - [Quarter 2 (April, May, June)](#quarter-2-april-may-june-1)
    - [Generally available: LastOwnershipUpdateTime disk property in Q2 2024](#generally-available-lastownershipupdatetime-disk-property-in-q2-2024)
  - [Quarter 1 (January, February, March)](#quarter-1-january-february-march-2)
    - [Generally available: Azure Backup support for virtual machines with Ultra Disks and Premium SSD v2 in Q1 2024](#generally-available-azure-backup-support-for-virtual-machines-with-ultra-disks-and-premium-ssd-v2-in-q1-2024)
    - [Generally available: Trusted launch support for Ultra Disks and Premium SSD v2 in Q1 2024](#generally-available-trusted-launch-support-for-ultra-disks-and-premium-ssd-v2-in-q1-2024)
    - [Expanded regional availability for Ultra Disks in Q1 2024](#expanded-regional-availability-for-ultra-disks-in-q1-2024)
    - [Expanded regional availability for zone-redundant storage disks in Q1 2024](#expanded-regional-availability-for-zone-redundant-storage-disks-in-q1-2024)
## What's new in 2026

### Quarter 3 (July, August, September)

#### Expanded regional availability for Ultra Disks in Australia Southeast in Q3 2026

In Q3 2026, Ultra Disks became available in Australia Southeast. For information about regional availability and deployment requirements, see [Deploy an Ultra Disk](disks-enable-ultra-ssd.md).

### Quarter 1 (January, February, March)

#### Expanded regional availability for Premium SSD v2 in Q1 2026

In Q1 2026, Premium SSD v2 disks became available in Brazil Southeast, Germany North, India South, Switzerland West, and US Gov Arizona, and in a third availability zone in Indonesia Central, Malaysia West, and New Zealand North.

## What's new in 2025

### Quarter 4 (October, November, December)

#### Expanded regional availability for Premium SSD v2 in Q4 2025

In Q4 2025, Premium SSD v2 disks became available in Austria East and in a second availability zone in Japan West.

#### Public preview: Instant Access Snapshot for Premium SSD v2 and Ultra Disks in Q4 2025

In Q4 2025, Instant Access Snapshot became available in public preview for Premium SSD v2 and Ultra Disks. You can restore new disks immediately after creating snapshots of Premium SSD v2 and Ultra Disks. Restored disks deliver high performance instantly, while data hydration continues rapidly in the background. For more information, see [Instant Access Snapshot for Azure managed disks](/azure/virtual-machines/disks-instant-access-snapshots?tabs=azure-cli%2Cazure-cli-snapshot-state#snapshots-of-ultra-disks-and-premium-ssd-v2).

#### Generally available: Azure Site Recovery for virtual machines with Premium SSD v2 and Ultra Disks in Q4 2025

In Q4 2025, Azure Site Recovery support for virtual machines with [Premium SSD v2](https://azure.microsoft.com/updates?id=495231) and [Ultra](https://azure.microsoft.com/updates?id=495843) disks became generally available. For support details, see the [Azure Site Recovery support matrix](/azure/site-recovery/azure-to-azure-support-matrix).

### Quarter 3 (July, August, September)

#### Generally available: Live resize for Premium SSD v2 and Ultra Disks using NVMe controllers in Q3 2025

In Q3 2025, live resize for Premium SSD v2 and Ultra Disks using [NVMe controllers](/azure/virtual-machines/nvme-overview) became generally available. You can dynamically expand disk capacity without disrupting your applications. To optimize costs, you can start with smaller disks and gradually increase their storage capacity as needed, without experiencing downtime. For more information, see [Expand Ultra Disks and Premium SSD v2](/azure/virtual-machines/windows/expand-disks#expand-with-ultra-disks-and-premium-ssd-v2).

#### Ultra Disk price reduction in West US 2, Central US, and UK South in Q3 2025

In Q3 2025, Ultra Disk prices decreased in West US 2, Central US, and UK South. The price reduction improves cost efficiency for performance-sensitive and mission-critical workloads while providing the same high-performance storage. For details, see the updates for [West US 2](https://azure.microsoft.com/updates?id=499401), [Central US](https://azure.microsoft.com/updates?id=499406), and [UK South](https://azure.microsoft.com/updates?id=499411).

### Quarter 2 (April, May, June)

#### Public preview: Azure Site Recovery for virtual machines with Premium SSD v2 and Ultra Disks in Q2 2025

In Q2 2025, Azure Site Recovery support for virtual machines with [Premium SSD v2](https://azure.microsoft.com/updates?id=495231) and [Ultra](https://azure.microsoft.com/updates?id=495843) disks became available in public preview. Azure Site Recovery provides disaster recovery for virtual machines across Azure regions and from on-premises to Azure. It offers cost-effective replication, automated failover, and disaster recovery simulation with minimal production impact. Built-in security, compliance support, and native integration with Azure services help your organization stay resilient and minimize downtime. For preview support details, see the [Azure Site Recovery support matrix](/azure/site-recovery/azure-to-azure-support-matrix).

#### Generally available: Troubleshoot disk performance with Microsoft Copilot in Azure in Q2 2025

In Q2 2025, disk performance troubleshooting with [Microsoft Copilot in Azure](https://azure.microsoft.com/updates?id=474649) moved from public preview to general availability when Microsoft Copilot in Azure became generally available. For more information, see [Troubleshoot disk performance using Microsoft Copilot in Azure](/azure/copilot/troubleshoot-disk-performance).

#### Generally available: Availability set support for Premium SSD v2 in Q2 2025

In Q2 2025, availability set support for Premium SSD v2 became [generally available](https://azure.microsoft.com/updates?id=494088). Availability sets enhance application availability by distributing virtual machines and their Premium SSD v2 disks across multiple fault domains, which reduces the risk of a single point of failure. Premium SSD v2 provides low latency, consistent performance, flexible scalability, and cost efficiency for enterprise workloads such as SAP, SQL Server, and Oracle. Combining availability sets with Premium SSD v2 can improve availability, performance, and cost optimization for critical applications. For more information, see [Availability sets with Premium SSD v2](https://aka.ms/AvSetWithPv2).

#### Expanded regional availability for Premium SSD v2 in Q2 2025

In Q2 2025, Premium SSD v2 disks became available in Australia Central 2, Australia Southeast, Canada East, Indonesia Central, Japan West, Malaysia West, New Zealand North, North Central US, Norway West, UK West, US West, and West Central US.

### Quarter 1 (January, February, March)

#### Public preview: Troubleshoot disk performance with Microsoft Copilot in Azure in Q1 2025

In Q1 2025, disk performance troubleshooting with [Microsoft Copilot in Azure](https://azure.microsoft.com/updates?id=474649) became available in public preview. You can use Microsoft Copilot in Azure to analyze [disk metrics](disks-metrics.md) and resolve performance degradation when your application requires more performance than you configured for your virtual machines and disks. For more information, see [Troubleshoot disk performance using Microsoft Copilot in Azure](/azure/copilot/troubleshoot-disk-performance).

## What's new in 2024

### Quarter 4 (October, November, December)

#### Generally available: Convert existing disks to Premium SSD v2 disks in Q4 2024

In Q4 2024, direct conversion from Standard HDD, Standard SSD, and Premium SSD disks to Premium SSD v2 became [generally available](https://azure.microsoft.com/updates/?id=466729). This feature makes it easier to move your workloads to Premium SSD v2 and take advantage of its balance of price and performance. For more information, see [Convert managed disk types to Premium SSD v2](disks-convert-types.md#convert-premium-ssd-v2-disks).

#### Generally available: Expand Ultra Disks and Premium SSD v2 without downtime in Q4 2024

In Q4 2024, expanding Ultra Disks and Premium SSD v2 disks without downtime became [generally available](https://azure.microsoft.com/updates?id=466724). This feature allows you to dynamically increase storage capacity without disrupting existing applications. For instructions, see [Expand a Windows OS disk](windows/expand-disks.md#expand-without-downtime) or [Expand disks on a Linux virtual machine](linux/expand-disks.md#expand-without-downtime).

#### Expanded regional availability for Premium SSD v2 in Q4 2024

In Q4 2024, Premium SSD v2 disks became available in Germany West Central, Israel Central, Italy North, Spain Central, and Mexico Central. For more information, see the [Azure update](https://azure.microsoft.com/updates/v2/generally-available-azure-premium-ssd-v2-disk-storage-is-now-available-in-more-regions).

### Quarter 2 (April, May, June)

#### Generally available: LastOwnershipUpdateTime disk property in Q2 2024

In Q2 2024, the `LastOwnershipUpdateTime` disk property became generally available in the Azure portal, Azure PowerShell, and Azure CLI. This property records when a disk's state last changed. Use it with `diskState` to identify a disk's current state and when that state was last updated. For more information, see the [Azure update](https://azure.microsoft.com/updates/ga-new-property-for-diskslastownershipupdatetime/) or [Find unattached Azure managed and unmanaged disks](/azure/virtual-machines/windows/find-unattached-disks).

### Quarter 1 (January, February, March)

#### Generally available: Azure Backup support for virtual machines with Ultra Disks and Premium SSD v2 in Q1 2024

In Q1 2024, Azure Backup support for virtual machines using Ultra Disks and Premium SSD v2 became generally available in all regions that support those disk types. Ultra Disks and Premium SSD v2 offer high throughput, high IOPS, and low latency. Azure Backup helps ensure business continuity and supports recovery from disasters or ransomware attacks. For more information, see [Azure virtual machine storage support for Azure Backup](/azure/backup/backup-support-matrix-iaas#vm-storage-support).


#### Generally available: Trusted launch support for Ultra Disks and Premium SSD v2 in Q1 2024

In Q1 2024, trusted launch support for virtual machines using Ultra Disks and Premium SSD v2 became generally available. You can combine the foundational compute security of trusted launch with the high throughput, high IOPS, and low latency of Ultra Disks and Premium SSD v2. For more information, see [Trusted launch for Azure virtual machines](trusted-launch.md) or the [Azure update](https://azure.microsoft.com/updates/premium-ssd-v2-and-ultra-disks-support-with-trusted-launch-vm/).

#### Expanded regional availability for Ultra Disks in Q1 2024

In Q1 2024, Ultra Disks became available in UK West and Poland Central.

#### Expanded regional availability for zone-redundant storage disks in Q1 2024

In Q1 2024, zone-redundant storage (ZRS) disks became available in West US 3 and Germany Central.

## What's new in 2023

### Quarter 4 (October, November, December)

#### Generally available: Encryption at host for Premium SSD v2 and Ultra Disks in Q4 2023

In Q4 2023, encryption at host became generally available for Premium SSD v2 and Ultra Disks. It was previously available only for Standard HDDs, Standard SSDs, and Premium SSDs. For more information, see [Encryption at host for virtual machine data](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

Some additional restrictions apply to Premium SSD v2 and Ultra Disks that enable encryption at host. For more information, see [Encryption at host restrictions](disk-encryption.md#encryption-at-host-restrictions).

#### Public preview: New latency metrics in Q4 2023

In Q4 2023, metrics dedicated to monitoring latency became available in public preview. For more information, see [Disk I/O, throughput, queue depth, and latency metrics](disks-metrics.md#disk-io-throughput-queue-depth-and-latency-metrics) or the [Azure update](https://azure.microsoft.com/updates/latency-metrics-for-azure-disks-and-performance-metrics-for-temporary-disks-on-azure-virtual-machines/).

#### Expanded regional availability for Premium SSD v2 in Q4 2023

In Q4 2023, Premium SSD v2 disks became available in Poland Central, China North 3, and US Gov Virginia. For more information, see the [Azure update](https://azure.microsoft.com/updates/generally-available-azure-premium-ssd-v2-disk-storage-is-now-available-in-more-regions-pcu/).


#### Expanded regional availability for zone-redundant storage disks in Q4 2023

In Q4 2023, zone-redundant storage (ZRS) disks became available in Norway East and UAE North. For more information, see the [Azure update](https://azure.microsoft.com/updates/generally-available-zone-redundant-storage-for-azure-disks-is-now-available-in-norway-east-uae-north-regions/).

### Quarter 3 (July, August, September)

#### Expanded regional availability for zone-redundant storage disks in Q3 2023

In Q3 2023, zone-redundant storage (ZRS) disks became available in China North 3, East Asia, India Central, Switzerland North, South Africa North, and Sweden Central.

#### Expanded regional availability for Premium SSD v2 in Q3 2023

In Q3 2023, Premium SSD v2 disks became available in Australia East, Brazil South, Canada Central, Central India, Central US, East Asia, France Central, Japan East, Korea Central, Norway East, South Africa North, Sweden Central, Switzerland North, and UAE North.

#### Generally available: Incremental snapshots for Premium SSD v2 and Ultra Disks in Q3 2023

In Q3 2023, incremental snapshots for Premium SSD v2 and Ultra Disks became generally available. For more information, see [Incremental snapshots of Premium SSD v2 and Ultra Disks](disks-incremental-snapshots.md#incremental-snapshots-of-premium-ssd-v2-and-ultra-disks) or the [Azure update](https://azure.microsoft.com/updates/general-availability-incremental-snapshots-for-premium-ssd-v2-disk-and-ultra-disk-storage-3/).

### Quarter 2 (April, May, June)

#### Expanded regional availability for Premium SSD v2 in Q2 2023

In Q2 2023, Premium SSD v2 disks became available in Southeast Asia, UK South, South Central US, and West US 3.

#### Expanded regional availability for zone-redundant storage disks in Q2 2023

In Q2 2023, zone-redundant storage (ZRS) disks became available in Australia East, Brazil South, Japan East, Korea Central, Qatar Central, UK South, East US, East US 2, South Central US, and Southeast Asia.

#### Public preview: Azure Backup support for Premium SSD v2 in Q2 2023

In Q2 2023, Azure Backup support for Azure virtual machines using Premium SSD v2 disks became available in public preview in East US and West Europe. For more information, see the [Azure update](https://azure.microsoft.com/updates/premium-ssd-v2-backup-support/).

### Quarter 1 (January, February, March)

#### Expanded regional availability for Premium SSD v2 in Q1 2023

In Q1 2023, Premium SSD v2 disks became available in East US 2, North Europe, and West US 2.

#### Public preview: Performance plus in Q1 2023

In Q1 2023, the performance plus feature became available in public preview. Performance plus increases IOPS and throughput limits for Premium SSDs, Standard SSDs, and Standard HDDs that are 513 GiB and larger. For details, see [Increase IOPS and throughput limits for Azure Premium SSDs, Standard SSDs, and Standard HDDs](disks-enable-performance.md).

#### Expanded regional availability for Ultra Disks in Q1 2023

In Q1 2023, Ultra Disks became available in Brazil Southeast, China North 3, Korea South, South Africa North, Switzerland North, and UAE North.

#### More transactions at no extra cost for Standard SSDs in Q1 2023

In Q1 2023, an hourly limit was added to the number of Standard SSD transactions that can incur a billable cost. Transactions beyond that limit don't incur a cost. For more information, see the [announcement](https://aka.ms/billedcapsblog) or [Standard SSD transactions](disks-types.md#standard-ssd-transactions).

#### Generally available: Create disks from snapshots encrypted with customer-managed keys across subscriptions in Q1 2023

In Q1 2023, support for creating disks from snapshots or other disks encrypted with customer-managed keys in different subscriptions within the same tenant became generally available. For more information, see the [Azure update](https://azure.microsoft.com/updates/ga-create-disks-from-cmkencrypted-snapshots-across-subscriptions-and-in-the-same-tenant/) or [Customer-managed keys for Azure managed disks](disk-encryption.md#customer-managed-keys).

#### Generally available: Microsoft Entra ID support for managed disks in Q1 2023

In Q1 2023, Microsoft Entra ID support for securing uploads and downloads of managed disks became generally available. For details, see [Secure downloads and uploads of Azure managed disks](disks-secure-upload-download.md).

## Next steps

- [Azure managed disk types](disks-types.md)
- [Introduction to Azure managed disks](managed-disks-overview.md)

