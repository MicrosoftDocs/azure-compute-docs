---
title: M family VM size series workloads include
description: Include file containing a summary of the 'M' family's potential workloads.
services: virtual-machines
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 10/24/2024
ms.author: mattmcinnes
ms.custom: include file
# Customer intent: "As a cloud architect, I want to evaluate M-family virtual machines, so that I can determine their suitability for high-memory workloads like SQL Server, big data applications, and enterprise systems in our infrastructure."
---
**SQL Server workloads with high memory needs:** The M-family is particularly effective for running SQL Server machines with high memory requirements, such as for [online transaction processing (OLTP)](/azure/architecture/data-guide/relational-data/online-transaction-processing) or [data analytics](/azure/architecture/data-guide/relational-data/online-analytical-processing). 

**In-memory databases:** The M-family is particularly effective for running in-memory databases that require large amounts of RAM like [SQL Server](/sql/relational-databases/in-memory-oltp/overview-and-usage-scenarios), or SAP HANA.

**Big data applications:** The M-family is ideal for handling [big data](/azure/architecture/databases/guide/big-data-architectures) applications that need to process and analyze huge datasets in memory, improving performance and reducing the time to insights.

**Data warehousing:** M-family VMs provide the performance and memory needed for [data warehousing](/azure/architecture/example-scenario/data/data-warehouse) applications, facilitating faster queries and better handling of large volumes of data.

**Enterprise applications:** The M-family supports large-scale enterprise applications, including ERP and CRM systems, which benefit from having more memory to manage larger datasets and more complex transactions efficiently.

**Heavy workloads in virtualized environments:** The M-family is well-equipped to handle heavy virtualized environments, offering substantial memory for hosting multiple virtual machines and applications on a single physical server.
