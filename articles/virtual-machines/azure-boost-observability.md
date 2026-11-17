---
title: Azure Boost observability for NVMe disks
description: Learn how to use Azure-Boost-Observability to query disk performance metrics and identify NVMe namespaces on compatible Azure virtual machines.
author: audreyroberts30
ms.service: azure-virtual-machines
ms.date: 10/27/2026
ms.author: auroberts
ms.topic: how-to
ai-usage: ai-assisted

# Customer intent: As a cloud infrastructure engineer, I want to query disk performance metrics and identify NVMe disks on Azure virtual machines, so that I can investigate disk performance and distinguish the disks presented to the guest operating system.
---

# Azure Boost observability for NVMe disks

Azure-Boost-Observability is an open-source command-line utility that exposes detailed remote disk performance metrics and disk identification data for NVMe™ disks on compatible Azure virtual machines (VMs). You can query performance metrics to examine remote disk I/O activity and latency, or identify disks by their name, type, size, and namespace identifier. The tool provides text and JSON-formatted output.

Use the tool from inside your VM to investigate disk performance and identify the disks presented to the guest operating system.

Azure-Boost-Observability supports Windows and Linux guest operating systems. This article provides a complete workflow for each guest OS. For an introduction to NVMe on Azure VMs, see [NVMe overview](nvme-overview.md).

## Features

- **Performance statistics**: Query performance counters and statistics from NVMe drives.
- **Device identification**: Identify NVMe namespaces and map them from your guest VM to the Azure portal.
- **Batch processing**: Query all NVMe drives in the system automatically.
- **Verbose output**: Get optional detailed logging for troubleshooting.

## Disk performance metrics

The tool reads performance metrics for remote data disks from log pages on the NVMe device. These metrics include cumulative read and write operation counts, bytes transferred, outstanding I/O requests, and read and write latency.

| Property | Description | Time span |
| --- | --- | --- |
| `total read IOPs` | The total read IOPs completed during the disk's lifecycle. | Disk lifetime |
| `total write IOPs` | The total write IOPs completed during the disk's lifecycle. | Disk lifetime |
| `total read bytes` | The total read bytes completed during the disk's lifecycle. | Disk lifetime |
| `total written bytes` | The total written bytes completed during the disk's lifecycle. | Disk lifetime |
| `outstanding IO` | The total IO requests outstanding in the disk. | 5-second sampling interval |
| `read_latency_max` (µs) | The maximum read latency observed during the sampling interval. | 5-second sampling interval |
| `read_latency_avg` (µs) | The average read latency observed between IOPs during the sampling interval. | 5-second sampling interval |
| `write_latency_max` (µs) | The maximum write latency observed during the sampling interval. | 5-second sampling interval |
| `write_latency_avg` (µs) | The average write latency observed between IOPs during the sampling interval. | 5-second sampling interval |
| `read_dispatch_latency_max` (µs) | The maximum read disk latency during the sampling interval. | 5-second sampling interval |
| `read_dispatch_latency_avg` (µs) | The average read disk latency observed in the disk scheduler during the sampling interval. | 5-second sampling interval |
| `write_dispatch_latency_max` (µs) | The maximum write disk latency during the sampling interval. | 5-second sampling interval |
| `write_dispatch_latency_avg` (µs) | The average write disk latency observed in the disk scheduler during the sampling interval. | 5-second sampling interval |
| `is degraded` | Indicates if the system isn't in hardware-accelerated mode. | Current accelerator state |

The total operation counts are cumulative counters, not I/O operations per second (IOPS).

## Disk identification

The tool provides a consistent way to correlate the disk shown in the Azure portal with the corresponding disk inside the guest OS by using the Identify Namespace command. The command returns the following fields:

| Field | JSON property | Description |
| --- | --- | --- |
| Disk | Object key | The Windows disk number or drive letter, or Linux device path, identifying the queried device. |
| NGUID | `NGUID` | The 128-bit Namespace Globally Unique Identifier reported by the NVMe device, formatted as a GUID. Displays `undefined` if no identifier is provided. |
| Disk Size | `DiskSizeGiB` | The namespace size in gibibytes (GiB), calculated from the number of logical blocks and the logical block size. One GiB is (1024^3) bytes. |
| Vendor Specific | `VS` | Disk identification data returned as a single string of comma-separated `key=value` pairs. Keys are `type` (disk role: `unknown`, `data`, `os`, or `local`) and `name` (disk name). If the string is empty, Windows displays `undefined` and Linux returns an empty string. |

## Prerequisites and availability

Use an Azure VM that's compatible with Azure-Boost-Observability and has NVMe disks attached. The following VM sizes are currently compatible:

- Intel: Dlsv7, Dldsv7, Dsv7, Ddsv7, Esv7, and Edsv7.
- Cobalt 200: Dplsv7, Dpldsv7, Dpsv7, Dpdsv7, Epsv7, Epdsv7, Mpsv4, Mpdsv4, and Lpsv5.

Performance metrics are available for remote disks in all Azure regions for compatible VM sizes. Disk identification covers both remote and local NVMe disks and is currently available in Early Updates Access Program (EUAP) regions, with availability in additional regions planned for 2027.

You might need administrative privileges for direct NVMe device access.

Get the source files from the [Azure-Boost-Observability GitHub repository](https://github.com/microsoft/Azure-Boost-Observability).

## Use the tool on Windows

Follow these steps to prepare your Windows VM and query disk metrics or identification data.

### Set up the tool

Build the Windows executable on your VM or on another Windows computer. If you build on another computer, copy the executable to your VM.

These steps build the tool for a Windows VM with an x64 guest OS by using the MSVC v143 toolset. The Visual Studio IDE is optional.

1. On the build computer, install the **MSVC v143 C++ x64/x86 build tools**, MSBuild, and a Windows SDK, either through Visual Studio 2022 or the standalone Build Tools for Visual Studio. In the installer, select the **Desktop development with C++** workload and ensure that the v143 tools are included. For instructions, see [Install C and C++ support in Visual Studio](/cpp/build/vscpp-step-0-installation) or [Use the Microsoft C++ Build Tools from the command line](/cpp/build/building-on-the-command-line).
1. Download and extract the source files from the [Azure-Boost-Observability GitHub repository](https://github.com/microsoft/Azure-Boost-Observability).
1. Build the executable by using either Visual Studio or the command line:

    **Visual Studio (if installed):** Open `AzureBoostObservability-Win\AzureBoostObservability.sln` from the extracted source folder. In the solution toolbar, select **Release** and **x64**, then select **Build** > **Build Solution**.

    **Command line:** Alternatively, if you prefer a command-line approach, open a **Developer Command Prompt** from your Visual Studio or Build Tools installation, then change to the extracted source folder. Use the following syntax, replacing `<configuration>` with `Release` or `Debug` and `<platform>` with `x64` or `x86`:

    ```text
    msbuild .\AzureBoostObservability-Win\AzureBoostObservability.sln /m /p:Configuration=<configuration> /p:Platform=<platform>
    ```

    For the x64 Windows VM in this procedure, build the release executable:

    ```powershell
    msbuild .\AzureBoostObservability-Win\AzureBoostObservability.sln /m /p:Configuration=Release /p:Platform=x64
    ```

1. After the build succeeds, locate `AzureBoostObservability-Win\out\x64\Release\Azure-Boost-Observability.exe` under the extracted source folder. If you built on another computer, copy the executable to your VM.
1. On the VM, open PowerShell as an administrator and change to the folder containing `Azure-Boost-Observability.exe`.

The following examples assume that you're in the folder containing the executable. To display the extended usage, run the executable with no parameters or specify `/help`:

```powershell
.\Azure-Boost-Observability.exe /help
```

### Query metrics on Windows

Use the performance query command to retrieve the values described in [Disk performance metrics](#disk-performance-metrics).

In an elevated PowerShell session, specify the physical disk number or drive letter. You can confirm the disk number in **Disk Management**.

To query disk 0, run:

```powershell
.\Azure-Boost-Observability.exe /perf 0
```

Sample text output:

```output
Disk: 0
  Degraded performance: No
  Total Ops
    Read Ops: 98060
    Write Ops: 21870
  Total Bytes
    Read Bytes: 3584886272
    Write Bytes: 851041792
  Outstanding I/Os: 48
  Ops Latency Max/Avg (us)
    Read Latency Max: 3883
    Read Latency Avg: 2168
    Write Latency Max: 29565
    Write Latency Avg: 6231
  Dispatch Latency Max/Avg (us)
    Read Dispatch Latency Max: 1
    Read Dispatch Latency Avg: 0
    Write Dispatch Latency Max: 1
    Write Dispatch Latency Avg: 1
```

To get JSON-formatted output, specify `/json` before `/perf`:

```powershell
.\Azure-Boost-Observability.exe /json /perf 0
```

Sample JSON output:

```json
{
    "0": {
        "is_degraded": false,
        "outstanding_io": 48,
        "read_dispatch_latency_avg": 0,
        "read_dispatch_latency_max": 1,
        "read_latency_avg": 2168,
        "read_latency_max": 3883,
        "total_read_bytes": 3584886272,
        "total_read_ops": 98060,
        "total_write_bytes": 851041792,
        "total_write_ops": 21870,
        "write_dispatch_latency_avg": 1,
        "write_dispatch_latency_max": 1,
        "write_latency_avg": 6231,
        "write_latency_max": 29565
    }
}
```

You can target a drive letter instead of a disk number:

```powershell
.\Azure-Boost-Observability.exe /perf D
```

To query disks without specifying a target, run:

```powershell
.\Azure-Boost-Observability.exe /perf
```

The tool queries disks in order starting at disk 0. It checks up to 32 disks and stops at the first device it can't query. To query a disk that isn't reached in this scan, specify its disk number explicitly.

For detailed logging, put `/verbose` before the command:

```powershell
.\Azure-Boost-Observability.exe /verbose /perf 0
```

### Identify disks on Windows

Use the identification command to retrieve the fields described in [Disk identification](#disk-identification).

To identify disk 0, run:

```powershell
.\Azure-Boost-Observability.exe /identify 0
```

To get JSON-formatted output, specify `/json` before `/identify`:

```powershell
.\Azure-Boost-Observability.exe /json /identify 0
```

Sample JSON output:

```json
{
    "0": {
        "DiskSizeGiB": 100.0,
        "NGUID": "a0a0a0a0-bbbb-cccc-dddd-e1e1e1e1e1e1",
        "VS": "type=os,name=FN-WinVM_ComboFO-XIO-os-1"
    }
}
```

To query disks without specifying a target, run:

```powershell
.\Azure-Boost-Observability.exe /identify
```

To get JSON-formatted output for the same scan, run:

```powershell
.\Azure-Boost-Observability.exe /json /identify
```

You can also target a drive letter:

```powershell
.\Azure-Boost-Observability.exe /identify D
```

As with performance queries, the tool scans up to 32 disks and stops at the first device it can't query.

## Use the tool on Linux

Follow these steps to prepare your Linux VM and query disk metrics or identification data.

### Set up the tool

1. Connect to your Linux VM.
1. Copy the `azure-boost-observability` script from the `AzureBoostObservability-Linux` folder to the VM.
1. Install Python 3 and `nvme-cli` by using your distribution's package manager. Queries that omit a disk also use `lsblk` to enumerate disks.
1. In the folder containing the script, modify the file permissions to make it executable:

    ```bash
    chmod +x ./azure-boost-observability
    ```

The following examples assume that you're in the folder containing the script. To display usage, run:

```bash
./azure-boost-observability --help
```

### Query metrics on Linux

Use the performance query command to retrieve the values described in [Disk performance metrics](#disk-performance-metrics).

Use `sudo` to run the script and specify the disk to query:

```bash
sudo ./azure-boost-observability --perf /dev/nvme0n1
```

Sample text output:

```output
Disk: /dev/nvme0n1
  Degraded performance: No
  Total Ops
    Read Ops: 7015
    Write Ops: 4034
  Total Bytes
    Read Bytes: 418151936
    Write Bytes: 47226880
  Outstanding I/Os: 29
  Ops Latency Max/Avg (us)
    Read Latency Max: 0
    Read Latency Avg: 0
    Write Latency Max: 18961
    Write Latency Avg: 5494
  Dispatch Latency Max/Avg (us)
    Read Dispatch Latency Max: 0
    Read Dispatch Latency Avg: 0
    Write Dispatch Latency Max: 4
    Write Dispatch Latency Avg: 2
```

To get JSON-formatted output, specify `--json`:

```bash
sudo ./azure-boost-observability --json --perf /dev/nvme0n1
```

Sample JSON output:

```json
{
    "/dev/nvme0n1": {
        "is_degraded": 0,
        "reads": 7015,
        "writes": 4034,
        "read_bytes": 418151936,
        "write_bytes": 47226880,
        "outstanding_io": 29,
        "read_latency_max": 0,
        "read_latency_avg": 0,
        "write_latency_max": 18961,
        "write_latency_avg": 5494,
        "read_dispatch_latency_max": 0,
        "read_dispatch_latency_avg": 0,
        "write_dispatch_latency_max": 4,
        "write_dispatch_latency_avg": 2
    }
}
```

If you don't specify a disk, the script enumerates disks by using `lsblk`, without filtering for NVMe devices. A failed device query can terminate the command before it prints any collected results. Specify an individual NVMe device to avoid unrelated-device failures.

### Identify disks on Linux

Use the identification command to retrieve the fields described in [Disk identification](#disk-identification).

To identify a specific disk, run:

```bash
sudo ./azure-boost-observability --identify /dev/nvme0n1
```

To get JSON-formatted output, specify `--json`:

```bash
sudo ./azure-boost-observability --json --identify /dev/nvme0n1
```

To query disks without specifying a target, run:

```bash
sudo ./azure-boost-observability --identify
```

To get JSON-formatted output for the same scan, run:

```bash
sudo ./azure-boost-observability --json --identify
```

The same Linux disk-enumeration and failure behavior described in [Query metrics on Linux](#query-metrics-on-linux) applies to identification.

## Related content

- For background on the storage interface, see [NVMe overview](nvme-overview.md).
- For other disk metrics in Azure, see [Disk metrics](disks-metrics.md).
- For the hardware offload architecture, see [Azure Boost overview](/azure/azure-boost/overview).

## Trademark attribution

The NVMe word mark is a registered and unregistered trademark and service mark of NVM Express, Inc. in the United States and other countries. Unauthorized use strictly prohibited.
