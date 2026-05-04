---
title: Troubleshooting known issues with HPC and GPU VMs - Azure Virtual Machines | Microsoft Docs
description: Learn about troubleshooting known issues with HPC and GPU virtual machine (VM) sizes in Azure.
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.custom: linux-related-content
ms.topic: troubleshooting
ms.date: 05/04/2026
ms.reviewer: cynthn
ms.author: padmalathas
author: padmalathas
# Customer intent: "As a cloud administrator managing HPC and GPU VMs, I want to troubleshoot known issues and implement solutions, so that I can ensure optimal performance and reliability of my virtual machine workloads."
---

# Known issues with H-series and N-series virtual machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article attempts to list recent common issues and their solutions when using the [HB-series](sizes-hpc.md) and [N-series](sizes-gpu.md) HPC and GPU VMs.

## NVIDIA-SMI Not Showing Full Telemetry on NCv6 (RTX Pro 6000) virtual machines

Running the `nvidia-smi` will not show full telemetry of the RTX Pro 6000 Blackwell GPU(s) on a NCv6-series virtual machine. Specifically, power and utilization statistics will not be exposed. This is due to the use of an SRIOV-based exposure of the GPU(s) to the virtual machine, as opposed to passthrough mode. This is a known limitation of NVIDIA's SRIOV driver supporting "vGPU" functionality.

## AKS Support on NCv6 (RTX Pro 6000) virtual machines

The Azure Kubernetes Service (AKS) is not supported on NCv6-series virtual machines as of May 2026. We are working to bring AKS support to this product in Q4 2026.

## InfiniBand RDMA and NUMA Node Affinity on HBv5 virtual machines

On certain HBv5 virtual machines (VMs), the InfiniBand RDMA device names (such as mlx5_[0-3]) may not align correctly with their respective NUMA node affinities. Ideally, each RDMA device should be mapped as follows:

-	mlx5_0 is on NUMA node: 0
-	mlx5_1 is on NUMA node: 4
-	mlx5_2 is on NUMA node: 8
-	mlx5_3 is on NUMA node: 12
  
However, an incorrect mapping example could be:

- mlx5_0 is on NUMA node: 4
- mlx5_1 is on NUMA node: 8
- mlx5_2 is on NUMA node: 12
- mlx5_3 is on NUMA node: 0
  
This misalignment can lead to performance degradation, particularly when running multinode MPI workloads.

To confirm whether your RDMA devices are correctly mapped to NUMA nodes, execute the following script:
```bash
for d in /sys/class/infiniband/*;
do
dev=$(basename "$d")
node=$(cat "$d/device/numa_node")
echo "$dev is on NUMA node: $node"
done
```
Compare the output with the ideal mapping listed above.

Solution: Persistent device naming with Udev rules

To remediate the misalignment issue, follow these steps:
1.	Create a new file in /etc/udev/rules.d/, for example: 99-rdma-persistent-naming.rules
2.	Add the following lines to the file:
    ```bash
    ACTION=="add", SUBSYSTEMS=="pci", KERNELS=="0101:00:00.0", PROGRAM="rdma_rename %k NAME_FIXED mlx5_ib0"
    ACTION=="add", SUBSYSTEMS=="pci", KERNELS=="0102:00:00.0", PROGRAM="rdma_rename %k NAME_FIXED mlx5_ib1"
    ACTION=="add", SUBSYSTEMS=="pci", KERNELS=="0103:00:00.0", PROGRAM="rdma_rename %k NAME_FIXED mlx5_ib2"
    ACTION=="add", SUBSYSTEMS=="pci", KERNELS=="0104:00:00.0", PROGRAM="rdma_rename %k NAME_FIXED mlx5_ib3"
    ```
3.	Reload udev rules and trigger device events:
    ```bash
    # udevadm control --reload
    # udevadm trigger --type=devices --action=add
    ```
This solution ensures that RDMA device naming persists across VM reboots.

## Cache topology on Standard_HB120rs_v3

`lstopo` displays incorrect cache topology on the Standard_HB120rs_v3 VM size. It may display that there’s only 32 MB L3 per nonuniform memory access (NUMA) node. However, in practice, there's indeed 120 MB L3 per NUMA as expected since the same 480 MB of L3 to the entire VM is available as with the other constrained-core HBv3 VM sizes. This incorrect display is a cosmetic error and shouldn't affect workloads.

## Access Restriction on queue pair 0

To prevent low-level hardware access that can result in security vulnerabilities, Queue Pair 0 isn't accessible to guest VMs. This restriction should only affect actions typically associated with administration of the ConnectX InfiniBand network interface card (NIC) and running some InfiniBand diagnostics like ibdiagnet, but not end-user applications.

## Accelerated Networking on InfiniBand-equipped virtual machines

[Azure Accelerated Networking](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) allows enhanced throughput and latencies over the Azure Ethernet network. Though Accelerated Networking is separate from the InfiniBand network, its use may affect behavior of certain MPI implementations when running jobs over InfiniBand. Specifically, the InfiniBand interface on some VMs may have a slightly different name (mlx5_1 as opposed to earlier mlx5_0). This issue may require tweaking of the MPI command lines, especially when using the UCX interface (commonly with OpenMPI and HPC-X).

For more information on this issue, see the [TechCommunity article with instructions on how to address any observed issues](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-and-hbv2/ba-p/2067965).

## Cache Cleaning

On HPC systems, it's often useful to clean up memory after a job finishes before the next user is assigned the same node. After running applications in Linux, you may find that your available memory reduces while your buffer memory increases, despite not running any applications.

![Screenshot of command prompt before cleaning](./media/hpc/cache-cleaning-1.png)

Using `numactl -H` shows which NUMAnodes the memory is buffered with (possibly all). In Linux, users can clean the caches in three ways to return buffered or cached memory to ‘free’. You need to be root or have sudo permissions.

```bash
sudo echo 1 > /proc/sys/vm/drop_caches [frees page-cache]
sudo echo 2 > /proc/sys/vm/drop_caches [frees slab objects e.g. dentries, inodes]
sudo echo 3 > /proc/sys/vm/drop_caches [cleans page-cache and slab objects]
```

![Screenshot of command prompt after cleaning](./media/hpc/cache-cleaning-2.png)




- Read about the latest announcements, HPC workload examples, and performance results at the [Azure Compute Tech Community Blogs](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- For a higher-level architectural view of running HPC workloads, see [High Performance Computing (HPC) on Azure](/azure/architecture/topics/high-performance-computing/).
