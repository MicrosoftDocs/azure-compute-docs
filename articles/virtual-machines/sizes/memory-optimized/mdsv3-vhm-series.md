---
title: Mdsv3 Very High Memory size series
description: Information on and specifications of the Mdsv3-VHM-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 04/08/2025
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
---

# Mdsv3 Very High Memory sizes series

[!INCLUDE [mdsv3-vhm-summary](./includes/mdsv3-vhm-series-summary.md)]

## Host specifications
[!INCLUDE [mdsv3-vhm-series-specs](./includes/mdsv3-vhm-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Not Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Not Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-virtual-machine-accelerated-networking): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>[Hibernation](../../hibernate-resume.md): Not Supported <br> [Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GiB) |
| --- | --- | --- |
| Standard_M896ixds_32_v3 | 896 | 30,400 |
| Standard_M1792ixds_32_v3 | 1,792 | 30,400 |

#### VM Basics resources
- [Disable SMT](/sql/sql-server/compute-capacity-limits-by-edition-of-sql-server#limit-number-of-logical-cores-per-numa-node-to-64) to run SQL Server on a VM with more than 64 vCores per NUMA node.
- VHM VM Sizes are virtual machine sizes that are Isolated to a specific hardware type and dedicated to a single customer.
- The Standard_M896ixds_32_v3 VM is the Microsoft recommended VM type with 32TB to host S/4HANA workload. This VM type has Simultaneous Multithreading (SMT) disabled. With that complies with the SAP recommendations stated in SAP note #2711650 for the specific underlying hardware used in Azure to host this Virtual Machine (VM) type. With typical S/4HANA workload, tests this VM realized the best performance.
- The Standard_M1792ixds_32_v3 VM has Simultaneous Multithreading (SMT) enabled and is ideal for analytical workloads as documented in SAP note #2711650 for the specific underlying hardware used in Azure to host this VM type. Typical S/4HANA workloads may show performance regressions compared to the Standard_M896ixds_32_v3 VM type. It is acknowledged that S/4HANA customer workloads are varying and can be different in nature. As a result, there might be cases where S/4HANA customer workloads could eventually benefit. And the Standard_M1792ixds_32_v3 VM type could provide performance improvements compared to the Standard_M896ixds_32_v3 VM type for a customer specific S/4HANA workload. Evaluating and hosting S/4HANA workload on Standard_M1792ixds_32_v3 is in the customer’s own responsibilities.
- It's also important to note that these VMs are compatible with only certain generation 2 Images. For a list of images that are compatible with the Mdsv3-series, please see below
    - Windows Server 2022 Datacenter Edition latest builds
    - SUSE Linux enterprise Server 15 SP4 and later
    - Red Hat Enterprise Linux 8.8 or later
    - Ubuntu 23.10 or later
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) |
| --- | --- | --- |
| Standard_M896ixds_32_v3 | 1 | 4,096 |
| Standard_M1792ixds_32_v3 | 1 | 4,096 |
#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Max Uncached Premium SSD Disk IOPS | Max Uncached Premium SSD Throughput (MB/s) | Max Uncached Ultra Disk and Premium SSD v2 IOPS | Max Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_M896ixds_32_v3 | 64 | 110,000 | 8,000 | 200,000 | 8,000 |
| Standard_M1792ixds_32_v3 | 64 | 110,000 | 8,000 | 200,000 | 8,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.
- <sup>2</sup>Special Storage refers to either [Ultra Disk](../../../virtual-machines/disks-enable-ultra-ssd.md) or [Premium SSD v2](../../../virtual-machines/disks-deploy-premium-v2.md) storage.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_M896ixds_32_v3 | 8 | 185,000 |
| Standard_M1792ixds_32_v3 | 8 | 185,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]
