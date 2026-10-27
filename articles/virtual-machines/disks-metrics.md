---
title: Disk metrics
description: Examples of disk bursting metrics
author: roygara
ms.service: azure-disk-storage
ms.topic: concept-article
ms.date: 10/12/2022
ms.author: rogarana
ms.custom: sfi-image-nochange
# Customer intent: "As a cloud operations manager, I want to utilize disk performance metrics for my virtual machines, so that I can monitor and optimize storage performance and diagnose potential bottlenecks effectively."
---
# Disk performance metrics

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure offers metrics in the Azure portal that provide insight on how your virtual machines (VM) and disks perform. The metrics can also be retrieved through an API call. This article is broken into 3 subsections:

- **Disk I/O, throughput, queue depth, and latency metrics** - These metrics help you see the storage performance from the perspective of a disk and a virtual machine.
- **Disk bursting metrics** - These are the metrics provide observability into our [bursting](disk-bursting.md) feature on our premium disks.
- **Storage I/O utilization metrics** - These metrics help diagnose bottlenecks in your storage performance with disks.

Most metrics are emitted every minute. Azure emits bursting credit percentage metrics every five minutes, and **Disk On-demand Burst Operations** every hour.

## Disk I/O, throughput, queue depth, and latency metrics
The following metrics are available to get insight on VM and disk I/O, throughput, and queue depth performance:

- **OS Disk Latency (Preview)**: The average time to complete I/O operations during the monitoring for the OS disk. This metric is only available for disks attached to VMs using the SCSI disk controller and not with disks attached to VMs using the NVMe disk controller. Values are in milliseconds.
- **OS Disk Queue Depth**: The number of current outstanding I/O requests that are waiting to be read from or written to the OS disk.
- **OS Disk Read Bytes/Sec**: The number of bytes that are read in a second from the OS disk. If Read-only or Read/write [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes bytes read from the cache.
- **OS Disk Read Operations/Sec**: The number of input operations that are read in a second from the OS disk. If Read-only or Readwrite [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes IOPS read from the cache.
- **OS Disk Write Bytes/Sec**: The number of bytes that are written in a second from the OS disk.
- **OS Disk Write Operations/Sec**: The number of output operations that are written in a second from the OS disk.
- **Data Disk Latency (Preview)**: The average time to complete I/O operations during the monitoring for the data disk. This metric is only available for disks attached to VMs using the SCSI disk controller and not with disks attached to VMs using the NVMe disk controller. Values are in milliseconds.
- **Data Disk Queue Depth**: The number of current outstanding I/O requests that are waiting to be read from or written to the data disks.
- **Data Disk Read Bytes/Sec**: The number of bytes that are read in a second from the data disks. If Read-only or Readwrite [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes bytes read from the cache.
- **Data Disk Read Operations/Sec**: The number of input operations that are read in a second from data disks. If Read-only or Readwrite [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes IOPS read from the cache.
- **Data Disk Write Bytes/Sec**: The number of bytes that are written in a second from the data disk(s).
- **Data Disk Write Operations/Sec**: The number of output operations that are written in a second from data disk(s).
- **Disk Read Bytes**: The total number of bytes that are read in a minute from all disks attached to a VM. If Read-only or Readwrite [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes bytes read from the cache.
- **Disk Read Operations/Sec**: The number of input operations that are read in a second from all disks attached to a VM. If Read-only or Readwrite [disk caching](premium-storage-performance.md#disk-caching-settings) is enabled, this metric includes IOPS read from the cache.
- **Disk Write Bytes**: The number of bytes that are written in a minute from all disks attached to a VM.
- **Disk Write Operations/Sec**: The number of output operations that are written in a second from all disks attached to a VM.
- **Temp Disk Latency (Preview)**: The average time to complete I/O operations during the monitoring for the temporary disk. This metric isn't available for NVMe temporary storage disks. Values are in milliseconds.
- **Temp Disk Queue Depth**: The number of current outstanding I/O requests that are waiting to be read from or written to the temporary disk. This metric isn't available for NVMe temporary storage disks.
- **Temp Disk Read Bytes/Sec**: The number of bytes that are read in a second from the temporary disk. This metric is not available for NVMe temporary storage disks.
- **Temp Disk Read Operations/Sec**: The number of input operations that are read in a second from the temporary disk. This metric is not available for NVMe temporary storage disks.
- **Temp Disk Write Bytes/Sec**: The number of bytes that are written in a second from the temporary disk. This metric is not available for NVMe temporary storage disks.
- **Temp Disk Write Operations/Sec**: The number of output operations that are written in a second from the temporary disk. This metric is not available for NVMe temporary storage disks.

> [!NOTE]
> Disk metrics can't log CRUD (Create, Read, Update, Delete) operations inside managed disks.

## Bursting metrics
The following metrics help with observability into our [bursting](disk-bursting.md) feature on our premium disks:

- **Data Disk Max Burst Bandwidth**: The throughput limit that the data disk(s) can burst up to.
- **OS Disk Max Burst Bandwidth**: The throughput limit that the OS disk can burst up to.
- **Data Disk Max Burst IOPS**: the IOPS limit that the data disk(s) can burst up to.
- **OS Disk Max Burst IOPS**: The IOPS limit that the OS disk can burst up to.
- **Data Disk Target Bandwidth**: The throughput limit that the data(s) disk can achieve without bursting.
- **OS Disk Target Bandwidth**: The throughput limit that the OS disk can achieve without bursting.
- **Data Disk Target IOPS**: The IOPS limit that the data disk(s) can achieve without bursting.
- **OS Disk Target IOPS**: The IOPS limit that the OS disk can achieve without bursting.
- **Data Disk Used Burst BPS Credits Percentage**: The accumulated percentage of the throughput burst used for the data disk(s). Emitted on a 5 minute interval.
- **OS Disk Used Burst BPS Credits Percentage**: The accumulated percentage of the throughput burst used for the OS disk. Emitted on a 5 minute interval.
- **Data Disk Used Burst IO Credits Percentage**: The accumulated percentage of the IOPS burst used for the data disk(s). Emitted on a 5 minute interval.
- **OS Disk Used Burst IO Credits Percentage**: The accumulated percentage of the IOPS burst used for the OS disk. Emitted on a 5 minute interval.
- **Disk On-demand Burst Operations**: The accumulated operations of burst transactions used for disks with on-demand bursting enabled. Emitted on an hour interval.

## VM bursting metrics
The following metrics provide insight on VM-level bursting:

- **VM Uncached Used Burst IO Credits Percentage**: The accumulated percentage of the VM’s uncached IOPS burst used. Emitted on a 5 minute interval.
- **VM Uncached Used Burst BPS Credits Percentage**: The accumulated percentage of the VM’s uncached throughput burst used. Emitted on a 5 minute interval.
- **VM Cached Used Burst IO Credits Percentage**: The accumulated percentage of the VM’s cached IOPS burst used. Emitted on a 5 minute interval.
- **VM Cached Used Burst BPS Credits Percentage**: The accumulated percentage of the VM’s cached throughput burst used. Emitted on a 5 minute interval.

## Storage I/O utilization metrics
The following metrics help diagnose bottleneck in your Virtual Machine and Disk combination. These metrics are only available on VM series that support premium storage.

Metrics that help diagnose disk I/O capping:

- **Data Disk IOPS Consumed Percentage**: The percentage calculated by dividing the actual data disk IOPS completed by the provisioned data disk IOPS. If this value reaches 100%, your application is I/O capped by your data disk's IOPS limit.
- **Data Disk Bandwidth Consumed Percentage**: The percentage calculated by dividing the actual data disk throughput completed by the provisioned data disk throughput. If this value reaches 100%, your application is I/O capped by your data disk's bandwidth limit.
- **OS Disk IOPS Consumed Percentage**: The percentage calculated by dividing the actual OS disk IOPS completed by the provisioned OS disk IOPS. If this value reaches 100%, your application is I/O capped by your OS disk's IOPS limit.
- **OS Disk Bandwidth Consumed Percentage**: The percentage calculated by dividing the actual OS disk throughput completed by the provisioned OS disk throughput. If this value reaches 100%, your application is I/O capped by your OS disk's bandwidth limit.

Metrics that help diagnose VM I/O capping:

- **VM Cached IOPS Consumed Percentage**: The percentage calculated by dividing the total actual cached IOPS completed by the max cached virtual machine IOPS limit. If this value reaches 100%, your application is I/O capped by your VM's cached IOPS limit.
- **VM Cached Bandwidth Consumed Percentage**: The percentage calculated by dividing the total actual cached throughput completed by the max cached virtual machine throughput. If this value reaches 100%, your application is I/O capped by your VM's cached bandwidth limit.
- **VM Uncached IOPS Consumed Percentage**: The percentage calculated by dividing the total actual uncached IOPS on a virtual machine completed by the max uncached virtual machine IOPS limit. If this value reaches 100%, your application is I/O capped by your VM's uncached IOPS limit.
- **VM Uncached Bandwidth Consumed Percentage**: The percentage calculated by dividing the total actual uncached throughput on a virtual machine completed over the max provisioned virtual machine throughput. If this value reaches 100%, your application is I/O capped by your VM's uncached bandwidth limit.

## Storage I/O metrics example

Let's run through an example of how to use these new Storage I/O utilization metrics to help debug where a bottleneck is in the system. The system setup is the same as the previous example, except this time the attached OS disk isn't cached.

**Setup:**

- Standard_D8s_v3
  - Cached IOPS: 16,000
  - Uncached IOPS: 12,800
- P30 OS disk
  - IOPS: 5,000
  - Host caching: **Disabled**
- Two P30 data disks × 2
  - IOPS: 5,000
  - Host caching: **Read/write**
- Two P30 data disks × 2
  - IOPS: 5,000
  - Host caching: **Disabled**

Let's run a benchmarking test on this virtual machine and disk combination that creates I/O activity. To learn how to benchmark storage I/O on Azure, see [Benchmark your application on Azure Disk Storage](disks-benchmarks.md). From the benchmarking tool, you can see that the VM and disk combination can achieve 22,800 IOPS:

![Screenshot of FIO output with a read rate of 22,800 IOPS highlighted.](media/disks-metrics/utilization-metrics-example/fio-output.jpg)



The Standard_D8s_v3 can achieve a total of 28,800 IOPS. Using the metrics, let's investigate what's going on and identify our storage I/O bottleneck. On the left pane, select **Metrics**:

![Screenshot of the IO-Utilization-Demo virtual machine overview with Metrics under Monitoring highlighted.](media/disks-metrics/utilization-metrics-example/metrics-menu.jpg)

Let's first take a look at our **VM Cached IOPS Consumed Percentage** metric:

![Screenshot of the VM Cached IOPS Consumed Percentage metric with an average value of 61 percent.](media/disks-metrics/utilization-metrics-example/vm-cached.jpg)

This metric tells us that 61% of the 16,000 IOPS allotted to the cached IOPS on the VM is being used. This percentage means that the storage I/O bottleneck isn't with the disks that are cached because it isn't at 100%. Now let's look at our **VM Uncached IOPS Consumed Percentage** metric:

![Screenshot of the VM Uncached IOPS Consumed Percentage metric with an average value of 101 percent.](media/disks-metrics/utilization-metrics-example/vm-uncached.jpg)

This metric is at 100%. It tells us that all of the 12,800 IOPS allotted to the uncached IOPS on the VM are being used. One way to remediate this issue is to change the size of our VM to a larger size that can handle the additional I/O. But before we do that, let's look at the attached disk to find out how many IOPS they are seeing. Check the OS Disk by looking at the **OS Disk IOPS Consumed Percentage**:

![Screenshot of the OS Disk IOPS Consumed Percentage metric with an average value of 87 percent.](media/disks-metrics/utilization-metrics-example/os-disk.jpg)

This metric tells us that around 90% of the 5,000 IOPS provisioned for this P30 OS disk is being used. This percentage means there's no bottleneck at the OS disk. Now let's check the data disks that are attached to the VM by looking at the **Data Disk IOPS Consumed Percentage**:

![Screenshot of the Data Disk IOPS Consumed Percentage metric averaged across all attached data disks, with Apply splitting highlighted.](media/disks-metrics/utilization-metrics-example/data-disks-no-splitting.jpg)

This metric tells us that the average IOPS consumed percentage across all the disks attached is around 42%. This percentage is calculated based on the IOPS that are used by the disks, and aren't being served from the host cache. Let's drill deeper into this metric by applying *splitting* on these metrics and splitting by the LUN value:

![Screenshot of Data Disk IOPS Consumed Percentage split by LUN, showing 89 percent for LUN 3, 82 percent for LUN 2, and 0 percent for LUNs 1 and 0.](media/disks-metrics/utilization-metrics-example/data-disks-splitting.jpg)

This metric tells us the data disks attached on LUN 3 and 2 are using around 85% of their provisioned IOPS. Here is a diagram of what the I/O looks like from the VM and disks architecture:

![Diagram of a Standard_D8s_v3 VM satisfying a 22,800-IOPS application request through one P30 OS disk, two uncached P30 data disks, and two cached P30 data disks.](media/disks-metrics/utilization-metrics-example/metrics-diagram.jpg)

## Next steps

- [Azure Monitor Metrics overview](/azure/azure-monitor/essentials/data-platform-metrics)
- [Metrics aggregation explained](/azure/azure-monitor/essentials/metrics-aggregation-explained)
- [Create, view, and manage metric alerts using Azure Monitor](/azure/azure-monitor/alerts/alerts-metric)
