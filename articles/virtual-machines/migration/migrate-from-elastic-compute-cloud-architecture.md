---
title: Migrate from Amazon Elastic Compute Cloud (Amazon EC2) to Azure Virtual Machines
description: Learn to migrate from Amazon Elastic Compute Cloud (Amazon EC2) to Azure Virtual Machines with step-by-step guidance, feature mapping, and validation strategies.
author: mattmcinnes
ms.author: mattmcinnes
ms.topic: how-to
ms.service: learn
ms.date: 02/20/2026
ms.collection:
  - migration
  - aws-to-azure
ai-usage: ai-assisted

#customer intent: As a workload architect, I want to understand how to migrate to Azure Virtual Machines from Amazon EC2 as part of my workload's like-for-like migration to Azure. Without this guidance I will miss behavior differences or implementation details that will cause my migration experience delay, frustration, or be to a failure.
---
# Migrate from Amazon EC2 to Azure Virtual Machines

If you use Amazon Elastic Compute Cloud (Amazon EC2) and plan to migrate your workload to Azure, this guide can help you understand the migration process, feature mappings, and best practices. It's for Amazon Web Services (AWS) professionals who are familiar with Amazon EC2 and plan to move workloads to Azure Virtual Machines. The guide highlights key similarities and differences between the platforms and outlines important architectural considerations. It also provides best practices for performance, cost, and availability to help you plan and complete a successful migration to an Azure infrastructure as a service (IaaS) environment.

## What you will accomplish

This guide describes how to do the following tasks:

- Map Amazon EC2 instance families and sizes to the right Azure virtual machine (VM) series and SKUs.
- Translate Amazon machine image (AMI)-based workloads to Microsoft Marketplace or custom images.
- Design Azure storage architectures that meet or exceed existing Amazon Elastic Block Store (Amazon EBS) performance characteristics.
- Recreate Amazon Virtual Private Cloud (Amazon VPC) networking, security, and load-balancing patterns by using Azure-native services.
- Learn about availability, scaling, and placement strategies in Azure that align with AWS designs.
- Build the prerequisite knowledge and skills for a successful migration.
- Validate performance, resiliency, and functionality after migration.

## Example scenario: Migrate a production Amazon EC2-based application stack

An organization runs a production web application on Amazon EC2 that uses the following components:

- General-purpose Amazon EC2 instances in multiple availability zones
- Amazon Elastic Load Balancing (Amazon ELB)
- Auto Scaling groups (ASGs) for elasticity
- Amazon EBS volumes for persistent storage
- Custom AMIs as a golden image baseline

The goal is to migrate this workload to Virtual Machines while you maintain availability, performance, and scaling characteristics.

## Use Azure Migrate to migrate your Amazon EC2 instances to Azure

Azure Migrate provides a unified platform to assess and migrate on-premises servers, infrastructure, applications, and data. You can use Azure Migrate to discover, assess, and migrate Amazon EC2 instances to Azure. 

> [!TIP]
> 
> Use Azure Migrate when your migration effort grows beyond a few manual builds and requires a repeatable, scalable approach.
> 
> Use these guidelines to determine when Azure Migrate suits your VM migration:
>
> - For VMs that use the same operating system, have similar sizes, and have simple dependencies, use Azure Migrate when you migrate five or more VMs.
> - For VMs that use different operating systems or sizes, have multiple disks, or have complex dependencies, use Azure Migrate when you migrate three or more VMs.

For more information, see [Migrate Amazon EC2 instances by using Azure Migrate](/azure/migrate/tutorial-migrate-aws-virtual-machines).

### Architectural overview

In AWS, the workload uses Amazon EC2 instances distributed across availability zones inside an Amazon VPC, fronted by an Amazon ELB and scaled by using ASGs. Amazon EBS provides storage, and custom AMIs provide images.

In Azure, the following components form the equivalent architecture:

- Virtual Machines or Azure Virtual Machine Scale Sets
- Azure Virtual Network with subnets
- Azure Load Balancer or Azure Application Gateway
- Azure managed disks
- Marketplace images or custom images stored in Azure Compute Gallery

### Production environment considerations

When you transition from Amazon EC2 to Virtual Machines, plan for the following considerations:

- Differences in VM sizing, disk performance coupling, and networking limits
- Image format and agent compatibility because you can't import AMIs directly
- Availability constructs, like availability zones versus availability sets
- Network security rule evaluation and service integration
- Differences in monitoring, logging, and automation

## Step 1: Plan

Inventory the existing Amazon EC2 workload and capture how it behaves in production.

The key assessment activities include the following items:

- Identify Amazon EC2 instance families, sizes, and CPU architecture, like `x86` or `ARM64`.
- Capture baseline and peak CPU, memory, disk input/output operations per second (IOPS), throughput, latency, and network usage.
- Document Amazon EBS volume types and performance settings.
- Review ASG policies and placement strategies.
- Enumerate security group and network access control list (NACL) rules.
- Identify AMI sources, like public, vendor, or custom images.

Formalize your findings by categorizing each capability into one of the following groups:

- Direct matches in Azure
- Matches that have configuration differences
- Capabilities that require alternative designs

### Direct capability mapping

| Amazon EC2 capability | Azure Virtual Machines equivalent | Migration approach |
|---|---|---|
| Amazon EC2 instance families like `t`, `m`, `c`, `r`, `i`, and `p` | Azure VM series like B, D, F, E, L, and NC, ND, or NP | Select Azure VM SKUs that have equivalent CPU-to-memory ratios and architecture. |
| ASGs | Virtual Machine Scale Sets | Set up autoscaling in Virtual Machine Scale Sets and distribute instances across zones. |
| Amazon ELB: Amazon Application Load Balancer (ALB) and Amazon Network Load Balancer (NLB) | Load Balancer and Application Gateway | Map layer-4 or layer-7 behavior and health probes. |
| Amazon EBS volumes | Azure managed disks | Map Amazon EBS volume types to the right disk SKUs and validate limits. |
| Availability zones | Azure availability zones | Deploy VMs or Virtual Machine Scale Sets instances across zones where supported. |

### Capability mismatches and alternative strategies

Some Amazon EC2 capabilities don't map directly to Azure:

- **AMI portability:** Azure doesn't support a lift-and-shift approach for AMIs. You must map images to Marketplace images or rebuild them as custom images.
- **Bursting behavior:** AWS supports CPU bursts beyond the `t` family. Azure limits CPU bursts to the B-series.
- **Disk performance scale:** AWS decouples disk performance from instance size. In Azure, both disk SKU and VM size constrain disk performance.
- **Hypervisor access:** Amazon EC2 bare-metal instances expose more control than Azure VMs. For hardware isolation requirements in Azure, use Azure Dedicated Host.

## Step 2: Prepare

Prepare the Azure environment before you migrate workloads:

- Design Azure virtual networks and subnets that align with existing Amazon VPC segmentation.
- Establish identity and access control by using Azure role-based access control (Azure RBAC).
- Choose availability zones or availability sets based on your workload requirements and regional support.
- Select base Marketplace images that match the operating system version, architecture, and boot mode.
- Remove AWS-specific agents and ensure Azure VM Agent support.
- Use Azure Migrate for discovery, dependency map creation, and rightsizing recommendations when you migrate multiple VMs.

## Step 3: Execute

Each stage of the migration process requires careful planning and execution to ensure a successful transition from Amazon EC2 to Virtual Machines. This section describes how to migrate compute, images, storage, networking, and availability features.

### Compute

When you migrate from Amazon EC2 to Virtual Machines, you must know how instance families map to Azure VM series to plan your workloads effectively. Both platforms organize their compute offerings by performance characteristics, but their naming conventions and configuration options differ.

#### Amazon EC2 instance families

AWS organizes compute resources into instance families based on workload type:

- **General purpose:** Balanced CPU, memory, and networking, like `t3` and `m5`.
- **Compute optimized:** High CPU-to-memory ratio for compute-intensive tasks, like `c5` and `c6g`.
- **Memory optimized:** Large memory footprint for in-memory databases or analytics, like `r5` and `x1e`.
- **Storage optimized:** High disk throughput for large datasets, like `i3` and `d2`.
- **Accelerated computing:** GPU-based workloads for machine learning or graphics, like `p3` and `g4dn`.

AWS instances scale vertically by selecting larger sizes within a family, like moving from `m5.large` to `m5.4xlarge`, and horizontally by using ASGs.

#### Azure VM series

Azure uses VM series to categorize compute resources:

- **D-series:** General-purpose workloads, similar to the AWS `m` family
- **F-series:** Compute optimized, comparable to the AWS `c` family
- **E-series:** Memory optimized, similar to the AWS `r` family
- **M-series:** Ultra-high memory for SAP HANA and large databases
- **L-series:** Storage optimized with local Non‑Volatile Memory Express (NVMe) disks, and comparable to the AWS `i` family
- **NC, ND, and NP-series:** GPU enabled for AI and machine learning workloads, similar to the AWS `p` and `g` families

Azure defines VM sizes by a SKU, like `Standard_D4s_v5`. The SKU specifies the following properties:

- Virtual CPU (vCPU) count
- Memory, expressed in gibibytes (GiB)
- Temporary storage
- Generation

For more information, see [VM size naming conventions](../vm-naming-conventions.md).

#### Key differences

- **Naming:** AWS uses family and size, like `c5.xlarge`. Azure uses VM series and VM SKU, like `Standard_F4s_v2`.
- **Performance tiers:** Azure sets disk performance according to VM size and disk SKU. AWS uses Amazon EBS-optimized instances.
- **Regional availability:** Features vary by region on both platforms. On Azure, features like availability zones and spot capacity are available only in specific regions.
- **Burstable options:** The AWS `t` family supports burstable workloads, and AWS provides burst capability on other eligible instance sizes. Azure limits bursting to the [B-series](../sizes/b-series-cpu-credit-model.md).
- **Hypervisor access:** Some AWS sizes allow for more direct control over the hypervisor, like `i3.metal`. Azure provides less control at this layer.

#### Mapping strategy

Use the [Azure VM size documentation](../sizes/overview.md) and [AWS instance type guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) to complete the following tasks:

1. Identify your Amazon EC2 instance family and size.
1. Match to an Azure VM series with equivalent CPU-to-memory ratio and the required CPU architecture, like `x86` or `ARM`.
1. Validate storage and network requirements to prevent overprovisioning or underprovisioning:
   - **Establish a baseline in AWS.** Capture typical and peak Amazon EBS IOPS, throughput, and latency, and capture network bandwidth and packets-per-second (PPS) usage.
   - **Map to Azure limits.** Confirm disk SKU and VM size caps and the VM's network limits, including whether it supports [Azure accelerated networking](/azure/virtual-network/accelerated-networking-overview).
   - **Test in Azure.** Run quick storage and network benchmarks before you finalize the VM size.
1. If you use accelerators, like GPU or Field‑Programmable Gate Array (FPGA), ensure that the Azure VM series supports them.
1. Consider Azure-specific features like [spot VMs](../spot-vms.md) for cost savings or [virtual machine scale sets](../../virtual-machine-scale-sets/overview.md) for elasticity.

### Images

When you migrate workloads that start from an AMI, plan for an *image translation* step.

> [!IMPORTANT]
> Azure doesn't support lift and shift of AMIs. You can't import an AMI and run it unchanged on Azure.
>
> Map an AMI to a Marketplace image (catalog to catalog) or map a custom AMI to a custom Azure image (custom to custom) instead. Then validate drivers, agents, and generation compatibility.

#### Find a matching (catalog) image

Start by identifying what the AMI represents:

- Operating system distribution and version, like Ubuntu 22.04, RHEL 8.9, and Windows Server 2022
- CPU architecture, like `x86_64` versus `ARM64`
- Boot mode and VM generation assumptions, like UEFI with Gen2 VMs or BIOS with Gen1 VMs
- Installed components, like monitoring agents, security tools, and web and app runtimes
- Licensing model, like bring-your-own versus marketplace-provided

Then find an equivalent Azure image:

- Marketplace images best match to public AMIs.
- Images that your organization publishes through Compute Gallery are most similar to private or shared AMIs.

If your AWS workload depends on a vendor AMI, like a firewall, appliance, or hardened image, identify the vendor's equivalent offering in Marketplace and confirm that it meets the following requirements:

- Supported VM sizes and required networking features
- Required disk layout and performance
- Licensing and support terms

#### Roll your own (custom-to-custom)

If the AMI is a custom image, like a golden image, a hardened baseline, or an image with a preinstalled app, the Azure equivalent is a custom image that you store and version in Compute Gallery.

Use the following steps as a recommended starting point:

1. **Pick a clean base image** from Marketplace that matches the operating system, version, and architecture that you need.
1. **Automate customization** by applying packages, configuration, agents, and hardening through a repeatable pipeline.
1. **Validate the image** when you deploy test VMs and run smoke tests.
1. **Publish and version the image** in Compute Gallery and replicate it to target regions.

Consider the following common tooling patterns:

- **Azure VM Image Builder** is a managed image build service for image creation and distribution.
- **HashiCorp Packer**, including the Azure builders, is for building images in continuous integration and continuous delivery (CI/CD) pipelines.
- **Configuration management tools** like Ansible, Chef, or Puppet help keep customization reproducible.

Consider these operational requirements to plan your *where do you start* checklist:

- **Generalization:**
  - On Windows, run Sysprep before you capture the image.
  - On Linux, install the Azure Linux VM Agent (`waagent`) before you capture the image.
- **Drivers and agents:** Ensure that the image supports the Azure VM Agent and remove any AWS-specific agents or tools that no longer apply.
- **VM generation:** Choose Gen1 or Gen2 early because your base image choice typically determines the generation.
- **Identity and secrets:** Use managed identity and Azure Key Vault instead of embedding secrets in images.
- **Updates:** Patch the image during build and define an image refresh cadence.

### Storage

Storage architecture is a critical factor when you migrate from Amazon EC2 to Virtual Machines. Both platforms provide persistent and ephemeral storage options, but implementation and performance models differ.

#### Amazon EC2 storage options

- **Amazon EBS:** Persistent block storage for Amazon EC2 instances. Supports solid-state drive (SSD) and hard disk drive (HDD) volumes:
  - General-purpose SSD that uses `gp3` and `gp2`
  - Provisioned IOPS SSD that uses `io1` and `io2`
  - Throughput-optimized HDD that uses `st1`
  - Cold HDD that uses `sc1`
- **Network-attached storage (NAS):** Shared file storage for Linux and Windows workloads:
  - **Amazon Elastic File System (Amazon EFS)**
  - **Amazon FSx**, like Windows File Server, NetApp ONTAP, and Lustre
- **Instance store:** Ephemeral storage that's physically attached to the host. Data is deleted when the instance stops or terminates.
- **Amazon Simple Storage Service (Amazon S3):** Object storage for unstructured data, backups, and archival storage.

Key features of Amazon EC2 storage options include the following items:

- Snapshots for Amazon EBS volumes are stored in Amazon S3.
- Performance depends on volume type and size.
- Encryption via AWS Key Management Service (AWS KMS).

#### Azure VM storage options

- **Managed disks:** Azure manages persistent block storage:
  - **Azure Standard HDD:** Cost effective for infrequent access, nonproduction workloads, and long-term backups.
  - **Azure Standard SSD:** Balanced performance for general workloads.
  - **Azure Premium SSD:** Low latency for production and performance-sensitive apps.
  - **Azure Ultra Disk Storage:** High throughput for data-intensive workloads.
- **Ephemeral OS disks:** Temporary storage for stateless workloads.
- **Externalized storage (NAS and object storage):** Network-attached and object-based storage for shared file access, backups, archival, and large-scale data:
  - Azure Blob Storage
  - Azure Files
  - Azure NetApp Files

Key features of Azure VM storage options include the following items:

- Disk performance tiers depend on VM size and disk SKU.
- Azure VM storage includes built‑in snapshot capabilities and integrates with Azure Backup.
- Encryption at rest uses default keys and supports customer-managed keys.

#### Architectural differences

| Feature | Amazon EC2 (Amazon EBS) | Azure Virtual Machines (managed disks) |
|---|---|---|
| Performance scaling | Based on volume type and size | Based on VM size and disk SKU |
| Snapshot integration | Stored in Amazon S3 | Built-in, integrates with Azure Backup |
| Encryption | AWS KMS | Azure disk encryption and Key Vault |
| Resiliency | Availability zone-level replication optional | Zone-redundant storage (ZRS) available |

For NAS, Amazon EFS and Amazon FSx map most directly to Azure Files and Azure NetApp Files.

#### Storage migration considerations

- Map Amazon EBS volumes to Azure managed disk tiers:
  - The `gp2` and `gp3` volume types map to Standard SSD for light or moderate use.
  - The `gp2` volume type maps to Premium SSD.
  - The `gp3` volume type maps to Premium SSD v2.
  - The `io2` volume type maps to Ultra Disk Storage.
- Validate IOPS and throughput requirements. Premium SSD and Ultra Disk Storage support high-performance workloads.
- Plan for encryption compliance. Use Azure disk encryption and Key Vault for sensitive data.
- For externalized storage migration, you can use the following approaches:
  - Migrate Amazon S3 to Blob Storage by using AzCopy or Azure Storage migration tools.
  - Migrate Amazon EFS and Amazon FSx to Azure Files for general-purpose file shares, or to Azure NetApp Files for high-performance NAS.

Validate IOPS and throughput requirements carefully because both disk SKU and VM size constrain Azure disk performance.

### Networking

Networking architecture is a critical component when you migrate from Amazon EC2 to Virtual Machines. Both platforms provide secure, isolated networks, but terminology, configuration, and feature sets differ.

#### Amazon EC2 networking

- **Amazon VPC:** Logical isolation of resources within AWS.
- **Subnets:** Network segments within an Amazon VPC for isolation and segmentation.
- **Security groups:** Stateful firewall rules applied at the instance level.
- **Network ACLs:** Stateless rules applied at the subnet level.
- **Elastic IP addresses:** Static public IP addresses for instances.
- **Load balancing:** Amazon ELB supports layer-4 and layer-7 traffic.
- **Hybrid connectivity:** Virtual private network (VPN) and AWS Direct Connect for private links to on-premises.

#### Azure VM networking components

- **Virtual network:** An isolated and segmented network environment equivalent to an Amazon VPC.
- **Subnets:** Network segments within a virtual network that provide isolation and support network security groups (NSGs) for traffic filtering.
- **NSGs:** Stateful inbound and outbound traffic rules applied at the subnet or network interface card (NIC) level.
- **Azure Firewall:** A managed, cloud-based firewall used for centralized policy enforcement.
- **Azure Private Link:** Private IP address-based access to Azure services.
- **Public IP addresses:** Static or dynamic public IP addresses that you assign to resources.
- **Load balancing:** Load Balancer at layer 4 and Application Gateway at layer 7 with Secure Sockets Layer (SSL) termination and Azure Web Application Firewall.
- **Azure Bastion:** Secure Remote Desktop Protocol (RDP) and Secure Shell (SSH) access without exposing public IP addresses.
- **Azure ExpressRoute:** Private dedicated connectivity to Azure for hybrid scenarios.

#### Key differences

| Feature | Amazon EC2 (Amazon VPC) | Azure Virtual Machines (virtual network) |
|---|---|---|
| Firewall integration | Security groups and NACLs | NSGs and Azure Firewall |
| Private service access | Amazon VPC endpoints | Private Link |
| Hybrid connectivity | VPN and AWS Direct Connect | Azure VPN Gateway and ExpressRoute |
| Secure remote access | Jump hosts | Azure Bastion |

#### Migration considerations

1. **Plan virtual network architecture:** Align with existing Amazon VPC design for subnet segmentation.
1. **Security rules:** Convert AWS security group rules to NSGs. Review inbound and outbound traffic.
1. **Hybrid connectivity:** Replace AWS Direct Connect with ExpressRoute for private connectivity.
1. **Load balancing:** Map Amazon ELB configurations to Load Balancer or Application Gateway.
1. **Access control:** Use Azure Bastion for secure remote access instead of exposing public IP addresses.

### Clustering, availability, and zones

High-availability and resiliency strategies vary between Amazon EC2 and Virtual Machines. These concepts are essential for a fault-tolerant architecture.

#### Amazon EC2 availability features

- **Availability zones:** Isolated locations within a region for redundancy.
- **ASGs:** Automatic scaling of instance counts based on demand.
- **Amazon ELB:** Traffic distribution across multiple instances.
- **Placement groups:** Instance placement for low-latency or high-throughput workloads.

#### Azure VM availability features

- **Availability zones:** Physically separate datacenters within a region for zone-level resiliency.
- **Availability sets:** Logical grouping to protect against rack-level failures.
- **Virtual Machine Scale Sets:** Built-in orchestration for horizontal workload scale.
- **Load Balancer and Application Gateway:** Layer-4 and layer-7 traffic distribution.

#### Mapping translation

| Feature | Amazon EC2 | Azure Virtual Machines | Migration considerations |
|---|---|---|---|
| Zone redundancy | Availability zone-based deployment | Availability zones and ZRS | Map AWS availability zone deployment to Azure availability zones. When you use Virtual Machine Scale Sets, distribute instances across zones for maximum resiliency. |
| Rack-level protection | Not explicit | Availability sets | Use availability sets for rack-level resiliency. |
| Automatic scaling | ASGs | Virtual Machine Scale Sets | Map AWS ASGs to Virtual Machine Scale Sets. Consider zone-distributed Virtual Machine Scale Sets where supported. |
| Load balancing | Amazon ELB that provides layer-4 and layer-7 capabilities | Load Balancer and Application Gateway | Set up load balancing. Replace Amazon ELB with Load Balancer or Application Gateway. |

#### Best practices

- Deploy across multiple zones for disaster recovery.
- Use Virtual Machine Scale Sets with autoscaling policies for elasticity.
- Use ZRS for critical data.
- Integrate Azure Monitor for health checks and alerting.

### Scaling and placement

AWS and Azure use different constructs for scaling and placement. These differences affect elasticity and fault isolation during migration.

#### AWS approach

- **ASGs:** Automatically adjust Amazon EC2 instance count based on demand by using launch templates, which define instance configuration and life cycle, including NICs and disks.
- **Partition placement groups:** Spread Amazon EC2 instances across multiple partitions for resiliency and low-latency workloads.

#### Azure approach

- **Virtual Machine Scale Sets:** Provides native orchestration for horizontal scaling, integrated with Load Balancer or Application Gateway.
- **VM profiles:** Defines VM configuration and support deep deletion for resource cleanup.
- **Fault domains and availability sets:** Provides rack-level isolation similar to AWS partitioning. For dedicated hardware, use Dedicated Host.

#### Key differences

| Feature | Amazon EC2 | Azure Virtual Machines |
|-----|-----|----|
| Scaling construct | ASGs | Virtual Machine Scale Sets and autoscaling |
| Instance configuration | Launch template | VM configuration profile and deep deletion |
| Placement strategy | Partition placement groups | Fault domains, availability sets, and Dedicated Host |

## Step 4: Evaluate

After the migration completes, do the following validation steps:

- Validate operating system boot and agent health.
- Confirm that disk throughput, latency, and network performance meet expectations.
- Test scaling behavior and load balancing.
- Validate application functionality and dependencies.
- Set up Azure Monitor alerts and logging.

Successful validation indicates that the workload is ready for production cutover.

## Related content

- [Virtual machines overview](/azure/virtual-machines/overview)
- [Azure for AWS professionals](/azure/architecture/aws-professional/)
- [Compare AWS and Azure compute services](/azure/architecture/aws-professional/compute)
- [Migrate a workload from AWS to Azure](/azure/migration/migrate-workload-from-aws-introduction)
