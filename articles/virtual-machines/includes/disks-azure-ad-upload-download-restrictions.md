---
 title: include file
 description: include file
 author: roygara
 ms.service: azure-disk-storage
 ms.topic: include
 ms.date: 10/17/2023
 ms.author: rogarana
 ms.custom: include file
# Customer intent: "As a cloud administrator, I want to understand the limitations of Azure Backup and Site Recovery regarding disk security, so that I can plan my backup and disaster recovery strategies effectively."
---
- VHDs can't be uploaded to empty snapshots.
- Azure Backup doesn't currently support disks secured with Microsoft Entra ID.
- Azure Site Recovery doesn't currently support disks secured with Microsoft Entra ID.