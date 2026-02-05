---
title: Migrate from Amazon EC2 to Azure Virtual Machines
description: Learn to migrate from Amazon EC2 to Azure Virtual Machines with step-by-step guidance, feature mapping, and validation strategies.
author: mattmcinnes
ms.author: mattmcinnes
ms.topic: how-to
ms.service: learn
ms.date: 02/05/2026
ms.collection:
  - migration
  - aws-to-azure

#customer intent: As a workload architect, I want to understand how to migrate to Azure Virtual Machines from Amazon EC2 as part of my workload's like-for-like migration to Azure. Without this guidance I will miss behavior differences or implementation details that will cause my migration experience delay, frustration, or be to a failure.
---
# Migrate from Amazon EC2 to Azure Virtual Machines

This guide is designed for Amazon Web Services (AWS) professionals that are familiar with Amazon EC2 and are planning to migrate a workload that uses EC2 to Azure Virtual Machines (VMs). It explains key differences and similarities between the two platforms, provides architectural considerations, and outlines best practices for performance, cost, and availability. The goal is to help you plan and perform a migration to Azure’s Infrastructure as a Service (IaaS) environment.

## What you will accomplish

After completing this guide, you will be able to:

- Map Amazon EC2 instance families and sizes to appropriate Azure VM series and SKUs.
- Translate Amazon Machine Image (AMI)-based workloads to Azure Marketplace or custom images.
- Design Azure storage architectures that meet or exceed existing EBS performance characteristics.
- Recreate AWS VPC networking, security, and load balancing patterns using Azure-native services.
- Understand availability, scaling, and placement strategies in Azure that align with AWS designs.
- Learn prerequisite knowledge and skills needed for a successful migration.
- Validate performance, resiliency, and functionality after migration.

### Example scenario: Migrating a production EC2-based application stack

An organization runs a production web application on Amazon EC2 using:

- General purpose EC2 instances in multiple Availability Zones
- Elastic Load Balancer (ALB) for Layer 7 traffic
- Auto Scaling Groups for elasticity
- EBS volumes for persistent storage
- Custom AMIs as a golden image baseline

The goal is to migrate this workload to Azure Virtual Machines while maintaining availability, performance, and scaling characteristics.

### Architectural overview

In AWS, the workload uses EC2 instances distributed across Availability Zones inside a VPC, fronted by an Elastic Load Balancer and scaled using Auto Scaling Groups. Storage is provided by EBS, and images are derived from custom AMIs.

In Azure, the equivalent architecture uses:

- Azure Virtual Machines or Virtual Machine Scale Sets
- Azure Virtual Network with subnets
- Azure Load Balancer or Application Gateway
- Azure Managed Disks
- Azure Marketplace images or custom images stored in Azure Compute Gallery


### Production environment considerations

When transitioning from Amazon EC2 to Azure Virtual Machines, plan for:

- Differences in VM sizing, disk performance coupling, and networking limits
- Image format and agent compatibility (AMIs cannot be imported directly)
- Availability constructs (Availability Zones vs Availability Sets)
- Network security rule evaluation and service integration
- Monitoring, logging, and automation differences


## Step 1: Assessment

Begin by inventorying the existing EC2 workload and capturing how it behaves in production.

Key assessment activities include:

- Identifying EC2 instance families, sizes, and CPU architecture (x86_64 or ARM64)
- Capturing baseline and peak CPU, memory, disk IOPS, throughput, latency, and network usage
- Documenting EBS volume types and performance settings
- Reviewing Auto Scaling Group policies and placement strategies
- Enumerating Security Group and NACL rules
- Identifying AMI sources (public, vendor, or custom)

Formalize your findings by categorizing capabilities as:

- Direct matches in Azure
- Matches with configuration differences
- Capabilities requiring alternative designs


### Direct capability mapping

| Amazon EC2 capability | Azure Virtual Machines equivalent | Migration approach |
|----------------------|-----------------------------------|--------------------|
| EC2 instance families (`t`, `m`, `c`, `r`, `i`, `p`) | Azure VM series (B, D, F, E, L, NC/ND/NP) | Select Azure VM SKUs with equivalent CPU-to-memory ratios and architecture. |
| Auto Scaling Groups | Virtual Machine Scale Sets (VMSS) | Configure VMSS autoscaling and distribute instances across zones. |
| Elastic Load Balancer (ALB/NLB) | Azure Load Balancer / Application Gateway | Map Layer 4 or Layer 7 behavior and health probes. |
| EBS volumes | Azure Managed Disks | Map EBS volume types to appropriate disk SKUs and validate limits. |
| Availability Zones | Azure Availability Zones | Deploy VMs or VMSS instances across zones where supported. |


### Capability mismatches and alternative strategies

Some EC2 capabilities do not translate directly:

- **AMI portability**: Azure does not support lift-and-shift of AMIs. Images must be mapped to Azure Marketplace images or rebuilt as custom images.
- **Bursting behavior**: AWS supports CPU bursting beyond the `t` family. Azure CPU bursting is limited to the B-series.
- **Disk performance scaling**: AWS decouples disk performance from instance size. In Azure, disk performance is constrained by both disk SKU and VM size.
- **Hypervisor access**: EC2 bare-metal instances expose more control than Azure VMs. Use Azure Dedicated Hosts for hardware isolation requirements.


## Step 2: Preparation

Prepare the Azure environment before migrating workloads:

- Design Azure Virtual Networks and subnets aligned with existing VPC segmentation.
- Establish identity and access control using Azure RBAC.
- Decide between Availability Zones and Availability Sets based on workload requirements and regional support.
- Select base Azure Marketplace images that match OS version, architecture, and boot mode.
- Remove AWS-specific agents and ensure Azure VM Agent support.
- Use **Azure Migrate** for discovery, dependency mapping, and right-sizing when migrating multiple VMs.

> **Tip**
>
> Use Azure Migrate when migrating:
> - **5+ uniform VMs** (same OS, similar size, simple dependencies)
> - **3+ heterogeneous VMs** (mixed OS, sizes, or complex dependencies)

## Step 3: Process

Each stage of the migration process requires careful planning and execution to ensure a successful transition from Amazon EC2 to Azure Virtual Machines. This section provides detailed guidance on how to approach the migration of compute, images, storage, networking, and availability features.

### Compute

When migrating from Amazon EC2 to Azure Virtual Machines, understanding how instance families map to Azure VM series is critical for workload planning. Both platforms offer compute resources grouped by performance characteristics, but naming conventions and configuration options differ.

#### AWS EC2 Instance Families
AWS organizes compute resources into instance families based on workload type:
- **General Purpose**: Balanced CPU, memory, and networking (e.g., `t3`, `m5`).
- **Compute Optimized**: High CPU-to-memory ratio for compute-intensive tasks (e.g., `c5`, `c6g`).
- **Memory Optimized**: Large memory footprint for in-memory databases or analytics (e.g., `r5`, `x1e`).
- **Storage Optimized**: High disk throughput for large datasets (e.g., `i3`, `d2`).
- **Accelerated Computing**: GPU-based workloads for ML or graphics (e.g., `p3`, `g4dn`).

AWS instances scale vertically by selecting larger sizes within a family (e.g., `m5.large` → `m5.4xlarge`) and horizontally using Auto Scaling Groups.

#### Azure VM Series
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

#### Key Differences
- **Naming**: AWS uses family + size (e.g., `c5.xlarge`), while Azure uses series + SKU (e.g., `Standard_F4s_v2`).
- **Performance Tiers**: Azure ties disk performance to VM size and disk SKU; AWS uses EBS-optimized instances.
- **Regional Availability**: Both platforms' features vary by region. On Azure, certain features such as **Availability Zones** and **Spot** capacity aren't available in every region.
- **Burstable Options**: AWS has `t` family for burstable workloads and can burst on other eligible sizes; Azure offers [B-series](./sizes/b-series-cpu-credit-model.md) for similar scenarios, and bursting is limited to the B-series.
- **Hypervisor Access**: Some AWS sizes allow for more direct control over the hypervisor (e.g., `i3.metal`); Azure does not expose this level of control.

#### Mapping Strategy
Use the [Azure VM size documentation](./sizes/overview.md) and AWS [instance type guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) to:
1. Identify your EC2 instance family and size.
1. Match to an Azure VM series with equivalent CPU/memory ratio and CPU architecture (x86 or ARM).
1. Validate storage and networking requirements (this is the step that prevents over/under-provisioning):
   - **Baseline in AWS**: capture typical and peak EBS (IOPS/throughput/latency) and network (bandwidth/PPS) usage.
   - **Map to Azure limits**: confirm disk SKU + VM size caps and the VM’s network limits (and whether **[Accelerated Networking](/azure/virtual-network/accelerated-networking-overview)** is supported).
   - **Test in Azure**: run quick storage/network benchmarks before you finalize the VM size.
1. If any accelerators (GPU, FPGA) are used, ensure the Azure VM series supports them.
1. Consider Azure-specific features like [Spot VMs](./spot-vms.md) for cost savings or [VM Scale Sets](../virtual-machine-scale-sets/overview.md) for elasticity.


> [!TIP]
> Use **Azure Migrate** when the migration effort shifts from “a few manual builds” to “repeatable at scale.” As a rule of thumb:
> - **Uniform VMs (same OS + similar size + simple dependencies):** use Azure Migrate when migrating **5+ VMs**.
> - **Different VMs (mixed OS/sizes, multiple disks, or non-trivial dependencies):** use Azure Migrate when migrating **3+ VMs**.
> 
> Azure Migrate helps with discovery, dependency mapping, and right-sizing recommendations so you can avoid over/under-provisioning.

### Images

When migrating workloads that start from an Amazon Machine Image (AMI), plan for an *image translation* step.

> [!IMPORTANT]
> **Azure does not support “lift and shifting” AMIs.** You can’t import an AMI and run it as-is on Azure.
>
> Instead, map **AMI → Azure Marketplace image** (catalog-to-catalog) or **custom AMI → custom Azure image** (custom-to-custom), and then validate drivers, agents, and generation (Gen1/Gen2) compatibility.

#### Find a matching (catalog) image

Start by identifying what the AMI actually represents:
- OS distribution and version (for example, Ubuntu 22.04, RHEL 8.9, Windows Server 2022)
- CPU architecture (x86_64 vs ARM64)
- Boot mode / VM generation assumptions (UEFI/Gen2 vs BIOS/Gen1)
- Installed components (agents, security tools, web/app runtimes)
- Licensing model (BYOL vs marketplace-provided)

Then find an equivalent Azure image:
- **Azure Marketplace images** are the closest match to **public AMIs**.
- Images published by your organization (via **Azure Compute Gallery**) are the closest match to **private/shared AMIs**.

If your AWS workload depends on a vendor AMI (for example, a firewall, appliance, or hardened image), look for the vendor’s equivalent offer in Azure Marketplace and validate:
- Supported VM sizes and required networking features
- Required disk layout and performance
- Licensing and support terms

#### Roll your own (custom-to-custom)

If the AMI is custom (golden image, hardening baseline, preinstalled app), the Azure equivalent is a **custom image** stored and versioned in **Azure Compute Gallery**.

Recommended starting point:
1. **Pick a clean base image** from Azure Marketplace that matches the OS/version/architecture you need.
1. **Automate customization** (packages, configuration, agents, hardening) using a repeatable pipeline.
1. **Validate** the image by deploying test VMs and running smoke tests.
1. **Publish and version** the image in Azure Compute Gallery and replicate it to target regions.

Common tooling patterns:
- **Azure VM Image Builder** (managed image build service) for creating and distributing images.
- **HashiCorp Packer** (including the Azure builders) to build images in CI/CD.
- Configuration management (for example, Ansible/Chef/Puppet) to keep customization reproducible.

Operational requirements to plan for (the “where do you start” checklist):
- **Generalization**
  - Windows: run **Sysprep** before capture.
  - Linux: Install the Azure Linux Agent (**waagent**) before capture.
- **Drivers/agents**: ensure Azure VM agent support and remove AWS-specific agents/tools that don’t apply.
- **VM generation**: decide **Gen1 vs Gen2** early (your base image choice typically determines this).
- **Identity/secrets**: don’t bake secrets into images; use managed identity + Key Vault.
- **Updates**: patch during build and define an image refresh cadence.

### Storage

Storage architecture is a critical factor when migrating from Amazon EC2 to Azure Virtual Machines. Both platforms provide persistent and ephemeral storage options, but implementation and performance models differ.

#### AWS EC2 Storage Options
- **Elastic Block Store (EBS)**  
  Persistent block storage for EC2 instances. Supports SSD and HDD volumes:
  - General Purpose SSD (`gp3`/`gp2`)
  - Provisioned IOPS SSD (`io1`/`io2`)
  - Throughput Optimized HDD (`st1`)
  - Cold HDD (`sc1`)
- **Network-attached storage (NAS)**  
  Shared file storage for Linux and Windows workloads:
  - **Amazon Elastic File System (EFS)**
  - **Amazon FSx** (for example, Windows File Server, NetApp ONTAP, and Lustre)
- **Instance Store**  
  Ephemeral storage physically attached to the host. Data lost on instance stop/terminate.
- **Amazon S3**  
  Object storage for unstructured data, backups, and archival.

Key Features:
- Snapshots for EBS volumes stored in S3.
- Performance tied to volume type and size.
- Encryption via AWS KMS.


#### Azure VM Storage Options
- **Managed Disks**  
  Persistent block storage managed by Azure:
    - **Standard HDD**: Cost-effective for infrequent access, non-production workloads, and long-term backups.
    - **Standard SSD**: Balanced performance for general workloads.
    - **Premium SSD**: Low latency for production and performance sensitive apps.
    - **Ultra Disk**: High throughput for data-intensive workloads.
- **Ephemeral OS Disks**  
  Temporary storage for stateless workloads.
- **Externalized storage (NAS and object storage)**  
  Network-attached and object-based storage for shared file access, backups, archival, and large-scale data:
  - **Azure Blob Storage**
  - **Azure Files**
  - **Azure NetApp Files**

Key Features:
- Disk performance tiers tied to VM size and disk SKU.
- Built-in snapshots and integration with **Azure Backup**.
- Encryption at rest enabled by default; supports customer-managed keys.


#### Architectural Differences
| Feature                | AWS EC2 (EBS)                     | Azure VMs (Managed Disks)                |
|------------------------|-----------------------------------|------------------------------------------|
| Performance Scaling    | Based on volume type and size    | Based on VM size and disk SKU           |
| Snapshot Integration   | Stored in S3                    | Built-in, integrates with Azure Backup  |
| Encryption             | AWS KMS                         | Azure Disk Encryption + Key Vault       |
| Resiliency            | AZ-level replication optional    | Zone-redundant storage available        |

For network-attached storage, AWS **EFS/FSx** map most closely to Azure **Azure Files/Azure NetApp Files**.


#### Storage Migration Considerations
- **Map EBS volumes to Azure Managed Disk tiers**:
  - `gp2/gp3` → Standard SSD  
  - `gp2` → Premium SSD
  - `gp3` → Premium SSD v2  
  - `io2` → Ultra Disk Storage  
- Validate IOPS and throughput requirements; Azure Premium SSD and Ultra Disk support high-performance workloads.
- Plan for encryption compliance: Use Azure Disk Encryption and Key Vault for sensitive data.
- For externalized storage migration:
  - **Amazon S3** → **Azure Blob Storage** using **AzCopy** or **Azure Storage Migration tools**.
  - **Amazon EFS / Amazon FSx** → **Azure Files** (general-purpose file shares) or **Azure NetApp Files** (high-performance NAS).


Validate IOPS and throughput requirements carefully, as Azure disk performance is constrained by both disk SKU and VM size.

### Networking

Networking architecture is a critical component when migrating from Amazon EC2 to Azure Virtual Machines. Both platforms provide secure, isolated networks, but terminology, configuration, and feature sets differ.

#### AWS EC2 Networking
- **Virtual Private Cloud (VPC):** Logical isolation of resources within AWS.
- **Subnets:** Divide VPC into smaller networks for segmentation.
- **Security Groups:** Stateful firewall rules applied at the instance level.
- **Network ACLs:** Stateless rules applied at the subnet level.
- **Elastic IPs:** Static public IP addresses for instances.
- **Load Balancing:** Elastic Load Balancer supports Layer 4 and Layer 7 traffic.
- **Hybrid Connectivity:** VPN and Direct Connect for private links to on-premises.

#### Azure VM Networking
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


#### Key Differences

| Feature                | AWS EC2 (VPC)           | Azure VMs (VNet)                |
|------------------------|-------------------------|---------------------------------|
| Firewall Integration   | Security Groups, NACLs | NSGs + Azure Firewall           |
| Private Service Access | VPC Endpoints          | Private Link                    |
| Hybrid Connectivity    | VPN, Direct Connect    | VPN Gateway, ExpressRoute       |
| Secure Remote Access   | Jump Hosts             | Azure Bastion                   |


#### Migration Considerations
1. **Plan VNet Architecture:** Align with existing AWS VPC design for subnet segmentation.
1. **Security Rules:** Convert AWS Security Group rules to NSGs; review inbound/outbound traffic.
1. **Hybrid Connectivity:** Replace Direct Connect with ExpressRoute for private connectivity.
1. **Load Balancing:** Map ELB configurations to Azure Load Balancer or Application Gateway.
1. **Access Control:** Use Azure Bastion for secure remote access instead of exposing public IPs.

### Clustering, Availability, and Zones

High availability and resiliency strategies differ between AWS EC2 and Azure Virtual Machines. Understanding these concepts is essential for maintaining a fault-tolerant architecture.

#### AWS EC2 Availability Features
- **Availability Zones (AZs):** Isolated locations within a region for redundancy.
- **Auto Scaling Groups:** Automatically adjust instance count based on demand.
- **Elastic Load Balancing (ELB):** Distributes traffic across multiple instances.
- **Placement Groups:** Control instance placement for low-latency or high-throughput workloads.

#### Azure VM Availability Features
- **Availability Zones:** Physically separate datacenters within a region for zone-level resiliency.
- **Availability Sets:** Logical grouping to protect against rack-level failures.
- **Virtual Machine Scale Sets (VMSS):** Built-in orchestration for scaling workloads horizontally.
- **Azure Load Balancer / Application Gateway:** Layer 4 and Layer 7 traffic distribution.


#### Mapping Translation

| Feature                | AWS EC2               | Azure VMs                                   | Migration considerations |
|------------------------|----------------------|---------------------------------------------|--------------------------|
| Zone Redundancy        | AZ-based deployment  | Availability Zones + zone-redundant storage | Map **AWS AZ deployment** to **Azure Availability Zones**. When using VM Scale Sets, **distribute instances across zones** for maximum resiliency. |
| Rack-level Protection  | Not explicit         | Availability Sets                           | **Design for fault domains:** Use **Availability Sets** for rack-level resiliency. |
| Auto Scaling           | Auto Scaling Groups  | VM Scale Sets (VMSS)                        | Map **AWS Auto Scaling Groups** to **Azure VM Scale Sets**. Consider **zone-distributed VMSS** where supported. |
| Load Balancing         | ELB (`L4`/`L7`)       | Azure Load Balancer / App Gateway           | **Set up load balancing:** Replace **ELB** with **Azure Load Balancer** or **Application Gateway**. |

#### Best Practices
- Deploy across **multiple zones** for disaster recovery.
- Use **VMSS with autoscaling policies** for elasticity.
- Enable **zone-redundant storage** for critical data.
- Integrate **Azure Monitor** for health checks and alerting.

### Scaling and Placement

AWS and Azure use different constructs for scaling and placement. Understanding these differences ensures elasticity and fault isolation during migration.

#### AWS Approach
- **Auto Scaling Groups (ASG):**  
  Automatically adjust EC2 instance count based on demand using **Launch Templates**, which define instance configuration and lifecycle (including NICs and disks).
- **Partition Placement Groups:**  
  Spread EC2 instances across multiple partitions for resiliency and low-latency workloads.

#### Azure Approach
- **Virtual Machine Scale Sets (VMSS):**  
  Native orchestration for horizontal scaling, integrated with Azure Load Balancer or Application Gateway.
- **VM Profiles:**  
  Define VM configuration and support **deep deletion** for resource cleanup.
- **Fault Domains and Availability Sets:**  
  Provide rack-level isolation similar to AWS partitioning. For dedicated hardware, use **Azure Dedicated Hosts (ADH)**.

#### Key Differences

| Feature                  | AWS EC2                          | Azure VMs                                  |
|-------------------------|---------------------------------|-------------------------------------------|
| Scaling Construct       | Auto Scaling Groups (ASG)      | VM Scale Sets (VMSS) + AutoScaling       |
| Instance Configuration  | Launch Template                | VM Profile + deep deletion               |
| Placement Strategy      | Partition Placement Groups     | Fault Domains + Availability Sets + ADH  |

## Step 4: Validation

After migration:

- Validate OS boot and agent health.
- Confirm disk throughput, latency, and network performance meet expectations.
- Test scaling behavior and load balancing.
- Validate application functionality and dependencies.
- Configure Azure Monitor alerts and logging.

Successful validation indicates the workload is ready for production cutover.

## Related content

- Azure Virtual Machines overview
- Compare AWS and Azure services
