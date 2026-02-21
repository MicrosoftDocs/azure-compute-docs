---
title: Migrate Oracle workloads to Oracle Database@Azure
description: Learn how to migrate Oracle workloads to Oracle Database@Azure
author: suzuber
ms.author: jacobjaygbay
ms.service: oracle-on-azure
ms.topic: concept-article
ms.date: 10/03/2024
# Customer intent: As an IT administrator, I want to migrate Oracle workloads to Oracle Database on Azure, so that I can leverage cloud infrastructure for improved reliability, resilience, and scalability while ensuring minimal disruption to business operations.
---

# Migrate Oracle workloads to Oracle Database@Azure

This article shows how to move your Oracle workload from your on-premises environment to the [Oracle Database@Azure](/azure/oracle/oracle-db/database-overview). A proven discovery, design, and deployment approach are recommended for the overall migration strategy, followed by data migration, and cut over. 

## Discovery

Migration begins with a detailed assessment of the Oracle product portfolio and applications involved. The existing Oracle database can operate Oracle Real Application Clusters (RAC). For applications, we need to discover the size of the infrastructure that can be done easily by using Azure Migrate based discovery. For the database sizing ask your Oracle Sales Representative for advice. 

## Design 

For Applications, [Azure Migrate do lift and shift](/azure/migrate/migrate-services-overview#migration-and-modernization-tool) infrastructure and applications to Azure IaaS based on discovery. The solution must have high [reliability](/azure/reliability/overview) and [resilience](https://azure.microsoft.com/files/Features/Reliability/AzureResiliencyInfographic.pdf) in the occurrence of disasters, as determined by the parameters of [Recovery Point Objective (RPO) and Recovery Time Objective (RTO)](/azure/reliability/disaster-recovery-overview). [Oracle landing zone](/azure/cloud-adoption-framework/scenarios/oracle-iaas/introduction-oracle-landing-zone) offers architecture guidance to choose the best solution architecture based on RPO and RTO requirements. 

## Types for data migration  

The data migration process distinct between physical and logical migration. The logical migration is a method for migrating databases where data and schemas are extracted, transformed, and loaded into the target database. It is particularly useful for scenarios where you need to migrate across different architectures, database versions, or platforms. This approach includes Data Pump and GoldenGate. 
The physical migration involves moving the entire database as-is at the storage or datafile level from a source system to a target system. The database structure remains unchanged. It's ideal when moving to a system with the same database version and compatible architecture. Tools used for this scenario are RMAN and Data Guard.
Please revisit [Oracle logical online migration](https://www.oracle.com/a/otn/docs/database/zdm-logical-online-migration-to-oracle-at-azure.pdf) for further guidance.

## Data migration approach

After you set up Oracle on Azure infrastructure, install Oracle database, and migrate related applications; the next step is to transfer data from on premise Oracle database to the new Oracle database on Azure. See the following Oracle tools: 

- [Recovery Manager (RMAN)](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/getting-started-rman.html)
- [Data Pump](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html)
- [Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/21/sbydb/introduction-to-oracle-data-guard-concepts.html) 
- [GoldenGate](https://docs.oracle.com/goldengate/c1230/gg-winux/GGCON/introduction-oracle-goldengate.htm)
- [ZDM](https://docs.oracle.com/en/solutions/oracle-db-at-azure-migration/index.html)

Azure enhances the Oracle tools with the right network connectivity, bandwidth, and commands that are powered by the following Azure capabilities for data migration.

- [VPN Connectivity](/azure/vpn-gateway/)
- [Express Route](/azure/expressroute/expressroute-introduction). Reliability of the ExpressRoute is the key. Refer to resiliency guidance for [Gateway](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/azure-resources/Network/expressRouteGateways/) and [Circuits](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/azure-resources/Network/expressRouteCircuits/).
- [AzCopy](/azure/storage/common/storage-ref-azcopy)
- [Data Box](/azure/databox/data-box-overview)

**Oracle tools for data migration**

To migrate, there are various options available. The following diagrams provide an overview of these. 

You need one of the Oracle Tools plus Azure infrastructures to deploy the correct solution architecture to migrate data. See the following reference solution scenarios:

Scenario-1: ZDM: Use Zero Downtime Migration, which can be a logical or physical migration based upon your Database version, operating systems and downtime requirements. This setup leverages Data Guard in a logical migration.

:::image type="content" source="./media/oracle-migration/zero-downtime-migration-scenario-1.png" alt-text="Diagram shows the setup for ZDM with Data Guard."Lightbox="./media/oracle-migration/zero-downtime-migration-scenario-1.png":::

Scenario-2: RMAN: Use RMAN backup and restore with Azure features, the setup for RMAN based recovery. The main thing is the network between on-premises and Azure.

:::image type="content" source="./media/oracle-migration/oracle-database-at-azure-rman-direct-migration-scenario-2.png" alt-text="Diagram shows direct migration by use of RMAN."Lightbox="./media/oracle-migration/oracle-database-at-azure-rman-direct-migration-scenario-2.png":::

Scenario-3: RMAN Backup Approach with nfs mount

:::image type="content" source="./media/oracle-migration/oracle-database-at-azure-rman-migration-nfs-storage-scenario-3.png" alt-text="Diagram shows the RMAN backup and restore approach by using a nfs share."Lightbox="./media/oracle-migration/oracle-database-at-azure-rman-migration-nfs-storage-scenario-3.png":::

> [!Note] 
> If you plan to use a private endpoint with a Azure Blob Storage and nfs mount, please make sure to deploy a [local NVA](https://techcommunity.microsoft.com/blog/fasttrackforazureblog/creating-a-local-network-virtual-appliance-in-azure-for-oracle-databaseazure/4218101) into a different subnet within the same VNet as your ODAA relies.

Scenario-4: Data Pump with Azure NetApp Files or Azure Virtual Machine with nfs mount

:::image type="content" source="./media/oracle-migration/oracle-database-at-azure-data-pump-migration-scenario-4.png" alt-text="Diagram shows modified versions of scenario 2."Lightbox="./media/oracle-migration/oracle-database-at-azure-data-pump-migration-scenario-4.png":::
 
> [!Note] 
> If you plan to use a private endpoint with a Azure Blob Storage and nfs mount, please make sure to deploy a [local NVA](https://techcommunity.microsoft.com/blog/fasttrackforazureblog/creating-a-local-network-virtual-appliance-in-azure-for-oracle-databaseazure/4218101) into a different subnet within the same VNet as your ODAA relies.

Scenario-5: Data Box - a unique scenario in which data is moved between the locations using a storage device and physical shipment.

:::image type="content" source="./media/oracle-migration/oracle-database-at-azure-data-box-scenario-5.png" alt-text="Diagram shows data moved between locations using a storage device with physical shipment."Lightbox="./media/oracle-migration/oracle-database-at-azure-data-box-scenario-5.png":::

> [!Note] 
> For planned migration which is performance sensitive, we recommend using Azure Netapp Files.

## Post migration tasks 

After the cutover, verify all business applications are functioning as expected to deliver business operations in tandem with on premise. 

- Perform validation checks to verify data consistency and application functionality.
- Update documentation, including: network diagrams, configuration details, and disaster recovery plans.
- Implement ongoing monitoring and maintenance processes for Azure VM hosting the Oracle database.

Throughout the migration process, it's essential to communicate effectively with stakeholders, including application owners, IT operations teams, and end-users, to manage expectations and minimize disruption. Additionally, consider engaging with experienced professionals or consulting services specializing in Oracle-to-Azure migrations to ensure a smooth and successful transition. 

## Next steps 

[Monitor Oracle Database@Azure](/azure/cloud-adoption-framework/scenarios/oracle-iaas/oracle-manage-monitor-oracle-database-azure) 
