---
title: Migrate from Amazon EC2 to Azure Virtual Machines
description: A comprehensive guide for AWS EC2 professionals transitioning to Azure Virtual Machines. Includes architectural comparisons, best practices, and step-by-step migration strategies.
author: mattmcinnes
ms.topic: article
ms.date: 12/04/2025
ms.author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: migration
---

# Migrating from Amazon EC2 to Azure Virtual Machines

This guide is designed for professionals familiar with Amazon EC2 who are planning to migrate workloads to Azure Virtual Machines (VMs). It explains key differences and similarities between the two platforms, provides architectural considerations, and outlines best practices for performance, cost, and availability. The goal is to help you plan and execute a smooth migration to Azure’s Infrastructure as a Service (IaaS) environment.

## Key concepts
- Comparing EC2 and Azure VM instances
- Architectural considerations for compute, storage, networking, and clustering
- Best practices for performance, cost optimization, and reliability.


## Compute

When migrating from Amazon EC2 to Azure Virtual Machines, understanding how instance families map to Azure VM series is critical for workload planning. Both platforms offer compute resources grouped by performance characteristics, but naming conventions and configuration options differ.

### AWS EC2 Instance Families
AWS organizes compute resources into instance families based on workload type:
- **General Purpose**: Balanced CPU, memory, and networking (e.g., `t3`, `m5`).
- **Compute Optimized**: High CPU-to-memory ratio for compute-intensive tasks (e.g., `c5`, `c6g`).
- **Memory Optimized**: Large memory footprint for in-memory databases or analytics (e.g., `r5`, `x1e`).
- **Storage Optimized**: High disk throughput for large datasets (e.g., `i3`, `d2`).
- **Accelerated Computing**: GPU-based workloads for ML or graphics (e.g., `p3`, `g4dn`).

AWS instances scale vertically by selecting larger sizes within a family (e.g., `m5.large` → `m5.4xlarge`) and horizontally using Auto Scaling Groups.

### Azure VM Series
Azure uses **VM series** to categorize compute resources:
- **D-series**: General-purpose workloads, similar to AWS `m` family.
- **F-series**: Compute-optimized, comparable to AWS `c` family.
- **E-series**: Memory-optimized, similar to AWS `r` family.
- **M-series**: Ultra-high memory for SAP HANA and large databases.
- **L-series**: Storage-optimized with local NVMe disks, comparable to AWS `i` family.
- **NC/ND/NP-series**: GPU-enabled for AI/ML workloads, similar to AWS `p` or `g` families.

Azure VM sizes are defined by **SKU** (e.g., `Standard_D4s_v5`), which specifies:
- vCPU count
- Memory (GiB)
- Temporary storage
- Generation

Check out this article on [VM size naming conventions](./vm-naming-conventions.md) for a more detailed description.

### Key Differences
- **Naming**: AWS uses family + size (e.g., `c5.xlarge`), while Azure uses series + SKU (e.g., `Standard_F4s_v2`).
- **Performance Tiers**: Azure ties disk performance to VM size and disk SKU; AWS uses EBS-optimized instances.
- **Regional Availability**: Both platforms vary by region, but Azure offers **Availability Zones** and **Spot VMs** for cost optimization.
- **Burstable Options**: AWS has `t` family for burstable workloads; Azure offers [B-series](./sizes/b-series-cpu-credit-model.md) for similar scenarios.

### Mapping Strategy
Use the [Azure VM size documentation](./sizes/overview.md) and AWS [instance type guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) to:
1. Identify your EC2 instance family and size.
1. Match to an Azure VM series with equivalent CPU/memory ratio.
1. Validate storage and networking requirements.
1. Consider Azure-specific features like [Spot VMs](./spot-vms.md) for cost savings or [VM Scale Sets](../virtual-machine-scale-sets/overview.md) for elasticity.


> [!TIP]
> For large-scale migrations, leverage **Azure Migrate** to automate instance discovery and size


## Storage

Storage architecture is a critical factor when migrating workloads from Amazon EC2 to Azure Virtual Machines. Both platforms provide persistent and ephemeral storage options, but implementation and performance models differ.

### AWS EC2 Storage Options
- **Elastic Block Store (EBS)**  
  Persistent block storage for EC2 instances. Supports SSD and HDD volumes:
  - General Purpose SSD (`gp3`/`gp2`)
  - Provisioned IOPS SSD (`io1`/`io2`)
  - Throughput Optimized HDD (`st1`)
  - Cold HDD (`sc1`)
- **Instance Store**  
  Ephemeral storage physically attached to the host. Data lost on instance stop/terminate.
- **Amazon S3**  
  Object storage for unstructured data, backups, and archival.

Key Features:
- Snapshots for EBS volumes stored in S3.
- Performance tied to volume type and size.
- Encryption via AWS KMS.


### Azure VM Storage Options
- **Managed Disks**  
  Persistent block storage managed by Azure:
    - **Standard HDD**: Cost-effective for infrequent access.
    - **Standard SSD**: Balanced performance for general workloads.
    - **Premium SSD**: Low latency for mission-critical apps.
    - **Ultra Disk**: High throughput for data-intensive workloads.
- **Ephemeral OS Disks**  
  Temporary storage for stateless workloads.
- **Azure Blob Storage**  
  Object storage for backups, archival, and large-scale data.

Key Features:
- Disk performance tiers tied to VM size and disk SKU.
- Built-in snapshots and integration with **Azure Backup**.
- Encryption at rest enabled by default; supports customer-managed keys.


### Architectural Differences
| Feature                | AWS EC2 (EBS)                     | Azure VMs (Managed Disks)                |
|------------------------|-----------------------------------|------------------------------------------|
| Performance Scaling    | Based on volume type and size    | Based on VM size and disk SKU           |
| Snapshot Integration   | Stored in S3                    | Built-in, integrates with Azure Backup  |
| Encryption             | AWS KMS                         | Azure Disk Encryption + Key Vault       |
| Resiliency            | AZ-level replication optional    | Zone-redundant storage available        |


### Storage Migration Considerations
- **Map EBS volumes to Azure Managed Disk tiers**:
  - `gp3` → Standard SSD
  - `io1`/`io2` → Premium SSD
  - `st1`/`sc1` → Standard HDD
- Validate IOPS and throughput requirements; Azure Premium SSD and Ultra Disk support high-performance workloads.
- Plan for encryption compliance: Use Azure Disk Encryption and Key Vault for sensitive data.
- For object storage migration, map S3 buckets to Azure Blob Storage using **AzCopy** or **Azure Storage Migration tools**.


## Networking

Networking architecture is a critical component when migrating workloads from Amazon EC2 to Azure Virtual Machines. Both platforms provide secure, isolated networks, but terminology, configuration, and feature sets differ.

### AWS EC2 Networking
- **Virtual Private Cloud (VPC):** Logical isolation of resources within AWS.
- **Subnets:** Divide VPC into smaller networks for segmentation.
- **Security Groups:** Stateful firewall rules applied at the instance level.
- **Network ACLs:** Stateless rules applied at the subnet level.
- **Elastic IPs:** Static public IP addresses for instances.
- **Load Balancing:** Elastic Load Balancer supports Layer 4 and Layer 7 traffic.
- **Hybrid Connectivity:** VPN and Direct Connect for private links to on-premises.

### Azure VM Networking
- **Virtual Network (VNet):** Equivalent to AWS VPC, provides isolation and segmentation.
- **Subnets:** Similar concept; supports Network Security Groups (NSGs) for traffic filtering.
- **Network Security Groups (NSGs):** Stateful rules for inbound/outbound traffic at subnet or NIC level.
- **Azure Firewall:** Managed firewall for centralized policy enforcement.
- **Private Link:** Secure access to Azure services over private IP.
- **Public IPs:** Assign static or dynamic public IP addresses.
- **Load Balancing:**  
  - Azure Load Balancer (Layer 4)  
  - Azure Application Gateway (Layer 7 with SSL termination and WAF)
- **Azure Bastion:** Secure RDP/SSH access without exposing public IPs.
- **ExpressRoute:** Private dedicated connectivity to Azure for hybrid scenarios.


### Key Differences

| Feature                | AWS EC2 (VPC)           | Azure VMs (VNet)                |
|------------------------|-------------------------|---------------------------------|
| Firewall Integration   | Security Groups, NACLs | NSGs + Azure Firewall           |
| Private Service Access | VPC Endpoints          | Private Link                    |
| Hybrid Connectivity    | VPN, Direct Connect    | VPN Gateway, ExpressRoute       |
| Secure Remote Access   | Jump Hosts             | Azure Bastion                   |


### Migration Considerations
1. **Plan VNet Architecture:** Align with existing AWS VPC design for subnet segmentation.
2. **Security Rules:** Convert AWS Security Group rules to NSGs; review inbound/outbound traffic.
3. **Hybrid Connectivity:** Replace Direct Connect with ExpressRoute for private connectivity.
4. **Load Balancing:** Map ELB configurations to Azure Load Balancer or Application Gateway.
5. **Access Control:** Use Azure Bastion for secure remote access instead of exposing public IPs.

## Clustering, Availability, and Zones

High availability and resiliency strategies differ between AWS EC2 and Azure Virtual Machines. Understanding these concepts is essential for designing fault-tolerant architectures during migration.

### AWS EC2 Availability Features
- **Availability Zones (AZs):** Isolated locations within a region for redundancy.
- **Auto Scaling Groups:** Automatically adjust instance count based on demand.
- **Elastic Load Balancing (ELB):** Distributes traffic across multiple instances.
- **Placement Groups:** Control instance placement for low-latency or high-throughput workloads.

### Azure VM Availability Features
- **Availability Zones:** Physically separate datacenters within a region for zone-level resiliency.
- **Availability Sets:** Logical grouping to protect against rack-level failures.
- **Virtual Machine Scale Sets (VMSS):** Built-in orchestration for scaling workloads horizontally.
- **Azure Load Balancer / Application Gateway:** Layer 4 and Layer 7 traffic distribution.


### Key Differences

| Feature                | AWS EC2                          | Azure VMs                              |
|------------------------|---------------------------------|----------------------------------------|
| Zone Redundancy        | AZ-based deployment            | Availability Zones + Zone-redundant storage |
| Rack-level Protection  | Not explicit                   | Availability Sets                      |
| Auto Scaling           | Auto Scaling Groups            | VM Scale Sets (VMSS)                  |
| Load Balancing         | ELB (`L4`/`L7`)                   | Azure Load Balancer / App Gateway     |


### Migration Considerations
1. **Map High Availability Strategy:**  
   - AWS Auto Scaling Groups → Azure VM Scale Sets.
   - AWS AZ deployment → Azure Availability Zones.
2. **Design for Fault Domains:**  
   - Use Availability Sets for rack-level resiliency.
3. **Load Balancing:**  
   - Replace ELB with Azure Load Balancer or Application Gateway.
4. **Combine Zones and Scale Sets:**  
   - Distribute VMSS instances across zones for maximum resiliency.


### Best Practices
- Deploy across **multiple zones** for disaster recovery.
- Use **VMSS with autoscaling policies** for elasticity.
- Enable **zone-redundant storage** for critical data.
- Integrate **Azure Monitor** for health checks and alerting.


### Related Resources
- [Azure Virtual Machines Overview](./overview.md)
- [Compare AWS and Azure Services](/architecture/aws-professional)