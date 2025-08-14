---
title: Convert Audit Logs to an Allowlist
description: Learn more about audit logs with Metadata Security Protocol (MSP).
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: As a cloud administrator, I want to convert audit logs into an allowlist, so that I can define access control rules effectively and enhance the security of my virtual machine environment.
---

# Convert audit logs to an allowlist

With Metadata Security Protocol (MSP), you can define a custom role-based access control (RBAC) allowlist to help secure metadata service endpoints. The contents of the allowlist come from audit logs. A new resource type in Azure Compute Gallery, `InVMAccessControlProfile`, enables the allowlist.

To learn more about RBAC and the `InVMAccessControlProfile` resource type, see [Advanced configuration for MSP](../advanced-configuration.md).

## Structure of an allowlist

An allowlist consists of:

- **Identities**: Processes on the machine.
- **Privileges**: Endpoints that the identities access.
- **Roles**: A grouping of privileges.
- **Role assignments**: Roles and the list of identities granted access for those roles.

## Collect audit logs

If you enable MSP in `Audit` or `Enforce` mode, the Guest Proxy Agent (GPA) creates audit logs in the following folders inside the virtual machine (VM):

| Operating system | Audit log location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |

## Convert logs to rules

To create an allowlist, you can use an automated method or a manual method.

### Generate an allowlist automatically

You can use an allowlist generator tool to generate the access control rules. The tool helps parse the audit logs and provides a UI to generate the rules.

1. Download and run the allowlist generator tool. On the [latest release page](https://github.com/Azure/GuestProxyAgent/releases/latest), under **Assets**, select `allowListTool.exe`.

   The tool parses the `ProxyAgentConnection` logs and displays the current privileges and identities on the VM.

1. Create roles and role assignments:

   - To create a role, select a grouping of privileges and give the role a descriptive name.
   - To create a role assignment, select a role and a grouping of identities. These identities can access the privileges grouped in that role. Give the role assignment a descriptive name.

### Manually create an allowlist

After you enable a VM with MSP in `Audit` or `Enforce` mode, the proxy agent captures all the requests being made to the host endpoints.

In the connection logs, you can analyze the applications that are making the requests to the Azure Instance Metadata Service or WireServer endpoints.

[![Screenshot of connection logs.](../images/create-shared-image-gallery/status-log.png)](../images/create-shared-image-gallery/status-log.png#lightbox)

The following example shows the format of the captured JSON.

[![Screenshot of audit logs with captured JSON.](../images/create-shared-image-gallery/parse-json-from-logs.png)](../images/create-shared-image-gallery/parse-json-from-logs.png#lightbox)

In the log file, you can identify the endpoints that you want to secure. These endpoints appear in `privileges` in the final `InVMAccessControlProfile` instance. You can also identify the identities (`identities`) that should have access.

A simple rules schema might look like the following example.

[![Screenshot of a simple rules schema.](../images/create-shared-image-gallery/example-access-control-rules.png)](../images/create-shared-image-gallery/example-access-control-rules.png#lightbox)

## Create an InVMAccessControlProfile instance by using an ARM template

1. [Create a new private gallery](/azure/virtual-machines/create-gallery) in Azure Compute Gallery.

1. Create an `InVMAccessControlProfile` definition with parameters for:

    - Gallery name to store in (from step 1)
    - Profile name
    - OS type
    - Host endpoint type (WireServer or Instance Metadata Service)

1. Create a specific version.

Here's a sample `InVMAccessControlProfile` instance:

```
"properties": {
    "mode": "Enforce",
    "defaultAccess": "Allow",
    "rules": {
      "privileges": [
        {
          "name": "GoalState",
          "path": "/machine",
          "queryParameters": {
            "comp": "goalstate"
          }
        }
      ],
      "roles": [
        {
          "name": "Provisioning",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "ManageGuestExtensions",
          "privileges": [
            "GoalState"
          ]
        },
        {
          "name": "MonitoringAndSecret",
          "privileges": [
            "GoalState"
          ]
        }
      ],
      "identities": [
        {
          "name": "WinPA",
          "userName": "SYSTEM",
          "exePath": "C:\\Windows\\System32\\cscript.exe"
        },
        {
          "name": "GuestAgent",
          "userName": "SYSTEM",
          "processName": "WindowsAzureGuestAgent.exe"
        },
        {
          "name": "WaAppAgent",
          "userName": "SYSTEM",
          "processName": "WaAppAgent.exe"
        },
        {
          "name": "CollectGuestLogs",
          "userName": "SYSTEM",
          "processName": "CollectGuestLogs.exe"
        },
        {
          "name": "AzureProfileExtension",
          "userName": "SYSTEM",
          "processName": "AzureProfileExtension.exe"
        },
        {
          "name": "AzurePerfCollectorExtension",
          "userName": "SYSTEM",
          "processName": "AzurePerfCollectorExtension.exe"
        },
        {
          "name": "WaSecAgentProv",
          "userName": "SYSTEM",
          "processName": "WaSecAgentProv.exe"
        }
      ],
      "roleAssignments": [
        {
          "role": "Provisioning",
          "identities": [
            "WinPA"
          ]
        },
        {
          "role": "ManageGuestExtensions",
          "identities": [
            "GuestAgent",
            "WaAppAgent",
            "CollectGuestLogs"
          ]
        },
        {
          "role": "MonitoringAndSecret",
          "identities": [
            "AzureProfileExtension",
            "AzurePerfCollectorExtension",
            "WaSecAgentProv"
          ]
        }
      ]
    },
```
