---
title: Azure VM Watch
description: Learn how to use the Azure VM watch service offering.
author: iamwilliew 
ms.author: wwilliams
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.update-cycle: 1095-days
ms.date: 01/7/2026
# Customer intent: "As a system administrator, I want to implement VM watch for my virtual machines, so that I can monitor their health efficiently and prevent potential issues with minimal resource impact."
---

# VM watch: Enhancing VM health monitoring 

VMWatch, a standardized, lightweight, and open-sourced testing framework designed to enhance the monitoring and management of guest VMs on the Azure platform, including both first-party (1P) and third-party (3P) instances. VMWatch is engineered to collect vital health signals across multiple dimensions, which will be seamlessly integrated into Azure's quality systems. By leveraging these signals, VMWatch will enable Azure to swiftly detect and prevent regressions induced by platform updates or configuration changes, identify gaps in platform telemetry, and ultimately improve the guest experience for all Azure customers.

VM watch is delivered via the [Application Health VM extension](/azure/virtual-machines/extensions/health-extension?tabs=rest-api) to provide ease of deployment and manageability for customers. In addition, VM watch is offered at no extra cost.

## Monitoring specifics for VM watch

- **Ease of adoption**: VM watch is available through the [Application Health VM extension](/azure/virtual-machines/extensions/health-extension?tabs=rest-api).
- **Flexible deployment**: You can enable VM watch by using an Azure Resource Manager template (ARM template), PowerShell, or the Azure CLI.
- **Compatibility**: VM watch operates seamlessly in both Linux and Windows environments. It's suitable for individual VMs and virtual machine scale sets alike.
- **Resource governance**: VM watch provides efficient monitoring without affecting system performance. Resource caps on the CPU and memory utilization of the VM watch process help protect VMs.
- **Out-of-the-box readiness**: VM watch comes equipped with a suite of default tests that you can configure for your scenarios.

### VM watch Memory Constraints

To maintain consistent performance across different virtual machine (VM) configurations, **VM watch** enforces memory usage limits according to the VM SKU's total available memory. The memory caps are adjusted dynamically based on the VM's memory tier.

| **VM Memory Range** | **VM watch Memory Cap** |
|:---:|:---:|
| Less than 8 GB | 80MB |
| 8 GB to 16 GB | 200MB |
| Greater than 16 GB | 400MB |

### VMWatch Signal Categories

VMWatch measures VM health by emitting signals in different categories, including Network, Disk, CPU, Process, IMDS, Clock, AzBlob, Hardware.

### Network

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Outbound connectivity** | Check | Verify the network outbound connectivity from the Azure VM. |
| **LatencyInNanoSeconds** | Metric | Latency of network outbound connectivity from the Azure VM. |
| **DNS Resolution** | Check | Verify if the dns name(s) can be resolved. |
| **TCPSynRetransmits (Linux Only)** | Metric | The number of times the system retransmits a TCP SYN and SYN/ACK packet before giving up on establishing a connection.|
| **SegmentsRetransmitted** | Metric | The number of transmitted TCP segments that contain one or more previously transmitted octets. |
| **NormalizedSegmentsRetransmitted** | Metric | **SegmentsRetransmitted** / (**SegmentsSent** + **SegmentsRetransmitted**) |
| **ConnectionResets** | Metric | The number of times that TCP connections made a direct transition to the `CLOSED` state from either the `ESTABLISHED` state or the `CLOSE_WAIT` state. |
| **NormalizedConnectionResets** | Metric | The percentage of connections that were reset during the last measurement interval. |
| **FailedConnectionAttempts** | Metric |The number of times that TCP connections made a direct transition to the `CLOSED` state from either the `SYN_SENT` state or the `SYN_RCVD` state. |
| **NormalizedFailedConnectionAttempts** | Metric | **FailedConnectionAttempts** / (**ActiveConnectionOpenings** + **PassiveConnectionOpenings**) |
| **ActiveConnectionOpenings** | Metric | The number of times that TCP connections made a direct transition to the `SYN_SENT` state from the `CLOSED` state. |
| **PassiveConnectionOpenings** | Metric | The number of times that TCP connections made a direct transition to the `SYN_RCVD` state from the `LISTEN` state. |
| **CurrentConnections** | Metric | The number of connections established. |
| **SegmentsReceived** | Metric | The number of segments received, including segments received in error. |
| **SegmentsSent** | Metric | The number of segments sent, including segments on current connections but excluding segments that contain only retransmitted octets. |


### Disk

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Disk I/O** | Check | Verify file creation, write, and read. Delete operations on each drive mounted to the VM. |
| **FreeSpaceInBytes** | Metric | The free disk space of the target mount point. |
| **UsedSpaceInBytes** | Metric | The used disk space of the target mount point. |
| **CapacityInBytes**  | Metric | The disk space capacity of the target mount point. |
| **UsedPercent**      | Metric | The percentage of used disk space for the target mount point. |
| **WriteOps**         | Metric | The write operations per second for the target disk/partition. |
| **ReadOps**          | Metric | The read operations per second for the target disk/partition. |
| **WriteLatencyNs**   | Metric | The write latency in nano seconds of the target disk/partition. | 
| **SyncLatencyNs**   | Metric | 	The write sync latency in nano seconds of the target disk/partition. |
| **ReadLatencyNs**   | Metric | 	The read latency in nano seconds of the target disk/partition. |
| **DiskReadBytesPerSec**| Metric | The read bytes per second of the target disk/partition. |
| **DiskTransfersPerSec**| Metric | The transfer bytes per seconds of the target disk/partition. |

### CPU

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **ProcessCPUCoreUsage** | Metric | An instantaneous measurement of the percentage of a single CPU core that the target process is using (100 = 100%, a whole core). |
| **ProcessCPUMachineUsage** | Metric | The percentage of the machine's total CPU that this process is using. |
| **MachineTotalCpuUsage** | Metric | The VM's total instantaneous CPU utilization. |
| **TotalCpuUsage** | Metric | The VM's total instantaneous CPU utlization. |
| **CpuCount** | Metric | The VM's total number of CPU cores. |
| **PerCore** | Metric | The list of CPU usage of each CPU core. |

### Memory

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **ProcessRSSPercent** | Metric | **Process RSS** / (**Machine Total Memory** * **100%**) |
| **ProcessPageFaults** | Metric | The number of page faults since the process started. |
| **MachineMemoryTotalInBytes** | Metric | The VM's total Memory in Bytes. |
| **MachineMemoryUsedPercent** | Metric | **Machine Used Memory** / (**Machine Total Memory** * **100%**) |
| **TotalPageFaults** | Metric | The total number of page faults for all running processes since they started. |

### Process

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Process** | Check | Start a lightweight process to validate that process creation is possible.Starts a lightweight process to validate that process creation is possible. Process to create: Windows: cmd.exe /c echo hello Linux: /bin/sh -c echo hello |
| **process_monitor** | Check | Verify if the target process or processes are running. |
| **UpTime** | Metric | How long the target process has been up and running since the last process startup. |

### IMDS

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **IMDS** | Check | Verify user can reach of IMDS endpoint from within the VM and VM information is returned from the IMDS endpoint query. |

### Clock

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Clock Skew** | Check | Verify the clock skew between remote NTP server and the Azure VM. For Windows VM, fallback to check if Windows Time Service is sysnced with w32tm if remote NTP server is unaccessible. |

### OS

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **System Errors** | EventLog | Collect the count of error at system level event log (Windows only) whose SystemData <=2 (including LOG_ALWAYS, Critital, Error). The measurementTarget is defined as Source_EventId_ShortHash of the EventLog using default Windows locale. A cap of no more than 10 different measurementTargets is applied in each collection. |
| **vm_blip** | Metric | Measure the elapsed time in milliseconds for the given measurement interval, to detect any VM blip. |

### azblob

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **az_storage_blob** | Check | Verify the connectivity to the Azure Storage Blob and download the Blob with MSI or SAS token. |

### Hardware

| **Signal name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Hardware Health Monitor** | Text | Collect hardware health info from Windows event log, currently only disk related critical events are collected, including events with id 7, 500, 504, 505, 512 and 549. Those events include critical metrics about disk health status, for NVMe directly attached devices, it is exclusively available on the VM side only. With those metric it is possible to monitor and alert the disk status, thus to improve the VM service availabilities. |
| **Hardware Health Nvidia Smi** | Text | Collect GPU stats including memory and GPU usage, temp and other by running nvidia-smi command (Linux Ubuntu only) |

## Related content

- [VM watch Collectors Suite](/azure/virtual-machines/vm-watch-collector-suite)
- [Install VM watch](install-vm-watch.md)
- [Configure VM watch](/azure/virtual-machines/configure-vm-watch)
- [Configure Event Hubs for VM watch](/azure/virtual-machines/configure-eventhub-vm-watch)
