---
title: HBv5-series VM sizes performance and scalability
description: Learn about performance and scalability of HBv5-series VM sizes in Azure.
services: virtual-machines
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.topic: concept-article
ms.date: 11/05/2025
ms.reviewer: cynthn
ms.author: padmalathas
author: padmalathas
# Customer intent: "As a data engineer, I want to understand the performance and scalability of HBv5-series VMs, so that I can optimize high-performance computing workloads and ensure efficient resource utilization."
---

# HBv5-series virtual machine performance

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Performance expectations using common HPC microbenchmarks are as follows:

| Workload                                        | HBv5                                                              |
|-------------------------------------------------|-------------------------------------------------------------------|
| STREAM Triad                                    | ≈ 6.6 TB/s                                                        |
| High-Performance Linpack (HPL)                  | Up to 16 TF (Rpeak, FP64) for 368-core VM size                    |
| RDMA latency & bandwidth                        | < 2 microseconds (1 byte), 800 Gb/s (one-way)                     |
| FIO on local NVMe SSDs (RAID0)                  | 50 GB/s reads, 30 GB/s writes                                     |

## Process pinning

[Process pinning](./workloads/hpc/compiling-scaling-applications.md#process-pinning) works well on HBv5-series VMs because we expose the underlying silicon as-is to the guest VM. We strongly recommend process pinning for optimal performance and consistency.

## STREAM Memory bandwidth test 

The STREAM memory test can be run using the scripts in this GitHub repository. 
```bash
git clone https://github.com/Azure/woc-benchmarking 
cd woc-benchmarking/apps/hpc/stream/ 
sh build_stream.sh 
sh stream_run_script.sh $PWD “hbrs_v5” 
```
## Compute performance test 

The HPL benchmark can be run using the script in this GitHub repository. 
```bash
git clone https://github.com/Azure/woc-benchmarking 
cd woc-benchmarking/apps/hpc/hpl 
sh hpl_build_script.sh 
sh hpl_run_scr_HBv5.sh $PWD 
```

## MPI latency

The MPI latency test from the OSU microbenchmark suite can be executed as shown. Sample scripts are on [GitHub](https://github.com/Azure/azurehpc/tree/master/apps/health_checks).

```bash
module load mpi/hpcx 
mpirun -np 2 --host $src,$dst --map-by node -x LD_LIBRARY_PATH $HPCX_OSU_DIR/osu_latency
```

> [!NOTE]
> Define source(src) and destination(dst).

## MPI bandwidth

The MPI bandwidth test from the OSU microbenchmark suite can be executed per below. Sample scripts are on [GitHub](https://github.com/Azure/azurehpc/tree/master/apps/health_checks).

```bash
module load mpi/hpcx 
mpirun -np 2 --host $src,$dst --map-by node -x LD_LIBRARY_PATH $HPCX_OSU_DIR/osu_bw
```

## Next steps
- Learn about [scaling MPI applications](./workloads/hpc/compiling-scaling-applications.md).
- Read about the latest announcements, HPC workload examples, and performance results at the [Azure HPC Microsoft Community Hub](https://techcommunity.microsoft.com/t5/azure-high-performance-computing/bg-p/AzureHighPerformanceComputingBlog).
- For a higher-level architectural view of running HPC workloads, see [High Performance Computing (HPC) on Azure](/azure/architecture/topics/high-performance-computing/).
