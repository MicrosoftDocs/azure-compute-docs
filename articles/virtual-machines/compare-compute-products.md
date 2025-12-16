---
title: Compare VM-based compute products
description: Compare different VM-based compute scaling products offered on Azure.
author: mattmcinnes
ms.service: azure-virtual-machines
ms.topic: overview
ms.date: 05/07/2025
ms.author: mattmcinnes
# Customer intent: "As a cloud architect, I want to compare different VM-based compute products, so that I can select the most suitable solution for varying workload requirements and optimize resource allocation for my applications."
---

# Compare virtual machine-based compute products

Microsoft Azure offers various virtual machine based compute products designed to meet different workload requirements. Standard Virtual Machines (VMs), Virtual Machine Scale Sets (VMSS), and Compute Fleet are some of the compute products that provide flexibility and scalability for various applications.

[Virtual Machines (VMs)](/azure/virtual-machines/overview) are the fundamental building blocks of Azure compute. They offer full control over the operating system and applications, making them suitable for a wide range of workloads. VMs provide the flexibility to choose the operating system, size, and configuration that best fits your needs.

[Virtual Machine Scale Sets (VMSS)](/azure/virtual-machine-scale-sets/overview) extend the capabilities of VMs by enabling the deployment and management of a group of VMs. This service is ideal for applications that require high availability or scalability. VMSS can ensure that your applications can handle varying loads efficiently by automatically increasing or decreasing the number of VMs in response to changes in demand.

[Compute Fleet](/azure/azure-compute-fleet/overview) is designed for large-scale, distributed computing environments. It allows you to manage and orchestrate a fleet of VMs across multiple regions and availability zones. This service is particularly beneficial for high-performance computing (HPC) applications, big data processing, and other workloads that require significant computational power and distributed resources.

The product you select depends on the workload you intend to run. Check out the product comparison table in this article or in the Azure portal to choose between these solutions.

## Product comparison table

| | :::image type="icon" source="./media/compare-compute-products/virtual-machines-table-icon.png" border="false"::: Virtual Machines | :::image type="icon" source="./media/compare-compute-products/vm-scale-sets-table-icon.png" border="false"::: Virtual Machine Scale Sets | :::image type="icon" source="./media/compare-compute-products/compute-fleet-table-icon.png" border="false"::: Compute Fleet  |
| --- | --- | --- | --- |
| **Product Description** | Create and manually configure individual VMs.| Load-balanced, automatically scaling VM groups with workload optimization across multiple availability zones. | Provision thousands of mixed-size VMs for performance and high availability. |
| **Instances** | Single instances deployed individually | 2 to 1,000 instances | Up to 10,000 instances |
| **Product Differences**<br><br>See top product capabilities and limitations side by side| - Deploy and manage each VM individually <br>- Fine-tune custom applications<br> - Assigned to one availability zone at a time<br>| - Up to 1,000 mixed-size VMs in a group <br> - Scale automatically <br> - Fault domains for high availability <br>- Integrate Azure Spot VMs to cut costs <br>- Optimized for stateless or stateful workloads | - Up to 10,000 mixed-size VMs in a group<br>- Fault domains for high availability<br>- Hyper-scale with demand <br>- Maintain capacity with Spot VMs to cut costs<br>- Optimize allocation by price, capacity, or both
| **Use Case**<br><br>Find the right product for your project | - Small, simple jobs <br>- Specialized single-instance apps <br>- Test configurations and experiment with Azure | - Scaling beyond a single VM <br>- Platform engineering or infrastructure as code <br>- Moving on-premises apps to the cloud <br>- Parallelized [high-performance computing](/azure/architecture/topics/high-performance-computing) workloads <br>- Databases <br>- Batch processing <br> - Maintain performance while controlling costs <br>- Mix VM sizes for more flexibility | - Large scale, highly parallelized workloads or batch jobs <br> - Flexibility with VM sizes <br> - Large scale cost optimization with Azure Spot |

### Product specifications

#### [Cost](#tab/prodcompcost)

The cost of any Azure VM environment depends upon the number of VM instances, their configuration, and selected add-on services. There's no added charge for using Virtual Machine Scale Sets or Compute Fleet.

| Virtual Machines | Virtual Machine Scale Sets | Compute Fleet  |
| --- | --- | --- |
| - Billed as an individual VM instance <br>- Cost-effective for consistent load and traffic | :::image type="icon" source="./media/compare-compute-products/cost-table-icon.png" border="false"::: **Optimize costs with VMSS** <br>- VMSS can automatically use less capacity when load/traffic is low, and optimizes cost while accommodating the dips and spikes of cyclical, intermittent, and growing workloads <br> - Use 100% Spot VMs, or mix with standard ones, to cut cost | :::image type="icon" source="./media/compare-compute-products/cost-table-icon.png" border="false"::: **Optimize costs with Compute Fleet** <br> - Combine discounts (like reserved instances and savings plans) with Spot VM savings, plus rank the top VM sizes you want Fleet to prioritize <br>- Your fleet will maintain capacity with discounted Spot VMs to optimize cost

#### [Scaling](#tab/prodcompscale)

| Virtual Machines | Virtual Machine Scale Sets | Compute Fleet  |
| --- | --- | --- |
| Vertical Scaling: <br>- Change the VM's size manually <br><br>Horizontal Scaling: <br>- None | Vertical Scaling: <br>- Change any VM's size manually <br><br>Horizontal Scaling: <br>- Manually increase/decrease capacity <br>- Manually add/detach individual VM instances <br>- Autoscale based on demand, schedule, or AI-predicted usage patterns| Vertical Scaling: <br>- Change any VM's size manually <br><br>Horizontal Scaling:<br>- Manually managed through modifying the target capacity |

#### [Management](#tab/prodcompmanagement)

| Virtual Machines | Virtual Machine Scale Sets | Compute Fleet  |
| --- | --- | --- |
Deploy and manage each VM individually | - Deploy and manage VMs as a group, or opt to manage them individually<br>- Orchestrate batch operations (start, stop, restart) and updates (configuration, app, security, and patching) for safer rollouts<br>- Attach and manage one universal add-on for services like data disk and public IP at the VMSS level | - Deploy and manage up to 10,000 VMs as an automated fleet-managed group<br>- Enhance the flexibility, efficiency and control of your VMs, attaching one universal add-on for services like data disk and Public IPs |

#### [Availability](#tab/prodcompavailability)

All VM, Virtual Machine Scale Set, and Compute Fleet products' instance availability are region dependant. For availability of VM sizes in Azure regions, see [Products available by region](https://azure.microsoft.com/regions/services/)

| Virtual Machines | Virtual Machine Scale Sets | Compute Fleet  |
| --- | --- | --- |
Deploy individual VMs, each assigned to an availability zone. To boost uptime, place one or more VMs per availability zone. | Automatically deploy and spread VMs into one (or more) availability zones to boost resilience and uptime | Automatically deploy and distribute VMs across multiple availability zones. Choose specific zones, all eligible zones, or all eligible zones within a given region.

---

## Next steps

Take advantage of the latest performance and features available for your workloads by [changing the size of a virtual machine](/azure/virtual-machines/sizes/resize-vm).

Utilize Microsoft's in-house designed ARM processors with [Azure Cobalt VMs](/azure/virtual-machines/sizes/cobalt-overview).

Learn how to [Monitor Azure virtual machines](/azure/virtual-machines/monitor-vm).
