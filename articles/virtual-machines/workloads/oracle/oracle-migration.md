---
title: Migrate Oracle workloads to Azure VMs
description: Learn how to migrate Oracle workloads to Azure VMs.
author: jessiehaessler
ms.author: jhaessler
ms.service: oracle-on-azure
ms.topic: concept-article
ms.date: 10/03/2024
# Customer intent: "As a database administrator, I want to migrate Oracle workloads to Azure VMs, so that I can leverage cloud scalability and maintain business continuity during the transition."
---

# Migrate Oracle workloads to Azure VMs  


This article explains how to migrate your Oracle workload from an on-premises environment to [Azure Virtual Machines (VMs)](/azure/cloud-adoption-framework/scenarios/oracle-iaas/introduction-oracle-landing-zone). It uses the landing zone for [Oracle on Azure VMs](/azure/cloud-adoption-framework/scenarios/oracle-iaas/#landing-zone-architecture-for-oracle-on-azure-virtual-machines), providing design guidance and best practices. The recommended strategy includes a structured approach for discovery, design, and deployment, followed by data migration and final cutover.

:::image type="content" source="media/oracle-migration/azure-virtual-machine-migration.png" alt-text="Screenshot of discovery, design, and deploy migration strategy."Lightbox="media/oracle-migration/azure-virtual-machine-migration.png":::

## Discovery

Migration begins with a comprehensive assessment of the Oracle product portfolio. This assessment includes evaluating the Oracle database versions, the current and target operating systems, as well as the applications and their dependencies.

When you plan to migrate Oracle applications, such as Oracle ([EBS](https://www.oracle.com/in/applications/ebusiness/), [Siebel](https://www.oracle.com/in/cx/siebel/), [PeopleSoft](https://www.oracle.com/in/applications/peoplesoft/), [JDE](https://www.oracle.com/in/applications/jd-edwards-enterpriseone/), or other non-Microsoft partner solutions like [SAP](https://pages.community.sap.com/topics/oracle) or custom applications, please consider the applications as part of the migration strategy.

The existing Oracle database environment may be running on standalone servers, Oracle Real Application Clusters (RAC), or non-Microsoft partner RAC solutions.


> [!Note] 
> Please note that Real Application Clustering (RAC) is not supported on Azure virtual machine. If this applies to your environment, ensure you provide RAC reports or PDB/CDB reports (depending on your architecture) from all RAC nodes. These reports must be generated from the same timeframe to ensure consistency. The most accurate sizing recommendations are obtained by generating these reports during peak usage periods.

For applications, determining the size of your infrastructure is straightforward using Azure Migrate's discovery capabilities.

During the discovery phase, it's essential to review all application dependencies. You should decide whether application downtime is acceptable during the migration, as this influences the choice of migration tools. Based on this decision, you can choose between online or offline migration methods.

If you opt for an online migration, ensure the necessary firewall ports are open to facilitate the migration process.

Network planning is a critical step during the migration period. Be sure to test the bandwidth required to transfer your data to Azure thoroughly, based on the size of your dataset.


## Design 

Application migrations can be seamlessly enabled by using [Azure Migrate](/azure/migrate/migrate-services-overview). Azure Migrate lift-and-shift your application to Azure IaaS based on the initial discovery.

In case you plan to migrate Oracle first-party applications review the [architecture requirements](/azure/virtual-machines/workloads/oracle/deploy-application-oracle-database-azure) before choosing an [Azure Migrate-based migration](https://azure.microsoft.com/products/azure-migrate).

The [Capacity Planning](/azure/cloud-adoption-framework/scenarios/oracle-iaas/oracle-capacity-planning)  for your Oracle database is always conducted through AWR reports which you generating during a one-hour peak-timeframe. 
In addition to that, it's important to set up your [storage layout](/azure/well-architected/oracle-iaas/choose-compute-storage). The data size is the size you need to focus on during the migration and take on the best-suited storage decision.
In order to find out your data size, you can utilize our [dbspace script](https://github.com/Azure/Oracle-Workloads-for-Azure/blob/main/az-oracle-sizing/dbspace.sql). 


Once the AWR reports are generated, run Azure [Oracle Migration Assistance Tool (OMAT)](https://github.com/Azure/Oracle-Workloads-for-Azure/tree/main/omat). 
The OMAT tool recommends the correct VM size and storage options required for your Oracle Database on Azure IaaS. 
As next step establish an architecture by thoroughly assessing your requirements. It's highly recommended to design the architecture highly[reliable](/azure/reliability/overview) and [resilient](https://azure.microsoft.com/files/Features/Reliability/AzureResiliencyInfographic.pdf) in the occurrence of disasters or failures, as determined by the parameters of [Recovery Point Objective (RPO) and Recovery Time Objective (RTO)](/azure/reliability/disaster-recovery-overview).

If you need support establishing the architecture design review the [Oracle reference architectures](/azure/virtual-machines/workloads/oracle/oracle-reference-architecture). It  offers architecture guidance to choose the best solution architecture based on RPO and RTO requirements. The RPO and RTO approach is applicable for separating RAC infrastructure into high availability (HA) and disaster recovery (DR) architecture using Oracle Data Guard.

## Deployment 

Based on your capacity planning and your architecture design, you can use Ansible to describe the infrastructure and architecture as [infrastructure as code (IaC)](/devops/deliver/what-is-infrastructure-as-code) and launch the landing zone with either Terraform or Bicep. Use the [GitHub actions available to automate the deployment](https://github.com/Azure/lza-oracle). 

## Types for data migration  

The type of data migration depends on the decisions made during the discovery phase. You can choose from tools and methods such as Data Box, RMAN, Data Pump, GoldenGate, Striim, SharePlex, and Data Guard based on your preferences and requirements.

For more guidance, refer to [Oracle Migration Planning](/azure/cloud-adoption-framework/scenarios/oracle-iaas/oracle-migration-planning) to review the characteristics of online and offline migrations.

> [!Note]
> Offline migrations typically take longer than online migrations. As a result, tools like Data Pump are not recommended for scenarios involving large data sizes and strict low-downtime requirements.

## Data migration approach

Once your Oracle infrastructure is set up on Azure, the Oracle database is installed, and related applications are migrated, the next step is to transfer data from your on-premises Oracle database to the new Oracle database on Azure. To facilitate this, consider using the following Oracle tools:

- [Recovery Manager (RMAN)](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/getting-started-rman.html)
- [Data Pump](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html)
- [Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/21/sbydb/introduction-to-oracle-data-guard-concepts.html) 
- [GoldenGate](https://docs.oracle.com/goldengate/c1230/gg-winux/GGCON/introduction-oracle-goldengate.htm)

Azure enhances the Oracle tools with the right network connectivity, bandwidth, and commands that are powered by the following Azure capabilities for data migration.

- [VPN Connectivity](/azure/vpn-gateway/)
- [Express Route](/azure/expressroute/expressroute-introduction). Reliability of the ExpressRoute is the key. Refer to resiliency guidance for [Gateway](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/azure-resources/Network/expressRouteGateways/) and [Circuits](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/azure-resources/Network/expressRouteCircuits/).
- [AzCopy](/azure/storage/common/storage-ref-azcopy)
- [Data Box](/azure/databox/data-box-overview)

**Oracle tools for data migration**

The following diagram is a pictographic representation of the overall migration portfolio.

:::image type="content" source="./media/oracle-migration/oracle-migrate-tools.png" alt-text="Diagram shows a pictographic representation of the migration portfolio."Lightbox="./media/oracle-migration/oracle-migrate-tools.png":::

You need one of the Oracle Tools plus Azure infrastructures to deploy the correct solution architecture to migrate data. See the following reference solution scenarios:

Scenario-1: RMAN: Use RMAN backup and restore with Azure features, the setup for RMAN based recovery. The main thing is the network between on-premises and Azure.

:::image type="content" source="./media/oracle-migration/oracle-migrate-diagram-scenario-1.png" alt-text="Diagram shows the setup for RMAN based recovery."Lightbox="./media/oracle-migration/oracle-migrate-diagram-scenario-1.png":::

Scenario-2: RMAN Backup Approach

:::image type="content" source="./media/oracle-migration/rman-backup-approach-scenario-2.png" alt-text="Diagram shows the RMAN backup and restore approach."Lightbox="./media/oracle-migration/rman-backup-approach-scenario-2.png":::
 
Scenario-3: Alternatively, setup can be modified in multiple different ways as depicted in the following scenario.

:::image type="content" source="./media/oracle-migration/rman-backup-approach-scenario-3.png" alt-text="Diagram shows modified versions of scenario 2."Lightbox="./media/oracle-migration/rman-backup-approach-scenario-3.png":::
 
Scenario-4: Data Pump and AzCopy - easy and straight forward approach using Data Pump backup and restore using Azure capabilities.

:::image type="content" source="./media/oracle-migration/data-pump-backup-approach-scenario-4.png" alt-text="Diagram shows Data Pump backup and restore using Azure capabilities."Lightbox="./media/oracle-migration/data-pump-backup-approach-scenario-4.png":::
 
Scenario-5: Data Box - a unique scenario in which data is moved between the locations using a storage device and physical shipment.

:::image type="content" source="./media/oracle-migration/oracle-migrate-diagram-scenario-5.png" alt-text="Diagram shows data moved between locations using a storage device with physical shipment."Lightbox="./media/oracle-migration/oracle-migrate-diagram-scenario-5.png":::

## Cutover

Now your data is migrated and Oracle database servers and applications are up and running. Use the following steps to transition business operations running on premise over to newfound Oracle workload and applications on Azure IaaS.

1. Schedule a maintenance window to minimize disruption to users.
2. Stop database activity on the source Oracle database.
3. Perform a final data synchronization to verify all changes are captured.
4. Update DNS configurations to point to the new Azure VM.
5. Start the Oracle database on the Azure VM and verify connectivity.
6. Monitor the system closely for any issues during the cutover process.

## Post migration tasks 

After the cutover, verify all business applications are functioning as expected to deliver business operations in tandem with on premise. 

- Perform validation checks to verify data consistency and application functionality.
- Update documentation, including: network diagrams, configuration details, and disaster recovery plans.
- Implement ongoing monitoring and maintenance processes for Azure VM hosting the Oracle database.

Throughout the migration process, it's essential to communicate effectively with stakeholders, including application owners, IT operations teams, and end-users, to manage expectations and minimize disruption. Additionally, consider engaging with experienced professionals or consulting services specializing in Oracle-to-Azure migrations to ensure a smooth and successful transition. 

## Next steps 

[Storage options for Oracle on Azure VMs](/azure/virtual-machines/workloads/oracle/oracle-performance-best-practice) 
