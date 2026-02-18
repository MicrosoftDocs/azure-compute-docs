---
title: Migrate from Amazon EC2 to Azure Virtual Machines
description: Learn to migrate from Amazon EC2 to Azure Virtual Machines with step-by-step guidance, feature mapping, and validation strategies.
author: mattmcinnes
ms.author: mattmcinnes
ms.topic: how-to
ms.service: learn
ms.date: 02/20/2026
ms.collection:
  - migration
  - aws-to-azure
ai-usage: ai-assisted

#customer intent: As a workload architect, I want to understand how to migrate from Amazon EC2 to Azure Virtual Machines as part of my workload's like-for-like migration to Azure. Without this guidance, I might overlook behavior differences or implementation details that could introduce delays, cause frustration, or lead to migration failures.
---
# Migrate from Amazon EC2 to Azure Virtual Machines

If you use Amazon EC2 and plan to migrate your workload to Azure, this guide helps you understand the migration process, feature mappings, and recommended practices. It's for Amazon Web Services (AWS) professionals who are familiar with Amazon EC2 and plan to move workloads to Azure Virtual Machines. The guide explains key similarities and differences between the platforms, highlights architectural considerations, and outlines best practices for performance, cost, and availability. The goal is to help you plan and complete a successful migration to an Azure infrastructure as a service (IaaS) environment.

## What you accomplish

After you complete this guide, you can take the following actions:

- Map Amazon EC2 instance families and sizes to appropriate Azure VM series and SKUs.

- Translate Amazon Machine Image (AMI)-based workloads to Microsoft Marketplace or custom images.

- Design Azure storage architectures that meet or exceed existing Amazon Elastic Block Store (EBS) performance characteristics.

- Recreate Amazon Virtual Private Cloud (AWS VPC) networking, security, and load balancing patterns by using Azure-native services.

- Understand availability, scaling, and placement strategies in Azure that align with AWS designs.

- Build the prerequisite knowledge and skills needed for a successful migration.

- Validate performance, resiliency, and functionality after migration.

### Example scenario: Migrate a production Amazon EC2-based application stack

An organization runs a production web application on Amazon EC2 that uses the following components:

- General purpose EC2 instances in multiple availability zones
- Elastic Load Balancer (ELB)
- Auto scaling groups (ASGs) for elasticity
- Amazon EBS volumes for persistent storage
- Custom AMIs as a golden image baseline

The goal is to migrate this workload to Virtual Machines while maintaining availability, performance, and scaling characteristics.

## Use Azure Migrate to migrate your EC2 instances to Azure

Azure Migrate provides a unified platform to assess and migrate on-premises servers, infrastructure, applications, and data to Azure. You can use Azure Migrate to discover, assess, and migrate Amazon Web Services (AWS) EC2 instances to Azure. 

> [!TIP]
> 
> Use Azure Migrate when your migration effort grows beyond a few manual builds and needs a repeatable, scalable approach.
> 
> Use these guidelines to determine when Azure Migrate is a good fit for your VM migration:
>
> For virtual machines (VMs) that use the same operating system, have similar sizes, and have simple dependencies, use Azure Migrate when you migrate five or more VMs.
>
> For VMs that use different operating systems or sizes, have multiple disks, or have complex dependencies, use Azure Migrate when you migrate three or more VMs.

For more information about discovery, dependency mapping, and rightsizing in Azure Migrate, see [Discover, assess, and migrate Amazon Web Services (AWS) EC2 instances to Azure](/azure/migrate/tutorial-migrate-aws-virtual-machines).

### Architectural overview

In AWS, the workload uses EC2 instances distributed across availability zones inside a VPC, fronted by an ELB and scaled by using ASGs. EBS provides storage, and custom AMIs provide images.

In Azure, the equivalent architecture uses:

- Virtual Machines or Azure Virtual Machine Scale Sets
- Azure Virtual Network with subnets
- Azure Load Balancer or Azure Application Gateway
- Azure Managed Disks
- Marketplace images or custom images stored in Azure Compute Gallery

### Production environment considerations

When you transition from Amazon EC2 to Virtual Machines, plan for the following considerations:

- Differences in VM sizing, disk performance coupling, and networking limits
- Image format and agent compatibility (you can't import AMIs directly)
- Availability constructs (availability zones versus availability sets)
- Network security rule evaluation and service integration
- Monitoring, logging, and automation differences

## Step 1: Assessment

Begin by inventorying the existing EC2 workload and capturing how it behaves in production.

The key assessment activities include these items:

- Identify EC2 instance families, sizes, and CPU architecture (x86_64 or ARM64).

- Capture baseline and peak CPU, memory, disk IOPS, throughput, latency, and network usage.

- Document EBS volume types and performance settings.

- Review ASG policies and placement strategies.

- Enumerate security group and network access control list (NACL) rules.

- Identify AMI sources (public, vendor, or custom).

Formalize your findings by categorizing capabilities as:

- Direct matches in Azure
- Matches with configuration differences
- Capabilities requiring alternative designs

### Direct capability mapping

| Amazon EC2 capability | Virtual Machines equivalent | Migration approach |
|---|---|---|
| EC2 instance families (`t`, `m`, `c`, `r`, `i`, `p`) | Azure VM series (B, D, F, E, L, NC/ND/NP) | Select Azure VM SKUs with equivalent CPU-to-memory ratios and architecture. |
| ASGs | Virtual Machine Scale Sets | Set up Virtual Machine Scale Sets autoscaling and distribute instances across zones. |
| Elastic Load Balancer (ALB/NLB) | Load Balancer and Application Gateway | Map layer 4 or layer 7 behavior and health probes. |
| EBS volumes | Azure Managed Disks | Map EBS volume types to appropriate disk SKUs and validate limits. |
| Availability zones | Azure availability zones | Deploy VMs or Virtual Machine Scale Sets instances across zones where supported. |

### Capability mismatches and alternative strategies

Some EC2 capabilities don't translate directly:

- **AMI portability:** Azure doesn't support lift-and-shift of AMIs. You must map images to Marketplace images or rebuild them as custom images.

- **Burst behavior:** AWS supports CPU bursts beyond the `t` family. Azure limits CPU bursts to the B-series.

- **Disk performance scale:** AWS decouples disk performance from instance size. In Azure, both disk SKU and VM size constrain disk performance.

- **Hypervisor access:** EC2 bare-metal instances expose more control than Azure VMs. Use Azure Dedicated Hosts for hardware isolation requirements.

## Step 2: Preparation

Prepare the Azure environment before you migrate workloads:

- Design Azure Virtual Networks and subnets that align with existing VPC segmentation.

- Establish identity and access control with Azure RBAC.

- Decide between availability zones and availability sets based on workload requirements and regional support.

- Select base Marketplace images that match OS version, architecture, and boot mode.

- Remove AWS-specific agents and ensure Azure VM Agent support.

- Use Azure Migrate for discovery, dependency map creation, and right-size recommendations when you migrate multiple VMs.

## Step 3: Process

Each stage of the migration process requires careful plan creation and execution to ensure a successful transition from Amazon EC2 to Virtual Machines. This section provides detailed guidance on how to approach the migration of compute, images, storage, network, and availability features.

### Compute

When you migrate from Amazon EC2 to Virtual Machines, understand how instance families map to Azure VM series is critical for workload plans. Both platforms provide compute resources that are grouped by performance characteristics, but name conventions and configuration options differ.

#### AWS EC2 instance families

AWS organizes compute resources into instance families based on workload type:

- **General purpose:** Balanced CPU, memory, and networking (like `t3`, `m5`).

- **Compute optimized:** High CPU-to-memory ratio for compute-intensive tasks (like `c5`, `c6g`).

- **Memory optimized:** Large memory footprint for in-memory databases or analytics (like `r5`, `x1e`).

- **Storage optimized:** High disk throughput for large datasets (like `i3`, `d2`).

- **Accelerated computing:** GPU-based workloads for ML or graphics (like `p3`, `g4dn`).

AWS instances scale vertically by selecting larger sizes within a family (like `m5.large` → `m5.4xlarge`) and horizontally using ASGs.

#### Azure VM series

Azure uses VM series to categorize compute resources:

- **D-series:** General-purpose workloads, similar to AWS `m` family.

- **F-series:** Compute-optimized, comparable to AWS `c` family.

- **E-series:** Memory-optimized, similar to AWS `r` family.

- **M-series:** Ultra-high memory for SAP HANA and large databases.

- **L-series:** Storage-optimized with local NVMe disks, comparable to AWS `i` family.

- **NC/ND/NP-series:** GPU-enabled for AI/ML workloads, similar to AWS `p` or `g` families.

Azure VM sizes are defined by SKU (like `Standard_D4s_v5`), which specifies:

- vCPU count
- Memory (GiB)
- Temporary storage
- Generation

Check out this article on [VM size naming conventions](../vm-naming-conventions.md) for a more detailed description.

#### Key differences

- **Naming:** AWS uses family and size (like `c5.xlarge`), while Azure uses series and SKU (like `Standard_F4s_v2`).

- **Performance tiers:** Azure ties disk performance to VM size and disk SKU; AWS uses EBS-optimized instances.

- **Regional availability:** Features vary by region on both platforms. On Azure, certain features like availability zones and spot capacity aren't available in every region.

- **Burstable options:** AWS has `t` family for burstable workloads and can burst on other eligible sizes; Azure provides [B-series](../sizes/b-series-cpu-credit-model.md) for similar scenarios, and limits bursting to the B-series.

- **Hypervisor access:** Some AWS sizes allow for more direct control over the hypervisor (like `i3.metal`); Azure doesn't expose this level of control.

#### Mapping strategy

Use the [Azure VM size documentation](../sizes/overview.md) and AWS [instance type guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) to:

1. Identify your EC2 instance family and size.

1. Match to an Azure VM series with equivalent CPU-to-memory ratio and the required CPU architecture (x86 or ARM).

1. Validate storage and network requirements to prevent over-provisioning or under-provisioning:

   - **Establish a baseline in AWS.** Capture typical and peak EBS IOPS, throughput, and latency, and capture network bandwidth and packets-per-second (PPS) usage.
   
   - **Map to Azure limits.** Confirm disk SKU and VM size caps and the VM's network limits, including whether it supports [accelerated networking](/azure/virtual-network/accelerated-networking-overview).
   
   - **Test in Azure.** Run quick storage and network benchmarks before you finalize the VM size.

1. If you use any accelerators (GPU, FPGA), ensure that the Azure VM series supports them.

1. Consider Azure-specific features like [spot VMs](../spot-vms.md) for cost savings or [VM Scale Sets](../../virtual-machine-scale-sets/overview.md) for elasticity.

### Images

When you migrate workloads that start from an AMI, plan for an *image translation* step.

> [!IMPORTANT]
> **Azure doesn't support lift and shift of AMIs.** You can't import an AMI and run it as-is on Azure.
>
> Instead, map **AMI → Marketplace image** (catalog-to-catalog) or **custom AMI → custom Azure image** (custom-to-custom), and then validate drivers, agents, and generation (Gen1/Gen2) compatibility.

#### Find a matching (catalog) image

Start by identifying what the AMI represents:

- OS distribution and version (like Ubuntu 22.04, RHEL 8.9, Windows Server 2022)

- CPU architecture (x86_64 vs ARM64)

- Boot mode and VM generation assumptions, like UEFI with Gen2 or BIOS with Gen1

- Installed components (agents, security tools, web/app runtimes)

- Licensing model (BYOL vs marketplace-provided)

Then find an equivalent Azure image:

- Marketplace images match most closely to public AMIs.

- Images that your organization publishes through Compute Gallery are most similar to private or shared AMIs.

If your AWS workload depends on a vendor AMI (like a firewall, appliance, or hardened image), look for the vendor's equivalent offering in Marketplace and validate:

- Supported VM sizes and required networking features

- Required disk layout and performance

- Licensing and support terms

#### Roll your own (custom-to-custom)

If the AMI is custom (golden image, harden baseline, preinstalled app), the Azure equivalent is a custom image that you store and version in Compute Gallery.

Recommended start point:

1. **Pick a clean base image** from Marketplace that matches the OS, version, and architecture that you need.

1. **Automate customization** by applying packages, configuration, agents, and hardening through a repeatable pipeline.

1. **Validate the image** when you deploy test VMs and run smoke tests.

1. **Publish and version the image** in Compute Gallery and replicate it to target regions.

Common tool patterns:

- **Azure VM Image Builder** is a managed image build service for image creation and distribution.

- **HashiCorp Packer** (includes the Azure builders) to build images in CI/CD.

- Configuration management (like Ansible/Chef/Puppet) to keep customization reproducible.

Operational requirements to plan for (the *where do you start* checklist):

- **Generalization:**

  - *Windows:* Run Sysprep before capture.
  
  - *Linux:* Install the Azure Linux Agent (**waagent**) before capture.

- **Drivers and agents:** Ensure Azure VM agent support and remove AWS‑specific agents or tools that don't apply.

- **VM generation:** Decide Gen1 vs Gen2 early (your base image choice typically determines this).

- **Identity and secrets:** Use managed identity and Azure Key Vault rather than embedding secrets in images.

- **Updates:** Patch during build and define an image refresh cadence.

### Storage

Storage architecture is a critical factor when you migrate from Amazon EC2 to Virtual Machines. Both platforms provide persistent and ephemeral storage options, but implementation and performance models differ.

#### AWS EC2 storage options

- **EBS:** Persistent block storage for EC2 instances. Supports SSD and HDD volumes:
  
  - General purpose SSD (`gp3`/`gp2`)
  - Provisioned IOPS SSD (`io1`/`io2`)
  - Throughput optimized HDD (`st1`)
  - Cold HDD (`sc1`)

- **Network-attached storage (NAS):** Shared file storage for Linux and Windows workloads:

  - **Amazon Elastic File System (EFS)**
  - **Amazon FSx** (like Windows File Server, NetApp ONTAP, and Lustre)

- **Instance store:** Ephemeral storage that is physically attached to the host. Data is lost on instance stop/terminate.

- **Amazon S3:** Object storage for unstructured data, backups, and archival.

Key features:

- Snapshots for EBS volumes are stored in S3.

- Performance depends on volume type and size.

- Encryption via AWS Key Management Service.

#### Azure VM storage options

- **Managed disks:** Azure manages persistent block storage:
  
  - **Standard HDD:** Cost-effective for infrequent access, non-production workloads, and long-term backups.

  - **Standard SSD:** Balanced performance for general workloads.

  - **Premium SSD:** Low latency for production and performance sensitive apps.

  - **Ultra Disk:** High throughput for data-intensive workloads.

- **Ephemeral OS disks:** Temporary storage for stateless workloads.

- **Externalized storage (NAS and object storage):** Network-attached and object-based storage for shared file access, backups, archival, and large-scale data:

  - Azure Blob Storage
  - Azure Files
  - Azure NetApp Files

Key features:

- Disk performance tiers depend on VM size and disk SKU.

- Built-in snapshots and integration with Azure Backup.

- Encryption at rest is enabled by default; supports customer-managed keys.


#### Architectural differences

| Feature | AWS EC2 (EBS) | Azure VMs (Managed Disks) |
|---|---|---|
| Performance scaling | Based on volume type and size | Based on VM size and disk SKU |
| Snapshot integration | Stored in S3 | Built-in, integrates with Azure Backup |
| Encryption | AWS Key Management Service | Azure Disk Encryption and Key Vault |
| Resiliency | Availability zone-level replication optional | Zone-redundant storage available |

For network-attached storage, AWS EFS/FSx map most closely to Azure Files and Azure NetApp Files.

#### Storage migration considerations

- Map EBS volumes to Azure Managed Disk tiers:

  - `gp2/gp3` → Standard SSD - Light/moderate use
  - `gp2` → Premium SSD
  - `gp3` → Premium SSD v2  
  - `io2` → Ultra Disk Storage

- Validate IOPS and throughput requirements; Azure Premium SSD and Ultra Disk support high-performance workloads.

- Plan for encryption compliance: Use Azure Disk Encryption and Key Vault for sensitive data.

- For externalized storage migration:

  - Amazon S3 → Azure Blob Storage by using AzCopy or Azure Storage Migration tools.
  
  - Amazon EFS / Amazon FSx → Azure Files (general-purpose file shares) or Azure NetApp Files (high-performance NAS).


Validate IOPS and throughput requirements carefully because both disk SKU and VM size constrain Azure disk performance.

### Networking

Networking architecture is a critical component when you migrate from Amazon EC2 to Virtual Machines. Both platforms provide secure, isolated networks, but terminology, configuration, and feature sets differ.

#### AWS EC2 networking

- **Virtual Private Cloud (VPC):** Logical isolation of resources within AWS.

- **Subnets:** Divide VPC into smaller networks for segmentation.

- **Security groups:** Stateful firewall rules applied at the instance level.

- **Network ACLs:** Stateless rules applied at the subnet level.

- **Elastic IP addresses:** Static public IP addresses for instances.

- **Load balancing:** Elastic Load Balancer supports layer 4 and layer 7 traffic.

- **Hybrid connectivity:** VPN and Direct Connect for private links to on-premises.

#### Azure VM networking

- **Virtual Network:** Equivalent to AWS VPC, provides isolation and segmentation.

- **Subnets:** Similar concept; supports network security groups (NSGs) for traffic filtering.

- **NSGs:** Stateful rules for inbound/outbound traffic at the subnet or network interface card (NIC)-level.

- **Azure Firewall:** Managed firewall for centralized policy enforcement.

- **Private Link:** Secure access to Azure services over private IP addresses.

- **Public IP addresses:** Static or dynamic public IP addresses that you assign.

- **Load balancing:**  

  - Load Balancer (layer 4)  
  - Application Gateway (layer 7 with SSL termination and WAF)

- **Azure Bastion:** Secure RDP/SSH access without exposing public IP addresses.

- **ExpressRoute:** Private dedicated connectivity to Azure for hybrid scenarios.

#### Key differences

| Feature | AWS EC2 (VPC) | Azure VMs (virtual network) |
|---|---|---|
| Firewall integration | Security groups, NACLs | NSGs and Azure Firewall |
| Private service access | VPC Endpoints | Private Link |
| Hybrid connectivity | VPN, Direct Connect | VPN Gateway, ExpressRoute |
| Secure remote access | Jump Hosts | Azure Bastion |

#### Migration considerations

1. **Plan virtual network architecture:** Align with existing AWS VPC design for subnet segmentation.

1. **Security rules:** Convert AWS security group rules to NSGs; review inbound/outbound traffic.

1. **Hybrid connectivity:** Replace Direct Connect with ExpressRoute for private connectivity.

1. **Load balancing:** Map ELB configurations to Load Balancer or Application Gateway.

1. **Access control:** Use Azure Bastion for secure remote access instead of exposing public IP addresses.

### Clustering, availability, and zones

High availability and resiliency strategies differ between AWS EC2 and Virtual Machines. Understand these concepts is essential to maintain a fault-tolerant architecture.

#### AWS EC2 availability features

- **Availability zones:** Isolated locations within a region for redundancy.

- **ASGs:** Automatically adjust instance count based on demand.

- **Elastic Load balancing (ELB):** Distributes traffic across multiple instances.

- **Placement groups:** Control instance placement for low-latency or high-throughput workloads.

#### Azure VM availability features

- **Availability zones:** Physically separate datacenters within a region for zone-level resiliency.

- **Availability sets:** Logical group to protect against rack-level failures.

- **Virtual Machine Scale Sets:** Built-in orchestration for horizontal workload scale.

- **Load Balancer and Application Gateway:** Layer 4 and layer 7 traffic distribution.

#### Mapping translation

| Feature | AWS EC2 | Azure VMs | Migration considerations |
|---|---|---|---|
| Zone redundancy | Availability zone-based deployment | Availability zones and zone-redundant storage | Map AWS availability zone deployment to Azure availability zones. When you use VM Scale Sets, distribute instances across zones for maximum resiliency. |
| Rack-level protection | Not explicit | Availability sets | Design for fault domains: Use availability sets for rack-level resiliency. |
| Auto scaling | ASGs | VM Scale Sets | Map AWS ASGs to Azure VM Scale Sets. Consider zone-distributed Virtual Machine Scale Sets where supported. |
| Load balancing | ELB (`L4`/`L7`) | Load Balancer and App Gateway | Set up load balancing: Replace ELB with Load Balancer or Application Gateway. |

#### Best practices

- Deploy across multiple zones for disaster recovery.

- Use Virtual Machine Scale Sets with autoscaling policies for elasticity.

- Enable zone-redundant storage for critical data.

- Integrate Azure Monitor for health checks and alerting.

### Scaling and placement

AWS and Azure use different constructs for scale and placement. Understand these differences ensures elasticity and fault isolation during migration.

#### AWS approach

- **ASGs:** Automatically adjust EC2 instance count based on demand by using launch templates, which define instance configuration and life cycle (including NICs and disks).

- **Partition placement groups:** Spread EC2 instances across multiple partitions for resiliency and low-latency workloads.

#### Azure approach

- **Virtual Machine Scale Sets:** Native orchestration for horizontal scaling, integrated with Load Balancer or Application Gateway.

- **VM profiles:** Define VM configuration and support deep deletion for resource cleanup.

- **Fault domains and availability sets:** Provide rack-level isolation similar to AWS partitioning. For dedicated hardware, use Azure Dedicated Hosts.
