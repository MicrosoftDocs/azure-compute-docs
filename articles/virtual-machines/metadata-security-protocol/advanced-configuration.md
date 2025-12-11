---
title: Advanced Configuration for MSP
description: Learn about allowlists for Metadata Security Protocol (MSP).
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: concept-article
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
# Customer intent: "As a cloud security administrator, I want to define custom RBAC allowlists for metadata service endpoints, so that I can enhance the security of our virtual machines by restricting access to necessary applications and users only."
---

# Advanced configuration for MSP

When you're working with Metadata Security Protocol (MSP), you can optionally define custom role-based access control (RBAC) allowlists for metadata service endpoints on a per-user and/or per-process basis. A new resource type in Azure Compute Gallery, `InVMAccessControlProfile`, enables these allowlists.

## Motivation

Traditionally, metadata services treat the entire virtual machine (VM) as the trust boundary and allow any software running in the guest to access them. This access is overly permissive, especially if your workload isn't strictly implemented with a modern cloud-native architecture. Microsoft Security Response Center (MSRC) found that most security attacks can be prevented if organizations limit access to only the necessary software.

The `InVMAccessControlProfile` resource type enables you to define more granular control on the applications or users who can access the endpoints. You can use it to write rules like these examples:

- Only the Guest Proxy Agent (GPA) and SQL Server can access the network configuration.
- Only the monitoring extension can access tokens for managed identities.

The new resource type facilitates managing these configurations at scale. The rules that are appropriate for your VM depend on what software it's running. You can use `InVMAccessControlProfile` to define your configuration once and link it against all applicable VMs.

> [!NOTE]
> We recommend treating these controls as *defense in depth* in your threat modeling. These controls can significantly enhance the security of a workload, but they're best used as extra protection instead of as core isolation mechanisms.
>
> In cloud-native multitenant workloads, especially if untrusted code is executed, hardware-backed or hypervisor-backed isolation provides greater protection. (For example, [Azure Container Instances](https://azure.microsoft.com/products/container-instances) offers this kind of isolation.) MSP authorization rules complement hardware-backed isolation.

## Properties

| Property | Type | Details |
|--|--|--|
| `mode` |Enumeration: `"Audit" \| "Enforce" \| "Disabled"` | See  [Inline configuration](./configuration.md#inline-configuration). |
| `defaultAccess` | Enumeration: `"Allow" \| "Deny"` | Controls if an endpoint is accessible or restricted when no privileges for that resource are specified. See [Privileges](#privileges) later in this article. |
| `rules` | `object` | The RBAC definition to apply. See [Writing rules](#writing-rules) later in this article. |

## MSP allowlist (RBAC) concepts

The first step of access control is to verify the identity of the client. Normally, identity verification uses some type of a credential. MSP was designed to avoid introducing any breaking changes to clients, to maximize compatibility and make onboarding easier. Additionally, having clients upgraded with credentials introduces another layer of trust bootstrapping problems to solve.

Instead, MSP supports defining virtual identities against existing OS and process metadata. A VM's kernel keeps track of which user account a process belongs to, in addition to other metadata about its invocation. The GPA can identify which process made the HTTP request. As a result, the GPA can access that metadata to determine the identity of a caller and make authorization decisions.

### Roles

You use roles to group multiple privileges into a named, reusable unit. Grouping privileges into roles improves organization and readability.

### Role assignments

Role assignments pair an identity with a role. Requests that come from a process that matches the identity have all the privileges associated with that role.

### Identities

With MSP, you can define a set of conditions to be evaluated against a property bag of process metadata. If all the conditions match, the process is considered to belong to this identity and is granted privileges.

The GPA supports writing *exact match* rules against these metadata items:

| Process metadata | Description |
|--|--|
| `username` | The human-readable name of the account that the process is running under. |
| `groupName` | The human-readable name of the group that contains the account that the process is running under. A user can belong to multiple groups. Conditions here are implemented as a *set contains* operation. |
| `processName` | The display name of the process. This name is purely defense in depth. |
| `exePath` | The full path of the running executable file. For some programs, the runtime (not the file) appears. An example of such a program is Python. |

The ideal way to configure your workload is to run each application in a dedicated user account with uniquely scoped groups, and then write rules only against those properties. The user ID can't be spoofed, and there's no pattern matching to contend with. But because this method isn't practical in all workloads, we offer the other properties.

If you specify multiple properties, the matching is performed as a logical `AND`. For example, consider this identity definition:

```json
{
    "name": "WebServer",
    "exePath": "/usr/sbin/apache2",
    "processName": "apache2",
    "groupName": "www-data"
}
```

The definition would generate the following validation:

```c#
bool isWebServerIdentity = properties["exePath"] == "/usr/sbin/apache2"
    && properties["exePath"] == "apache2"
    && properties["groups"].contains("www-data");
```

### Privileges

Privileges grant access to specific endpoints. Some endpoints are RESTful and can be expressed purely as a path. Others are influenced by query parameters.

Privileges are defined with a `name` value, a `path` value, and an optional set of `queryParameters` values:

```json
"privileges": [
    {
        "name": "GoalState",
        "path": "/machine",
        "queryParameters": {"comp": "goalstate"}
    },
    {
        "name": "config",
        "path": "/machine",
        "queryParameters": {"comp": "config"}
    }
]
```

#### Comparison rules

- String matching is *case-insensitive*.
- If *no* query parameters are specified, the privilege grants access to *all* possible values on that path.

#### Default access rules

You can use default access (`defaultAccess`) modes to slowly lock down a workload. You can also use them to simplify rule authoring when you want to restrict access to only specific endpoints.

| `defaultAccess` mode | Behavior |
|--|--|
| `Deny` | If no explicit assignment exists to authorize the guest app to access the requested resource, the request is rejected. |
| `Allow` | If no rules exist for a specific privilege, access is allowed by default. If any privilege for that resource exists, the behavior reverts to denying by default for that resource. |

- If query parameters are provided, the question *"Is a privilege defined?"* for determining if `defaultAccess` applies becomes: *"Does any rule match this path + every specified query parameter in the rule is present + matches in the request?"*. Extra query parameters are ignored in this assessment.
- The request is rejected if duplicate query parameter keys in a request are invalid.

## Writing rules

Specifying an allowlist via RBAC rules is optional. But if you specify one, you must supply *all* fields. For example, if you didn't have any `roles` values to define, you would still provide the empty array.

The easiest way to generate rules is by first running MSP in `Audit` mode. Then, [use the audit logs to generate rules](./other-examples/audit-logs-to-rules.md).

The following full Azure Instance Metadata Service schema expansion shows an example of the `rules` subsection:

```json
{
  "defaultAccess": "allow",
  "mode": "enforce",
  "rules": {
    "privileges": [
      {
        "name": "Msi",
        "path": "/metadata/identity/oauth2/token"
      }
    ],
    "roles": [
      {
        "name": "MyWebApp",
        "privileges": [
          "Msi"
        ]
      }
    ],
    "identities": [
      {
        "name":"MyWebApp1",
        "processName":"WebApp1"
      }
    ],
    "roleAssignments": [
      {
        "role": "MyWebApp",
        "identities": [
          "MyWebApp1"
        ]
      }
    ]
  },
  "id": "D/uTRPye9b9xSUCRIgfRDF41zeY="
}
```

The following full WireServer schema expansion shows an example of the `rules` subsection:

```json
{
  "defaultAccess": "allow",
  "mode": "enforce",
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
      }
    ],
    "identities": [
      {
        "name": "WinPA",
        "exePath": "C:\\Windows\\OEM\\Unattend.wsf.exe",
        "processName": "winpa.exe",
        "userName": "SYSTEM"
      }
    ],
    "roleAssignments": [
      {
        "role": "Provisioning",
        "identities": [
          "WinPA"
        ]
      }
    ]
  },
  "id": "D/uTRPye9b9xSUCRIgfRDF41zeY="
}
```

## Implicit and default configurations

The following RBAC definition is applied as default behavior for defense-in-depth access rules that existed before MSP:

```json
{
    "imds": {
        "defaultAccess": "Allow",
        "mode": "Disabled",
    },
    "wireServer": {
        "defaultAccess": "Allow",
        "mode": "Enforce",
    }
}
```

Remember that WireServer allows only access from administrator or root processes. The `defaultAccess` value is still `Allow` here because this rule isn't user configurable. It's part of the GPA. The GPA checks if the client is an admin user before evaluating any RBAC rules for a WireServer request. If the client is not an admin user, the request is immediately rejected.
